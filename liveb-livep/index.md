# [#SPRESENSE 2026]Cインタプリタ(VM)を作ったにょ[LiveB/LiveP編]

2026年8月31日

![](images/img01.jpg)

# 【組み込み自作VM】C・Python・BASICが動くスタック型インタプリタファミリー（LiveC / LiveP / LiveB）

マイコン（RP2040やSony Spresense等）上で、動的にプログラムを記述・実行できるように開発した**軽量インタプリタファミリー（LiveC / LiveP / LiveB）**のまとめです。

最初に **C言語サブセット（LiveC）** を作成し、そのスタック型VMアーキテクチャを踏襲して **Python風（LiveP）** および **クラシックBASIC（LiveB）** を展開しました。

3つとも互いに依存関係を持たない**独立した単一ヘッダーファイル（`.hpp`）**として実装されており、外部ライブラリ無しで動く設計になっています。

---

## 1. 全体アーキテクチャと設計思想

3言語で構文解析（Lexer / Parser）は異なりますが、VM内部のメモリ構造や実行エンジンには共通の設計思想を採用しています。

- **3セグメント構成**:
  - `text_` (`std::vector<long>`): コンパイルされたバイトコード命令列
  - `data_` (`std::vector<long>` / `char*`): 文字列リテラル、グローバル変数、配列ストレージ
  - `stack_` (`std::vector<long>`): 式評価、引数渡し、GOSUB/関数コールバック用の共通ワークスタック
- **1ワード ＝ `long` 単位のアライメント**:
  
  組み込み環境でのメモリ境界違反（Alignment Error）を防ぎ、シンプルなポインタ演算を実現。
- **コンパイル時判定による型タグの軽量化**:
  
  実行時の重い動的型付け（Type Tag）を持たず、コンパイル時に「文字列アドレス」か「数値」かを静的に特定してVM命令・システムコールを発行。
- **ホスト連携システムコール**:
  
  `registerSyscall()` 経由でグラフィック（LovyanGFX等）、GPIO、シリアル入出力（`PRINT`, `INPUT` 等）を共通のインターフェースで柔軟にバインド可能。

---

## 2. 各インタプリタの特徴比較

| 言語 | 拡張子 | 対応構文・主な機能 | 制限事項・割り切り | 主な用途 |
| --- | --- | --- | --- | --- |
| **LiveC** | `.c` | • C言語サブセット（c4スタイル）<br>• `int`, `char`, ポインタ, 1次元配列<br>• `struct`（`.` / `->` 対応）<br>• `if/else`, `while`, `for`, `switch/case`<br>• オブジェクト型 `#define` 展開 | • `float` 非対応<br>• 多次元配列非対応<br>• 構造体の値渡し非対応 | 高速制御、グラフィック描画、複雑なデータ構造処理 |
| **LiveP** | `.py` | • Python風インデント（オフサイドライン）<br>• `def` 関数定義、`if/elif/else`<br>• `while`, `for i in range(...)`<br>• 裸の文字列リテラル検出 | • オブジェクト指向/クラス非対応<br>• リスト/辞書等コレクション非対応<br>• 文字列連結演算非対応 | 直感的なスクリプト記述、各種実験・自動化 |
| **LiveB** | `.bas` | • 行番号付きクラシックBASIC<br>• `LET`, `PRINT`, `INPUT`, `DIM`<br>• `IF ... THEN` (単一行)<br>• `GOTO`, `GOSUB`, `RETURN`<br>• `FOR ... TO ... STEP / NEXT`<br>• `$` 接尾辞による文字列変数処理 | • `ELSE` 非対応<br>• ソースコード順実行（行自動ソートなし）<br>• 多次元配列、文字列演算非対応 | レトロ計算機体験、デバッグコマンド実行 |

---

## 3. LiveB (liveB.hpp) の実装ポイント

今回追加した **LiveB** の特徴的な実装技術です。

1. **遅延ラインジャンプ解決 (Deferred Line Jumps)**
  - BASICでは `GOTO 200` や `GOSUB 500` のように、まだコンパイルされていない先の行番号へのジャンプ（前方参照）が発生します。
  - コンパイル時には命令スロットと目的行番号のペア（`pendingJumps_`）を記録しておき、全体のコンパイル完了後に行番号→バイトコードオフセットテーブル（`lineOffsets_`）を参照して一括パッチを適用します。
2. **`$` サフィックス変数とコンパイル時判定**
  - `A$` のように末尾に `$` が付く変数は、コンパイル時に「文字列アドレスが格納される変数」としてマーキングされます。
  - `PRINT A$` 実行時は、実行時型チェックを行わず即座に `__print_str` システムコールを発行するようにバイトコードが生成されます。
3. **専用ループスタック (`FORSETUP` / `NEXTOP`)**
  - `FOR i = 0 TO 10 STEP 2` などのループ構造を高速に処理するため、VM内部に `ForFrame`（変数アドレス・上限値・ステップ数・ループ先PC）を保持する専用スタックを搭載しています。

---

## 4. ホスト側での拡張子ルーティング例

拡張子に応じたインタプリタの呼び分けコード例です。

```cpp
#include "liveC.hpp"
#include "liveP.hpp"
#include "liveB.hpp"

enum class ScriptType { LiveC, LiveP, LiveB, Unknown };

ScriptType detectScriptType(const std::string& filename) {
    if (filename.length() >= 2 && filename.substr(filename.length() - 2) == ".c") return ScriptType::LiveC;
    if (filename.length() >= 3 && filename.substr(filename.length() - 3) == ".py") return ScriptType::LiveP;
    if (filename.length() >= 4 && filename.substr(filename.length() - 4) == ".bas") return ScriptType::LiveB;
    return ScriptType::Unknown;
}

bool executeScript(const std::string& filename, const std::string& source) {
    ScriptType type = detectScriptType(filename);
    
    if (type == ScriptType::LiveC) {
        CInterp vm;
        setupSyscalls(vm); // 各種システムコール登録
        return vm.run(source);
    } else if (type == ScriptType::LiveP) {
        LiveP vm;
        setupSyscalls(vm);
        return vm.run(source);
    } else if (type == ScriptType::LiveB) {
        LiveB vm;
        setupSyscalls(vm);
        return vm.run(source);
    }
    
    fprintf(stderr, "Unrecognized file extension: %s\n", filename.c_str());
    return false;
}
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

## コード(liveB.hpp)

```
//
// liveB.hpp
//
// "liveBasic": a classic, line-numbered BASIC for the same embedded
// stack-VM family as liveC.hpp / liveP.hpp. All three are independent,
// side-by-side interpreters -- this file doesn't touch or depend on
// either of the others.
//
//   10 REM add two numbers
//   20 LET X = 5
//   30 LET Y = 10
//   40 IF X < Y THEN PRINT "X IS SMALLER"
//   50 FOR I = 0 TO 4
//   60   PRINT I
//   70 NEXT I
//   80 GOSUB 200
//   90 END
//   200 REM subroutine
//   210 PRINT "IN SUBROUTINE"
//   220 RETURN
//
// Supported:
//   NUMBER stmt(':'stmt)*   -- lines run in the order they appear in the
//     source (ascending line numbers is the author's job, same as real
//     BASIC; this interpreter does not reorder them)
//   LET (optional) / PRINT / INPUT (numeric only, see below) /
//   IF expr THEN (NUMBER | stmt)   -- single-line, no ELSE
//   GOTO / GOSUB / RETURN          -- line numbers may be forward-referenced
//   FOR var = a TO b [STEP s] / NEXT [var]
//   DIM name(N)                     -- fixed-size 1-D array, 0-based
//   NAME(args)                      -- as a statement, calls a
//     host-registered syscall and discards its return value (e.g.
//     CLS(), GOTOXY(x,y)); as part of an expression, same call but the
//     return value is used (e.g. X = RND(10) if the host registers RND)
//   REM / ' (rest of line ignored) / END
//   +-*/ %, comparisons, AND/OR/NOT, parentheses, unary -
//   All variables are GLOBAL (classic BASIC has no per-GOSUB scope).
//   A name ending in '$' is a STRING variable (e.g. A$) -- this is a
//   compile-time marker, not a runtime type tag: it just tells PRINT/
//   assignment "treat this one as a string address", the same static
//   trick liveP.hpp uses for detecting a bare string literal.
// Not supported:
//   ELSE, DEF FN, multi-dimensional arrays, string concatenation or any
//     string operations, DATA/READ/RESTORE, INPUT of a string ($),
//     reordering lines by number (source order is execution order)
//
// PRINT accepts a comma/semicolon-separated list of string literals
// and/or numeric expressions (comma inserts a space; semicolon inserts
// nothing; a trailing ';' at the end of the list suppresses the final
// newline, the classic BASIC idiom for building a line across multiple
// PRINT statements). Like liveP's print(), which literal-vs-numeric this
// resolves is decided AT COMPILE TIME per argument, not via a real
// runtime type tag.
//
#pragma once

#include <cstdint>
#include <cstring>
#include <string>
#include <vector>
#include <map>
#include <functional>
#include <cstdio>
#include <cctype>

#ifndef LIVEB_TEXT_SIZE
#define LIVEB_TEXT_SIZE (16 * 1024)   // bytecode budget, in longs
#endif
#ifndef LIVEB_DATA_SIZE
#define LIVEB_DATA_SIZE (8 * 1024)    // string pool + variable/array storage, in longs
#endif
#ifndef LIVEB_STACK_SIZE
#define LIVEB_STACK_SIZE (1 * 1024)   // VM stack word count (expr eval + GOSUB return addrs)
#endif

class LiveB {
  public:
    std::function<void(const std::string&)> onError =
    [](const std::string& s) {
      fprintf(stderr, "%s\n", s.c_str());
    };

    void registerSyscall(const std::string& name, std::function<long(LiveB&, long*, int)> fn) {
      Id& id = idFor(name);
      id.Class = Sys;
      id.Val = (long)syscalls_.size();
      syscalls_.push_back(fn);
    }

    bool run(const std::string& src) {
      if (!compileInternal(src)) return false;
      return exec();
    }

    bool compileOnly(const std::string& src) {
      return compileInternal(src);
    }

  private:
    enum { Num = 200, Str, Id_,
           Let, Print, Input, If, Then, Goto, Gosub, Return_, For, To, Step, Next,
           Dim, End, And, Or, Not,
           Assign, Ne, Lt, Gt, Le, Ge, Add, Sub, Mul, Div, Mod,
           NewlineTok, Eof_
         };
    enum { IMM = 1, JMP, GOSUB_OP, RETURN_OP, BZ, PUSH, LI, SI,
           EQ, NE, LT, GT, LE, GE, ADD, SUB, MUL, DIV, MOD,
           FORSETUP, NEXTOP, SYSC, HALT
         };
    enum { Glo = 1, Arr, Sys };

    struct Id {
      int Class = 0; long Val = 0; long ArrSize = 0; // ArrSize > 0 only for Class==Arr
    };

    std::string src_;
    const char* p_ = nullptr;
    int line_ = 1;
    bool ok_ = true;

    int tk_ = 0; long ival_ = 0; std::string lastName_;
    Id* curId_ = nullptr;

    std::vector<long> text_, data_, stack_;
    long* e_ = nullptr; char* d_ = nullptr;

    std::map<std::string, Id> sym_;
    std::vector<std::function<long(LiveB&, long*, int)>> syscalls_;

    std::map<long, long> lineOffsets_;                 // BASIC line number -> bytecode offset
    std::vector<std::pair<long*, long>> pendingJumps_; // (operand slot to patch, target line number)

    long printNumIdx_ = -1, printStrIdx_ = -1, printSpaceIdx_ = -1, printNlIdx_ = -1;

    Id& idFor(const std::string& n) { return sym_[n]; }

    void err(const std::string& msg) {
      ok_ = false;
      onError("line " + std::to_string(line_) + ": " + msg);
    }

    long off(long* target) { return (long)(target - text_.data()); }
    void emit(long v) {
      if (e_ - text_.data() >= (long)text_.size()) { err("out of code memory"); return; }
      *e_++ = v;
    }
    void alignData() {
      size_t byteOff = (size_t)((char*)d_ - (char*)data_.data());
      size_t rem = byteOff % sizeof(long);
      if (rem != 0) d_ += (sizeof(long) - rem);
    }

    // ---- Lexer ----------------------------------------------------------
    // BASIC is traditionally case-insensitive for both keywords and
    // variable names -- identifiers are upcased as they're lexed, so
    // "PRINT"/"print"/"Print" and "X"/"x" are always the same token/name.

    static bool isIdentStart(char c) { return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z'); }
    static bool isIdentChar(char c) { return isIdentStart(c) || (c >= '0' && c <= '9'); }

    void next() {
      for (;;) {
        char c = *p_;
        if (c == '\0') { tk_ = Eof_; return; }
        if (c == ' ' || c == '\t' || c == '\r') { p_++; continue; }
        if (c == '\n') { p_++; line_++; tk_ = NewlineTok; return; }
        p_++;

        if (isIdentStart(c)) {
          const char* start = p_ - 1;
          while (isIdentChar(*p_)) p_++;
          if (*p_ == '$') p_++; // string-variable suffix is part of the name
          std::string name(start, p_ - start);
          for (auto& ch : name) ch = (char)toupper((unsigned char)ch);
          if (name == "REM") { while (*p_ && *p_ != '\n') p_++; continue; }
          static const std::map<std::string, int> kw = {
            {"LET", Let}, {"PRINT", Print}, {"INPUT", Input}, {"IF", If}, {"THEN", Then},
            {"GOTO", Goto}, {"GOSUB", Gosub}, {"RETURN", Return_}, {"FOR", For}, {"TO", To},
            {"STEP", Step}, {"NEXT", Next}, {"DIM", Dim}, {"END", End},
            {"AND", And}, {"OR", Or}, {"NOT", Not}
          };
          auto it = kw.find(name);
          if (it != kw.end()) { tk_ = it->second; return; }
          lastName_ = name;
          curId_ = &idFor(name);
          tk_ = Id_;
          return;
        }
        if (c >= '0' && c <= '9') {
          long v = c - '0';
          while (*p_ >= '0' && *p_ <= '9') { v = v * 10 + (*p_ - '0'); p_++; }
          ival_ = v;
          tk_ = Num;
          return;
        }
        if (c == '"') {
          std::string s;
          while (*p_ && *p_ != '"' && *p_ != '\n') s += *p_++;
          if (*p_ == '"') p_++; else err("unterminated string literal");
          if (d_ - (char*)data_.data() + (long)s.size() + 1 >= (long)(data_.size() * sizeof(long))) {
            err("out of string memory");
          } else {
            ival_ = (long)d_;
            for (char sc : s) *d_++ = sc;
            *d_++ = '\0';
            alignData(); // string bytes are unaligned; whatever gets allocated
                         // next (a variable/array, via 'long' pointer access)
                         // needs d_ back on an 8-byte boundary first
          }
          tk_ = Str;
          return;
        }
        if (c == '\'') { while (*p_ && *p_ != '\n') p_++; continue; } // apostrophe comment (common BASIC extension)
        if (c == '=') { tk_ = Assign; return; } // context decides assignment vs. equality
        if (c == '<') {
          if (*p_ == '=') { p_++; tk_ = Le; return; }
          if (*p_ == '>') { p_++; tk_ = Ne; return; }
          tk_ = Lt; return;
        }
        if (c == '>') { if (*p_ == '=') { p_++; tk_ = Ge; } else tk_ = Gt; return; }
        if (c == '+') { tk_ = Add; return; }
        if (c == '-') { tk_ = Sub; return; }
        if (c == '*') { tk_ = Mul; return; }
        if (c == '/') { tk_ = Div; return; }
        if (c == '%') { tk_ = Mod; return; }
        // ( ) , ; are returned as their raw char code
        tk_ = (unsigned char)c;
        return;
      }
    }

    // ---- Expressions ------------------------------------------------------

    bool primaryIsStr_ = false;

    // Emits code that leaves an ARRAY ELEMENT'S ADDRESS in `a` (base +
    // index*wordsize), given the array's Id and with the index
    // expression's own code about to be compiled by the caller. Shared
    // by both the read path (primary(), which LIs it afterward) and the
    // write path (arrayAssign(), which SIs into it).
    void emitArrayIndexExprThenAddress(Id* id) {
      // Caller has just consumed '(' ; we compile the index expression,
      // then combine it with the array's base address.
      exprOr();
      if (tk_ == ')') next(); else err("')' expected");
      emit(PUSH);
      emit(IMM); emit((long)sizeof(long));
      emit(MUL);
      emit(PUSH);
      emit(IMM); emit(id->Val);
      emit(ADD);
    }

    // Assumes '(' has just been consumed for a Sys-class identifier
    // (tk_ is now the first token of the argument list, or ')'). PUSHes
    // each comma-separated argument, then SYSC. Leaves the return value
    // in `a` -- a caller that doesn't need it (a bare statement-level
    // call like CLS()) simply doesn't use it afterward.
    void compileSysCall(Id* id) {
      int argc = 0;
      while (ok_ && tk_ != ')') {
        exprOr();
        emit(PUSH);
        argc++;
        if (tk_ == ',') next();
      }
      if (tk_ == ')') next(); else err("')' expected");
      if (argc > 16) err("too many arguments (max 16)");
      emit(SYSC); emit(id->Val); emit(argc);
    }

    void primary() {
      primaryIsStr_ = false;
      if (tk_ == Num) { emit(IMM); emit(ival_); next(); return; }
      if (tk_ == Str) { primaryIsStr_ = true; emit(IMM); emit(ival_); next(); return; }
      if (tk_ == Sub) { next(); primary(); emit(PUSH); emit(IMM); emit(-1); emit(MUL); return; }
      if (tk_ == Not) { next(); exprCmp(); emit(PUSH); emit(IMM); emit(0); emit(EQ); return; }
      if (tk_ == '(') { next(); exprOr(); if (tk_ == ')') next(); else err("')' expected"); return; }
      if (tk_ == Id_) {
        std::string nm = lastName_; Id* id = curId_; next();
        if (tk_ == '(') {
          next();
          if (id->Class == Sys) { compileSysCall(id); return; }
          if (id->Class != Arr) { err(nm + " is not an array or function"); }
          emitArrayIndexExprThenAddress(id);
          emit(LI);
          return;
        }
        if (id->Class != Glo) { err("undefined variable: " + nm); return; }
        emit(IMM); emit(id->Val);
        emit(LI);
        return;
      }
      err("invalid expression");
      next();
    }

    void exprMul() {
      primary();
      for (;;) {
        if (tk_ == Mul) { next(); emit(PUSH); primary(); emit(MUL); }
        else if (tk_ == Div) { next(); emit(PUSH); primary(); emit(DIV); }
        else if (tk_ == Mod) { next(); emit(PUSH); primary(); emit(MOD); }
        else break;
      }
    }

    void exprAdd() {
      exprMul();
      for (;;) {
        if (tk_ == Add) { next(); emit(PUSH); exprMul(); emit(ADD); }
        else if (tk_ == Sub) { next(); emit(PUSH); exprMul(); emit(SUB); }
        else break;
      }
    }

    void exprCmp() {
      exprAdd();
      for (;;) {
        if (tk_ == Assign) { next(); emit(PUSH); exprAdd(); emit(EQ); } // '=' as equality, in expression context
        else if (tk_ == Ne) { next(); emit(PUSH); exprAdd(); emit(NE); }
        else if (tk_ == Lt) { next(); emit(PUSH); exprAdd(); emit(LT); }
        else if (tk_ == Gt) { next(); emit(PUSH); exprAdd(); emit(GT); }
        else if (tk_ == Le) { next(); emit(PUSH); exprAdd(); emit(LE); }
        else if (tk_ == Ge) { next(); emit(PUSH); exprAdd(); emit(GE); }
        else break;
      }
    }

    void exprAnd() {
      exprCmp();
      while (tk_ == And) {
        next();
        emit(BZ); long* falseLabel = e_; emit(0);
        exprCmp();
        *falseLabel = off(e_);
      }
    }

    void exprOr() {
      exprAnd();
      while (tk_ == Or) {
        next();
        emit(BZ); long* checkRightLabel = e_; emit(0); // left is 0 (false) -> fall through, check right
        emit(IMM); emit(1);
        emit(JMP); long* endLabel = e_; emit(0);
        *checkRightLabel = off(e_);
        exprAnd();
        *endLabel = off(e_);
      }
    }

    // ---- Statements -------------------------------------------------------

    void requireVarForAssignOrDecl(const std::string& nm, Id*& id) {
      id = &idFor(nm);
      if (id->Class == 0) { id->Class = Glo; id->Val = (long)d_; d_ += sizeof(long); }
    }

    void assignStmt() {
      std::string nm = lastName_;
      Id* id = curId_;
      next();
      if (tk_ == '(') {
        next();
        if (id->Class == Sys) {
          // Statement-level call, e.g. CLS() / GOTOXY(x,y) -- the
          // return value (if any) is simply discarded.
          compileSysCall(id);
          return;
        }
        // Array element write: NAME(index) = expr
        if (id->Class != Arr) { err(nm + " is not an array (DIM it first)"); }
        emitArrayIndexExprThenAddress(id);
        emit(PUSH);
        if (tk_ == Assign) next(); else err("'=' expected");
        exprOr();
        emit(SI);
        return;
      }
      requireVarForAssignOrDecl(nm, id);
      emit(IMM); emit(id->Val);
      emit(PUSH);
      if (tk_ == Assign) next(); else err("'=' expected");
      exprOr();
      emit(SI);
    }

    // Prints one already-positioned argument (a string literal token, a
    // bare $-suffixed string variable, or a numeric expression), NOT
    // including any separator/newline -- the caller (printStmt) handles
    // those between/after items.
    void printOneArg() {
      if (tk_ == Str) {
        long addr = ival_;
        next();
        emit(IMM); emit(addr);
        emit(PUSH);
        emit(SYSC); emit(printStrIdx_); emit(1);
        return;
      }
      if (tk_ == Id_ && !lastName_.empty() && lastName_.back() == '$') {
        // A bare $-suffixed variable: its stored value is a string
        // address by the $ naming convention (a compile-time/static
        // check on the name, not a runtime type tag -- same idea as
        // liveP's bare-string-literal detection).
        std::string nm = lastName_; Id* id = curId_; next();
        if (id->Class != Glo) { err("undefined variable: " + nm); return; }
        emit(IMM); emit(id->Val);
        emit(LI);
        emit(PUSH);
        emit(SYSC); emit(printStrIdx_); emit(1);
        return;
      }
      exprOr();
      emit(PUSH);
      emit(SYSC); emit(printNumIdx_); emit(1);
    }

    void printStmt() {
      next(); // consume PRINT
      bool trailingSemi = false;
      bool any = false;
      while (ok_ && tk_ != NewlineTok && tk_ != ':' && tk_ != Eof_) {
        if (any) { emit(SYSC); emit(printSpaceIdx_); emit(0); }
        printOneArg();
        any = true;
        trailingSemi = false;
        if (tk_ == ',') { next(); trailingSemi = false; continue; }
        if (tk_ == ';') { next(); trailingSemi = true; continue; }
        break;
      }
      if (!trailingSemi) { emit(SYSC); emit(printNlIdx_); emit(0); }
    }

    void inputStmt() {
      next(); // consume INPUT
      if (tk_ != Id_) { err("variable name expected"); return; }
      std::string nm = lastName_;
      if (!nm.empty() && nm.back() == '$') { err("INPUT of a string ($) isn't supported -- numeric variables only"); }
      Id* id = curId_;
      next();
      requireVarForAssignOrDecl(nm, id);
      emit(IMM); emit(id->Val);
      emit(PUSH);
      emit(SYSC); emit(inputNumIdx_); emit(0);
      emit(SI);
    }
    long inputNumIdx_ = -1;

    // GOTO/GOSUB/IF-THEN-<linenum> all need to jump to a line number that
    // may not have been compiled yet (forward reference) -- emit a
    // placeholder now and resolve every one of these once the whole
    // program (and therefore every line's offset) is known.
    void emitDeferredLineJump(long opcode, long targetLine) {
      emit(opcode);
      long* slot = e_;
      emit(0);
      pendingJumps_.push_back({slot, targetLine});
    }

    void gotoStmt() {
      next();
      if (tk_ != Num) { err("line number expected"); return; }
      emitDeferredLineJump(JMP, ival_);
      next();
    }

    void gosubStmt() {
      next();
      if (tk_ != Num) { err("line number expected"); return; }
      emitDeferredLineJump(GOSUB_OP, ival_);
      next();
    }

    void ifStmt() {
      next(); // consume IF
      exprOr();
      if (tk_ == Then) next(); else err("THEN expected");
      emit(BZ); long* skip = e_; emit(0);
      if (tk_ == Num) {
        emitDeferredLineJump(JMP, ival_);
        next();
      } else {
        stmt();
      }
      *skip = off(e_);
    }

    void forStmt() {
      next(); // consume FOR
      if (tk_ != Id_) { err("loop variable expected"); return; }
      std::string nm = lastName_;
      Id* id = curId_;
      next();
      requireVarForAssignOrDecl(nm, id);
      if (tk_ == Assign) next(); else err("'=' expected");
      // var = start
      emit(IMM); emit(id->Val);
      emit(PUSH);
      exprOr();
      emit(SI);
      if (tk_ == To) next(); else err("TO expected");
      exprOr(); // limit -> leaves it in `a`; stash for FORSETUP
      emit(PUSH);
      if (tk_ == Step) { next(); exprOr(); } else { emit(IMM); emit(1); }
      emit(PUSH);
      // stack currently (top to bottom): step, limit
      emit(FORSETUP); emit(id->Val);
    }

    void nextStmt() {
      next(); // consume NEXT
      if (tk_ == Id_) next(); // variable name is accepted but not checked against the innermost FOR (documented simplification)
      emit(NEXTOP);
    }

    void dimStmt() {
      next(); // consume DIM
      if (tk_ != Id_) { err("array name expected"); return; }
      std::string nm = lastName_;
      Id* id = curId_;
      next();
      if (tk_ != '(') { err("'(' expected"); return; }
      next();
      if (tk_ != Num) { err("array size must be a constant"); return; }
      long size = ival_;
      next();
      if (tk_ != ')') { err("')' expected"); return; }
      next();
      if (size < 1) size = 1;
      if (id->Class != 0) { err(nm + " is already declared"); return; }
      id->Class = Arr;
      id->Val = (long)d_;
      id->ArrSize = size;
      d_ += size * sizeof(long);
    }

    void stmt() {
      if (tk_ == Let) { next(); if (tk_ != Id_) { err("variable name expected"); return; } assignStmt(); return; }
      if (tk_ == Id_) { assignStmt(); return; }
      if (tk_ == Print) { printStmt(); return; }
      if (tk_ == Input) { inputStmt(); return; }
      if (tk_ == If) { ifStmt(); return; }
      if (tk_ == Goto) { gotoStmt(); return; }
      if (tk_ == Gosub) { gosubStmt(); return; }
      if (tk_ == Return_) { next(); emit(RETURN_OP); return; }
      if (tk_ == For) { forStmt(); return; }
      if (tk_ == Next) { nextStmt(); return; }
      if (tk_ == Dim) { dimStmt(); return; }
      if (tk_ == End) { next(); emit(HALT); return; }
      err("statement expected");
      next();
    }

    void line() {
      long lineNum = ival_;
      next(); // consume the line number
      if (lineOffsets_.count(lineNum)) err("duplicate line number: " + std::to_string(lineNum));
      lineOffsets_[lineNum] = off(e_);
      while (ok_ && tk_ != NewlineTok && tk_ != Eof_) {
        stmt();
        if (tk_ == ':') { next(); continue; }
        break;
      }
      if (tk_ == NewlineTok) next();
    }

    // ---- VM exec ----------------------------------------------------------

    struct ForFrame { long varAddr; long limit; long step; long* bodyPC; };
    std::vector<ForFrame> forStack_;

    bool exec() {
      long* pc = text_.data();
      long* sp = stack_.data() + stack_.size();
      long a = 0;
      long cycles = 0;
      const long MAX_CYCLES = 200L * 1000L * 1000L;
      forStack_.clear();
      long* textEnd = text_.data() + text_.size();
      while (pc < textEnd) {
        if (++cycles > MAX_CYCLES) { onError("execution aborted: cycle limit exceeded (possible infinite loop)"); return false; }
        long op = *pc++;
        if (op == 0) break; // ran off the end of compiled code (no explicit END)
        switch (op) {
          case IMM: a = *pc++; break;
          case JMP: pc = text_.data() + *pc; break;
          case GOSUB_OP: {
            long target = *pc++;
            if (sp <= stack_.data()) { onError("stack overflow (GOSUB nested too deeply)"); return false; }
            *--sp = (long)pc;
            pc = text_.data() + target;
            break;
          }
          case RETURN_OP:
            if (sp >= stack_.data() + stack_.size()) { onError("RETURN without GOSUB"); return false; }
            pc = (long*)*sp++;
            break;
          case BZ: pc = a ? pc + 1 : text_.data() + *pc; break;
          case PUSH:
            if (sp <= stack_.data()) { onError("stack overflow"); return false; }
            *--sp = a;
            break;
          case LI: a = *(long*)a; break;
          case SI: *(long*)*sp++ = a; break;
          case EQ: a = (*sp++ == a); break;
          case NE: a = (*sp++ != a); break;
          case LT: a = (*sp++ < a); break;
          case GT: a = (*sp++ > a); break;
          case LE: a = (*sp++ <= a); break;
          case GE: a = (*sp++ >= a); break;
          case ADD: a = *sp++ + a; break;
          case SUB: a = *sp++ - a; break;
          case MUL: a = *sp++ * a; break;
          case DIV:
            if (a == 0) { onError("division by zero"); return false; }
            a = *sp++ / a;
            break;
          case MOD:
            if (a == 0) { onError("division by zero"); return false; }
            a = *sp++ % a;
            break;
          case FORSETUP: {
            long varAddr = *pc++;
            long step = *sp++;
            long limit = *sp++;
            ForFrame fr; fr.varAddr = varAddr; fr.limit = limit; fr.step = step; fr.bodyPC = pc;
            forStack_.push_back(fr);
            break;
          }
          case NEXTOP: {
            if (forStack_.empty()) { onError("NEXT without FOR"); return false; }
            ForFrame& fr = forStack_.back();
            long v = *(long*)fr.varAddr + fr.step;
            *(long*)fr.varAddr = v;
            bool cont = (fr.step >= 0) ? (v <= fr.limit) : (v >= fr.limit);
            if (cont) pc = fr.bodyPC;
            else forStack_.pop_back();
            break;
          }
          case SYSC: {
            long idx = *pc++;
            int argc = (int)*pc++;
            // Arguments were PUSHed left-to-right, so sp[0] is the LAST
            // one (top of stack) -- reverse into a small local buffer so
            // host callbacks see args[0] == the first source argument,
            // matching liveC's calling convention.
            long argsBuf[16];
            int n = argc < 16 ? argc : 16;
            for (int i = 0; i < n; i++) argsBuf[i] = sp[argc - 1 - i];
            a = syscalls_[idx](*this, argsBuf, argc);
            sp += argc;
            break;
          }
          case HALT: return true;
          default:
            onError("bad instruction");
            return false;
        }
        if (sp < stack_.data() || sp > stack_.data() + stack_.size()) { onError("stack corruption"); return false; }
      }
      return true;
    }

    bool compileInternal(const std::string& src) {
      // NOTE: sym_ is intentionally NOT cleared here -- registerSyscall()
      // calls made by the host before run()/compileOnly() must survive.
      // This assumes one LiveB instance is used for exactly one compile+
      // run cycle (create a fresh instance per script), matching how
      // this whole project's interpreters are used elsewhere.
      text_.assign(LIVEB_TEXT_SIZE, 0);
      data_.assign(LIVEB_DATA_SIZE, 0);
      stack_.assign(LIVEB_STACK_SIZE, 0);
      lineOffsets_.clear();
      pendingJumps_.clear();
      ok_ = true;
      line_ = 1;

      auto need = [&](const char* name) -> long {
        auto it = sym_.find(name);
        if (it == sym_.end() || it->second.Class != Sys) {
          err(std::string("host must register '") + name + "' before running a program");
          return -1;
        }
        return it->second.Val;
      };
      printNumIdx_ = need("__print_num");
      printStrIdx_ = need("__print_str");
      printSpaceIdx_ = need("__print_space");
      printNlIdx_ = need("__print_nl");
      inputNumIdx_ = need("__input_num");
      if (!ok_) return false;

      // Clear any stale Glo/Arr bindings from a hypothetical earlier
      // compile on this same instance, without touching Sys entries.
      for (auto& kv : sym_) {
        if (kv.second.Class == Glo || kv.second.Class == Arr) kv.second = Id();
      }

      src_ = src;
      p_ = src_.c_str();
      e_ = text_.data();
      d_ = (char*)data_.data();

      next();
      while (ok_ && tk_ != Eof_) {
        if (tk_ == NewlineTok) { next(); continue; }
        if (tk_ != Num) { err("line number expected"); break; }
        line();
      }
      if (!ok_) return false;

      for (auto& pj : pendingJumps_) {
        auto it = lineOffsets_.find(pj.second);
        if (it == lineOffsets_.end()) {
          err("undefined line number: " + std::to_string(pj.second));
          continue;
        }
        *pj.first = it->second;
      }
      return ok_;
    }
};
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

## コード(liveP.hpp)

```
//
// liveP.hpp
//
// "livePython": a lightweight, Python-FLAVORED scripting language for the
// same embedded stack-VM family as liveC.hpp / liveB.hpp. All three are
// independent, side-by-side interpreters -- this file doesn't touch or
// depend on either of the others.
//
// This is NOT a real Python implementation. It borrows Python's SURFACE
// SYNTAX (indentation-based blocks, no type declarations, `def`/`if`/
// `while`/`for x in range(...)`/`and`/`or`/`not`) while keeping the same
// underlying execution model as liveC/liveB: every value is one machine
// word, resolved statically at compile time -- there is no runtime type
// tag, no garbage collector, no dynamically-sized list/dict/tuple, and
// no string concatenation/slicing. Think of it as "Python-shaped syntax
// over a C-like machine", not CPython-compatible semantics.
//
//   def add(a, b):
//       return a + b
//
//   def main():
//       x = 5
//       y = 10
//       if x < y:
//           print("x is smaller")
//       for i in range(5):
//           print(i)
//       while x > 0:
//           x = x - 1
//       return 0
//
// Supported:
//   def name(a, b): ...                 (top-level only; must be defined
//                                         before use, like liveC)
//   if / elif / else, while,
//   for i in range(n) / range(a,b) / range(a,b,step)   (step may be negative)
//   return [expr], pass
//   assignment (name = expr) -- first assignment inside a function makes
//     it a local for that whole function; at the top level it's global
//   + - * / %, comparisons, and / or / not (short-circuit, returns the
//     actual operand value -- matching real Python's and/or, not forced
//     to 0/1), parentheses, unary -
//   string literals (usable directly as a bare print(...) argument, or
//     stored in / loaded from a variable -- see the print() note below)
//   import name  -- textually splices in another module's source (via a
//     host-supplied moduleLoader callback), NOT a real Python package import
//   # comments
//   NAME(args) as a bare statement (e.g. cls(), gotoxy(1,2)) calls a
//     host-registered function and discards its return value
// Not supported:
//   lists/dicts/tuples/sets/classes, string concatenation/slicing/
//     f-strings, multiple return values, *args/**kwargs, decorators,
//     generators/yield, try/except, with, lambda, walrus, one-line
//     "if x: y" (block form only), multi-line expressions inside
//     brackets, real dynamic typing (a variable's "kind" is whatever its
//     first assignment made it, statically), break/continue,
//     compound expression-statements beyond a single call/variable
//     (e.g. "f() + 1" as a bare statement -- write "x = f() + 1" instead)
//
// print(...) is special-cased by the compiler (not a normal registered
// syscall): each argument is checked AT COMPILE TIME -- a bare string
// literal token prints as text, anything else prints as a number -- since
// there's no runtime type tag to make that call at execution time.
// Printing a variable that holds a string address, or any string
// operation, is out of scope for the same reason. The host must register
// "__print_num" / "__print_str" / "__print_space" / "__print_nl".
//
#pragma once

#include <cstdint>
#include <cstring>
#include <string>
#include <vector>
#include <map>
#include <functional>
#include <cstdio>

#ifndef LIVEP_TEXT_SIZE
#define LIVEP_TEXT_SIZE (16 * 1024)   // bytecode budget, in longs
#endif
#ifndef LIVEP_DATA_SIZE
#define LIVEP_DATA_SIZE (8 * 1024)    // string pool + global storage, in longs
#endif
#ifndef LIVEP_STACK_SIZE
#define LIVEP_STACK_SIZE (1 * 1024)   // VM stack word count
#endif

class LiveP {
  public:
    std::function<void(const std::string&)> onError =
    [](const std::string& s) {
      fprintf(stderr, "%s\n", s.c_str());
    };

    // Called for "import name" -- must return the module's full source
    // text and set found=true, or set found=false if the module can't
    // be located. The host (Arduino glue) wires this to an SD read; the
    // portable core doesn't know how to read files.
    std::function<std::string(const std::string&, bool&)> moduleLoader = nullptr;

    void registerSyscall(const std::string& name, std::function<long(LiveP&, long*, int)> fn) {
      Id& id = idFor(name);
      id.Class = Sys;
      id.Val = (long)syscalls_.size();
      syscalls_.push_back(fn);
    }

    bool run(const std::string& src) {
      if (!compileInternal(src)) return false;
      return exec(entryOff_);
    }

    bool compileOnly(const std::string& src) {
      return compileInternal(src);
    }

  private:
    enum { Num = 200, Str, Id_,
           Def, If, Elif, Else, While, For, In, Range, Return, Import, Pass,
           And, Or, Not, True_, False_,
           Assign, Eq, Ne, Lt, Gt, Le, Ge, Add, Sub, Mul, Div, Mod,
           NewlineTok, Indent, Dedent, Eof_
         };
    enum { LEA = 1, IMM, JMP, JSR, BZ, BNZ, ENT, ADJ, LEV, LI, SI, PUSH,
           EQ, NE, LT, GT, LE, GE, ADD, SUB, MUL, DIV, MOD, SYSC
         };
    enum { Glo = 1, Loc, Fun, Sys };

    struct Id {
      int Class = 0; long Val = 0;
      int HClass = 0; long HVal = 0;
    };

    std::string src_;
    const char* p_ = nullptr;
    int line_ = 1;
    bool ok_ = true;

    int tk_ = 0; long ival_ = 0; std::string lastName_;
    Id* curId_ = nullptr;

    std::vector<int> indentStack_{0};
    int pendingDedents_ = 0;
    bool atLineStart_ = true;

    std::vector<long> text_, data_, stack_;
    long* e_ = nullptr; char* d_ = nullptr;
    long entryOff_ = 0;

    std::map<std::string, Id> sym_;
    std::vector<std::function<long(LiveP&, long*, int)>> syscalls_;

    int curLocals_ = 0;
    long* curAdjSlot_ = nullptr;
    bool curFuncActive_ = false;

    long printNumIdx_ = -1, printStrIdx_ = -1, printSpaceIdx_ = -1, printNlIdx_ = -1;

    Id& idFor(const std::string& n) { return sym_[n]; }

    void err(const std::string& msg) {
      ok_ = false;
      onError("line " + std::to_string(line_) + ": " + msg);
    }

    long off(long* target) { return (long)(target - text_.data()); }
    void emit(long v) {
      if (e_ - text_.data() >= (long)text_.size()) { err("out of code memory"); return; }
      *e_++ = v;
    }
    void alignData() {
      size_t byteOff = (size_t)((char*)d_ - (char*)data_.data());
      size_t rem = byteOff % sizeof(long);
      if (rem != 0) d_ += (sizeof(long) - rem);
    }

    // ---- Lexer ----------------------------------------------------------

    static bool isIdentStart(char c) { return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || c == '_'; }
    static bool isIdentChar(char c) { return isIdentStart(c) || (c >= '0' && c <= '9'); }

    // Skips blank lines and comment-only lines, measuring the indent of
    // the next real line of content (spaces and tabs both count as 1
    // column each -- mixing them consistently is the caller's job, the
    // same limitation real Python warns about but we don't detect).
    // Returns -1 at EOF.
    int measureIndentSkippingBlankLines() {
      for (;;) {
        int width = 0;
        while (*p_ == ' ' || *p_ == '\t') { width++; p_++; }
        if (*p_ == '\0') return -1;
        if (*p_ == '\r') { p_++; continue; }
        if (*p_ == '\n') { p_++; line_++; continue; }
        if (*p_ == '#') { while (*p_ && *p_ != '\n') p_++; continue; }
        return width;
      }
    }

    void next() {
      if (pendingDedents_ > 0) {
        pendingDedents_--;
        tk_ = Dedent;
        return;
      }

      if (atLineStart_) {
        int width = measureIndentSkippingBlankLines();
        if (width < 0) {
          if (indentStack_.size() > 1) {
            pendingDedents_ = (int)indentStack_.size() - 2;
            indentStack_.assign(1, 0);
            tk_ = Dedent;
            return;
          }
          tk_ = Eof_;
          return;
        }
        if (width > indentStack_.back()) {
          indentStack_.push_back(width);
          atLineStart_ = false;
          tk_ = Indent;
          return;
        }
        if (width < indentStack_.back()) {
          int pops = 0;
          while (indentStack_.size() > 1 && indentStack_.back() > width) { indentStack_.pop_back(); pops++; }
          if (indentStack_.back() != width) err("inconsistent indentation");
          atLineStart_ = false;
          if (pops > 1) pendingDedents_ = pops - 1;
          tk_ = Dedent;
          return;
        }
        atLineStart_ = false;
        // width == current level: fall through, lex the real token below
      }

      for (;;) {
        char c = *p_;
        if (c == '\0') {
          tk_ = NewlineTok; // implicit final newline so trailing DEDENTs can follow
          atLineStart_ = true;
          return;
        }
        if (c == ' ' || c == '\t' || c == '\r') { p_++; continue; }
        if (c == '#') { while (*p_ && *p_ != '\n') p_++; continue; }
        if (c == '\n') {
          p_++; line_++;
          tk_ = NewlineTok;
          atLineStart_ = true;
          return;
        }
        p_++;

        if (isIdentStart(c)) {
          const char* start = p_ - 1;
          while (isIdentChar(*p_)) p_++;
          std::string name(start, p_ - start);
          static const std::map<std::string, int> kw = {
            {"def", Def}, {"if", If}, {"elif", Elif}, {"else", Else},
            {"while", While}, {"for", For}, {"in", In}, {"range", Range},
            {"return", Return}, {"import", Import}, {"pass", Pass},
            {"and", And}, {"or", Or}, {"not", Not},
            {"True", True_}, {"False", False_}
          };
          auto it = kw.find(name);
          if (it != kw.end()) { tk_ = it->second; return; }
          lastName_ = name;
          curId_ = &idFor(name);
          tk_ = Id_;
          return;
        }
        if (c >= '0' && c <= '9') {
          long v = c - '0';
          while (*p_ >= '0' && *p_ <= '9') { v = v * 10 + (*p_ - '0'); p_++; }
          ival_ = v;
          tk_ = Num;
          return;
        }
        if (c == '"' || c == '\'') {
          char quote = c;
          std::string s;
          while (*p_ && *p_ != quote && *p_ != '\n') {
            char ch = *p_++;
            if (ch == '\\' && *p_) {
              char esc = *p_++;
              switch (esc) {
                case 'n': ch = '\n'; break;
                case 't': ch = '\t'; break;
                case '0': ch = '\0'; break;
                default: ch = esc; break;
              }
            }
            s += ch;
          }
          if (*p_ == quote) p_++; else err("unterminated string literal");
          if (d_ - (char*)data_.data() + (long)s.size() + 1 >= (long)(data_.size() * sizeof(long))) {
            err("out of string memory");
          } else {
            ival_ = (long)d_;
            for (char sc : s) *d_++ = sc;
            *d_++ = '\0';
            alignData();
          }
          tk_ = Str;
          return;
        }
        if (c == '=') { if (*p_ == '=') { p_++; tk_ = Eq; } else tk_ = Assign; return; }
        if (c == '!') { if (*p_ == '=') { p_++; tk_ = Ne; return; } err("unexpected '!'"); continue; }
        if (c == '<') { if (*p_ == '=') { p_++; tk_ = Le; } else tk_ = Lt; return; }
        if (c == '>') { if (*p_ == '=') { p_++; tk_ = Ge; } else tk_ = Gt; return; }
        if (c == '+') { tk_ = Add; return; }
        if (c == '-') { tk_ = Sub; return; }
        if (c == '*') { tk_ = Mul; return; }
        if (c == '/') { tk_ = Div; return; }
        if (c == '%') { tk_ = Mod; return; }
        // ( ) , : are returned as their raw char code
        tk_ = (unsigned char)c;
        return;
      }
    }

    // ---- Expressions --------------------------------------------------
    // Precedence (low to high): or, and, not(prefix), comparisons, + -, * / %, unary -, primary

    // Compiles a syscall/function call whose '(' has just been consumed
    // (tk_ is the first token of the argument list, or ')'). PUSHes each
    // comma-separated argument, then emits the call. Leaves the return
    // value in `a` -- callers that don't need it (a bare statement-level
    // call like cls()) simply don't use it afterward.
    void compileCallArgs(const std::string& nm, Id* id) {
      int argc = 0;
      while (ok_ && tk_ != ')') {
        expr();
        emit(PUSH);
        argc++;
        if (tk_ == ',') next();
      }
      if (tk_ == ')') next(); else err("')' expected");
      if (id->Class == Sys) {
        emit(SYSC); emit(id->Val); emit(argc);
      } else if (id->Class == Fun) {
        emit(JSR); emit(id->Val);
        if (argc) { emit(ADJ); emit(argc); }
      } else {
        err("undefined function: " + nm);
      }
    }

    // print(...) needs to know, per argument and AT COMPILE TIME,
    // whether it's a bare string literal (print as text) or a numeric
    // expression (print as a number) -- see the file header. This can't
    // share the generic push-then-batch call path above since each
    // argument needs different codegen depending on that check, so each
    // argument is compiled and printed immediately instead of PUSHed.
    void compilePrintCall() {
      next(); // consume '('
      int idx = 0;
      while (ok_ && tk_ != ')') {
        if (idx > 0) { emit(SYSC); emit(printSpaceIdx_); emit(0); }
        if (tk_ == Str) {
          long addr = ival_;
          next();
          emit(IMM); emit(addr);
          emit(PUSH);
          emit(SYSC); emit(printStrIdx_); emit(1);
        } else {
          expr();
          emit(PUSH);
          emit(SYSC); emit(printNumIdx_); emit(1);
        }
        idx++;
        if (tk_ == ',') next();
      }
      if (tk_ == ')') next(); else err("')' expected");
      emit(SYSC); emit(printNlIdx_); emit(0);
      emit(IMM); emit(0); // print() "returns" 0 -- a harmless stand-in for Python's None
    }

    void loadVariable(const std::string& nm, Id* id) {
      if (id->Class == Loc || id->Class == Glo) {
        emit(id->Class == Loc ? LEA : IMM);
        emit(id->Val);
        emit(LI);
      } else {
        err("undefined name: " + nm);
      }
    }

    // Handles everything that can follow an already-lexed identifier
    // (tk_ is now whatever comes after the name): a call (print(...) or
    // any other function/syscall) or a bare variable load. Shared by
    // primary() and the expression-statement path in stmt(), so a call
    // like cls() compiles identically whether it's part of a larger
    // expression or standing alone as its own statement.
    void primaryFromId(const std::string& nm, Id* id) {
      if (tk_ == '(') {
        if (nm == "print") { compilePrintCall(); return; }
        next();
        compileCallArgs(nm, id);
        return;
      }
      loadVariable(nm, id);
    }

    void primary() {
      if (tk_ == Num) { emit(IMM); emit(ival_); next(); return; }
      if (tk_ == Str) { emit(IMM); emit(ival_); next(); return; }
      if (tk_ == True_) { emit(IMM); emit(1); next(); return; }
      if (tk_ == False_) { emit(IMM); emit(0); next(); return; }
      if (tk_ == Sub) { next(); primary(); emit(PUSH); emit(IMM); emit(-1); emit(MUL); return; }
      if (tk_ == Not) { next(); expr(); emit(PUSH); emit(IMM); emit(0); emit(EQ); return; }
      if (tk_ == '(') { next(); expr(); if (tk_ == ')') next(); else err("')' expected"); return; }
      if (tk_ == Id_) {
        std::string nm = lastName_; Id* id = curId_; next();
        primaryFromId(nm, id);
        return;
      }
      err("invalid expression");
      next();
    }

    void exprMul() {
      primary();
      for (;;) {
        if (tk_ == Mul) { next(); emit(PUSH); primary(); emit(MUL); }
        else if (tk_ == Div) { next(); emit(PUSH); primary(); emit(DIV); }
        else if (tk_ == Mod) { next(); emit(PUSH); primary(); emit(MOD); }
        else break;
      }
    }

    void exprAdd() {
      exprMul();
      for (;;) {
        if (tk_ == Add) { next(); emit(PUSH); exprMul(); emit(ADD); }
        else if (tk_ == Sub) { next(); emit(PUSH); exprMul(); emit(SUB); }
        else break;
      }
    }

    void exprCmp() {
      exprAdd();
      for (;;) {
        if (tk_ == Eq) { next(); emit(PUSH); exprAdd(); emit(EQ); }
        else if (tk_ == Ne) { next(); emit(PUSH); exprAdd(); emit(NE); }
        else if (tk_ == Lt) { next(); emit(PUSH); exprAdd(); emit(LT); }
        else if (tk_ == Gt) { next(); emit(PUSH); exprAdd(); emit(GT); }
        else if (tk_ == Le) { next(); emit(PUSH); exprAdd(); emit(LE); }
        else if (tk_ == Ge) { next(); emit(PUSH); exprAdd(); emit(GE); }
        else break;
      }
    }

    void exprAnd() {
      exprCmp();
      while (tk_ == And) {
        next();
        emit(BZ); long* falseLabel = e_; emit(0);
        exprCmp();
        *falseLabel = off(e_);
      }
    }

    void expr() {
      exprAnd();
      while (tk_ == Or) {
        next();
        emit(BNZ); long* trueLabel = e_; emit(0);
        exprAnd();
        *trueLabel = off(e_);
      }
    }

    // ---- Statements -------------------------------------------------------

    void expectNewline() {
      if (tk_ == NewlineTok) next();
      else err("end of line expected");
    }

    // stmt() is defined further down in this file, but ifStmt()/
    // whileStmt()/forStmt()/funcDef() below can already call it (via
    // block()) without a forward declaration: member function BODIES
    // defined inside a class are compiled in "complete-class context",
    // where the whole class (all member names) is already visible,
    // regardless of textual order. (A separate forward declaration here
    // would actually break this -- C++ doesn't allow redeclaring a
    // member function signature that's later given a body in the same
    // class.)
    void block() {
      expectNewline();
      if (tk_ != Indent) { err("indented block expected"); return; }
      next();
      while (ok_ && tk_ != Dedent && tk_ != Eof_) stmt();
      if (tk_ == Dedent) next(); else err("dedent expected");
    }

    void ifStmt() {
      next(); // consume 'if'
      expr();
      if (tk_ == ':') next(); else err("':' expected");
      emit(BZ); long* elseLabel = e_; emit(0);
      block();
      std::vector<long*> endPatches;
      emit(JMP); endPatches.push_back(e_); emit(0);
      *elseLabel = off(e_);
      while (tk_ == Elif) {
        next();
        expr();
        if (tk_ == ':') next(); else err("':' expected");
        emit(BZ); long* nextLabel = e_; emit(0);
        block();
        emit(JMP); endPatches.push_back(e_); emit(0);
        *nextLabel = off(e_);
      }
      if (tk_ == Else) {
        next();
        if (tk_ == ':') next(); else err("':' expected");
        block();
      }
      long endOff = off(e_);
      for (long* p : endPatches) *p = endOff;
    }

    void whileStmt() {
      next(); // consume 'while'
      long top = off(e_);
      expr();
      if (tk_ == ':') next(); else err("':' expected");
      emit(BZ); long* exitLabel = e_; emit(0);
      block();
      emit(JMP); emit(top);
      *exitLabel = off(e_);
    }

    // Declares (or reuses) a local variable slot for 'name' in the
    // function currently being compiled -- matching liveC's declare-on-
    // first-assignment pattern for a for-loop's init clause.
    Id& localSlot(const std::string& name) {
      Id& id = idFor(name);
      if (id.Class != Loc) {
        id.HClass = id.Class; id.HVal = id.Val;
        id.Class = Loc;
        id.Val = -(++curLocals_);
        if (curAdjSlot_) *curAdjSlot_ = curLocals_;
      }
      return id;
    }

    Id& globalSlot(const std::string& name) {
      Id& id = idFor(name);
      if (id.Class == 0) {
        id.Class = Glo;
        id.Val = (long)d_;
        d_ += sizeof(long);
      }
      return id;
    }

    Id& targetSlot(const std::string& name) {
      return curFuncActive_ ? localSlot(name) : globalSlot(name);
    }

    void forStmt() {
      next(); // consume 'for'
      if (tk_ != Id_) { err("loop variable expected"); return; }
      std::string varName = lastName_;
      next();
      if (tk_ == In) next(); else err("'in' expected");
      if (tk_ == Range) next(); else err("only 'for x in range(...)' is supported");
      if (tk_ == '(') next(); else err("'(' expected");

      // range(a) / range(a,b) / range(a,b,step) -- each argument is
      // stashed in its own hidden local slot as it's parsed, since
      // start/stop/step are each referenced more than once below.
      int slots[3] = { -1, -1, -1 };
      int argCount = 0;
      while (ok_ && tk_ != ')' && argCount < 3) {
        int slot = ++curLocals_;
        if (curAdjSlot_) *curAdjSlot_ = curLocals_;
        emit(LEA); emit(-slot);
        emit(PUSH);
        expr();
        emit(SI);
        slots[argCount] = slot;
        argCount++;
        if (tk_ == ',') next();
      }
      if (tk_ == ')') next(); else err("')' expected");
      if (tk_ == ':') next(); else err("':' expected");
      if (argCount == 0) { err("range() needs at least 1 argument"); return; }

      int startSlot, stopSlot, stepSlot;
      if (argCount == 1) {
        stopSlot = slots[0];
        startSlot = ++curLocals_; if (curAdjSlot_) *curAdjSlot_ = curLocals_;
        emit(LEA); emit(-startSlot); emit(PUSH); emit(IMM); emit(0); emit(SI);
        stepSlot = ++curLocals_; if (curAdjSlot_) *curAdjSlot_ = curLocals_;
        emit(LEA); emit(-stepSlot); emit(PUSH); emit(IMM); emit(1); emit(SI);
      } else if (argCount == 2) {
        startSlot = slots[0]; stopSlot = slots[1];
        stepSlot = ++curLocals_; if (curAdjSlot_) *curAdjSlot_ = curLocals_;
        emit(LEA); emit(-stepSlot); emit(PUSH); emit(IMM); emit(1); emit(SI);
      } else {
        startSlot = slots[0]; stopSlot = slots[1]; stepSlot = slots[2];
      }

      Id& var = localSlot(varName);
      // var = start
      emit(LEA); emit(var.Val); emit(PUSH);
      emit(LEA); emit(-startSlot); emit(LI);
      emit(SI);

      long top = off(e_);
      // condition: step >= 0 ? (var < stop) : (var > stop) -- supports a
      // negative STEP counting down, matching real range()
      emit(LEA); emit(-stepSlot); emit(LI);
      emit(PUSH); emit(IMM); emit(0); emit(GE);
      emit(BZ); long* negBranch = e_; emit(0);
      emit(LEA); emit(var.Val); emit(LI);
      emit(PUSH); emit(LEA); emit(-stopSlot); emit(LI);
      emit(LT);
      emit(JMP); long* condDone = e_; emit(0);
      *negBranch = off(e_);
      emit(LEA); emit(var.Val); emit(LI);
      emit(PUSH); emit(LEA); emit(-stopSlot); emit(LI);
      emit(GT);
      *condDone = off(e_);

      emit(BZ); long* exitLabel = e_; emit(0);
      block();
      // var = var + step
      emit(LEA); emit(var.Val); emit(PUSH);
      emit(LEA); emit(var.Val); emit(LI);
      emit(PUSH); emit(LEA); emit(-stepSlot); emit(LI);
      emit(ADD);
      emit(SI);
      emit(JMP); emit(top);
      *exitLabel = off(e_);
    }

    void returnStmt() {
      next();
      if (tk_ != NewlineTok) expr(); else { emit(IMM); emit(0); }
      expectNewline();
      emit(LEV);
    }

    void passStmt() { next(); expectNewline(); }

    void importStmt() {
      next();
      if (tk_ != Id_) { err("module name expected"); return; }
      std::string modName = lastName_;
      next();
      expectNewline();
      if (!moduleLoader) { err("import isn't available in this environment (no module loader configured)"); return; }
      bool found = false;
      std::string modSrc = moduleLoader(modName, found);
      if (!found) { err("module not found: " + modName); return; }

      // Splice the module's source in place by swapping the lexer onto
      // it and back once exhausted. We save an OFFSET into the original
      // source (not a raw pointer) because reassigning src_ can
      // reallocate its buffer, which would leave a raw pointer dangling.
      // We also save the CURRENTLY HELD token: expectNewline() above
      // already lexed the first token of whatever follows this import
      // statement (e.g. "def" for a following "def main():"), so on
      // restore we must put that token back rather than re-lexing from
      // the raw character position, which would silently skip it.
      size_t savedOffset = (size_t)(p_ - src_.c_str());
      std::string savedSrc = src_;
      int savedLine = line_;
      auto savedIndentStack = indentStack_;
      int savedPending = pendingDedents_;
      bool savedAtStart = atLineStart_;
      int savedTk = tk_;
      long savedIval = ival_;
      std::string savedLastName = lastName_;
      Id* savedCurId = curId_;

      src_ = modSrc;
      p_ = src_.c_str();
      line_ = 1;
      indentStack_ = {0};
      pendingDedents_ = 0;
      atLineStart_ = true;
      next();
      while (ok_ && tk_ != Eof_) topLevelStmt();

      src_ = savedSrc;
      p_ = src_.c_str() + savedOffset; // recomputed from the (possibly reallocated) restored buffer
      line_ = savedLine;
      indentStack_ = savedIndentStack;
      pendingDedents_ = savedPending;
      atLineStart_ = savedAtStart;
      tk_ = savedTk;
      ival_ = savedIval;
      lastName_ = savedLastName;
      curId_ = savedCurId;
      // NOTE: no next() call here -- tk_ already holds the correct
      // not-yet-consumed token (see comment above).
    }

    // A statement that starts with an identifier is either an assignment
    // (name = expr) or an expression-statement (most commonly a bare
    // call, e.g. print(...) / cls()). Compound expression-statements
    // beyond a single call/variable aren't supported -- see file header.
    void assignOrExprStmt() {
      std::string nm = lastName_;
      Id* id = curId_;
      next();
      if (tk_ == Assign) {
        next();
        Id& target = targetSlot(nm);
        emit(target.Class == Loc ? LEA : IMM);
        emit(target.Val);
        emit(PUSH);
        expr();
        emit(SI);
        expectNewline();
        return;
      }
      primaryFromId(nm, id);
      expectNewline();
    }

    void funcDef() {
      next(); // consume 'def'
      if (tk_ != Id_) { err("function name expected"); return; }
      std::string fname = lastName_;
      Id& fid = idFor(fname);
      fid.Class = Fun;
      fid.Val = off(e_);
      next();
      if (tk_ == '(') next(); else err("'(' expected");
      std::vector<std::string> params;
      while (ok_ && tk_ != ')') {
        if (tk_ != Id_) { err("parameter name expected"); break; }
        params.push_back(lastName_);
        next();
        if (tk_ == ',') next();
      }
      if (tk_ == ')') next(); else err("')' expected");
      if (tk_ == ':') next(); else err("':' expected");

      emit(ENT); long* adjSlot = e_; emit(0);
      curLocals_ = 0;
      curAdjSlot_ = adjSlot;
      curFuncActive_ = true;

      int N = (int)params.size();
      for (int i = 0; i < N; i++) {
        Id& pid = idFor(params[i]);
        pid.HClass = pid.Class; pid.HVal = pid.Val;
        pid.Class = Loc; pid.Val = N + 1 - i; // bp+(N+1-i), matching liveC's convention
      }

      block();

      emit(IMM); emit(0);
      emit(LEV);
      *adjSlot = curLocals_;
      curAdjSlot_ = nullptr;
      curFuncActive_ = false;

      for (auto& pn : params) {
        Id& pid = sym_[pn];
        pid.Class = pid.HClass;
        pid.Val = pid.HVal;
      }
      for (auto& kv : sym_) {
        if (kv.second.Class == Loc) {
          kv.second.Class = kv.second.HClass;
          kv.second.Val = kv.second.HVal;
        }
      }
    }

    void stmt() {
      if (tk_ == If) { ifStmt(); return; }
      if (tk_ == While) { whileStmt(); return; }
      if (tk_ == For) { forStmt(); return; }
      if (tk_ == Return) { returnStmt(); return; }
      if (tk_ == Pass) { passStmt(); return; }
      if (tk_ == Import) { importStmt(); return; }
      if (tk_ == NewlineTok) { next(); return; } // stray blank logical line inside a block
      if (tk_ == Id_) { assignOrExprStmt(); return; }
      err("statement expected");
      next();
    }

    void topLevelStmt() {
      if (tk_ == Def) { funcDef(); return; }
      if (tk_ == NewlineTok) { next(); return; }
      stmt();
    }

    // ---- VM exec ------------------------------------------------------

    bool exec(long startOff) {
      static long EXIT_SENTINEL = -1;
      long* EXIT_MARK = &EXIT_SENTINEL;
      long* pc = text_.data() + startOff;
      long* sp = stack_.data() + stack_.size();
      long* bp = sp;
      if (sp <= stack_.data()) { onError("stack too small"); return false; }
      *--sp = (long)EXIT_MARK; // fake return address so main()'s LEV ends execution
      long a = 0;
      long cycles = 0;
      const long MAX_CYCLES = 200L * 1000L * 1000L;
      for (;;) {
        if (pc == EXIT_MARK) return true;
        if (++cycles > MAX_CYCLES) { onError("execution aborted: cycle limit exceeded (possible infinite loop)"); return false; }
        long op = *pc++;
        switch (op) {
          case LEA: a = (long)(bp + *pc++); break;
          case IMM: a = *pc++; break;
          case JMP: pc = text_.data() + *pc; break;
          case JSR: *--sp = (long)(pc + 1); pc = text_.data() + *pc; break;
          case BZ:  pc = a ? pc + 1 : text_.data() + *pc; break;
          case BNZ: pc = a ? text_.data() + *pc : pc + 1; break;
          case ENT: *--sp = (long)bp; bp = sp; sp -= *pc++; break;
          case ADJ: sp += *pc++; break;
          case LEV: sp = bp; bp = (long*)*sp++; pc = (long*)*sp++; break;
          case LI:  a = *(long*)a; break;
          case SI:  *(long*)*sp++ = a; break;
          case PUSH:
            if (sp <= stack_.data()) { onError("stack overflow"); return false; }
            *--sp = a;
            break;
          case EQ: a = (*sp++ == a); break;
          case NE: a = (*sp++ != a); break;
          case LT: a = (*sp++ < a); break;
          case GT: a = (*sp++ > a); break;
          case LE: a = (*sp++ <= a); break;
          case GE: a = (*sp++ >= a); break;
          case ADD: a = *sp++ + a; break;
          case SUB: a = *sp++ - a; break;
          case MUL: a = *sp++ * a; break;
          case DIV:
            if (a == 0) { onError("division by zero"); return false; }
            a = *sp++ / a;
            break;
          case MOD:
            if (a == 0) { onError("division by zero"); return false; }
            a = *sp++ % a;
            break;
          case SYSC: {
            long idx = *pc++;
            int argc = (int)*pc++;
            // Arguments were PUSHed left-to-right, so sp[0] is the LAST
            // one (top of stack) -- reverse into a local buffer so host
            // callbacks see args[0] == the first source argument,
            // matching liveC's/liveB's calling convention.
            long argsBuf[16];
            int n = argc < 16 ? argc : 16;
            for (int i = 0; i < n; i++) argsBuf[i] = sp[argc - 1 - i];
            a = syscalls_[idx](*this, argsBuf, argc);
            sp += argc;
            break;
          }
          default:
            onError("bad instruction");
            return false;
        }
        if (sp < stack_.data() || sp > stack_.data() + stack_.size() + 1) { onError("stack corruption"); return false; }
      }
    }

    bool compileInternal(const std::string& src) {
      // NOTE: sym_ is NOT cleared here -- registerSyscall() calls made by
      // the host before run()/compileOnly() must survive. This assumes
      // one LiveP instance is used for exactly one compile+run cycle
      // (create a fresh instance per script), matching how this whole
      // project's interpreters are used elsewhere. Stale Loc/Glo/Fun
      // bindings from a hypothetical earlier compile on the SAME
      // instance are cleared below, without touching Sys entries.
      for (auto& kv : sym_) {
        if (kv.second.Class == Glo || kv.second.Class == Loc || kv.second.Class == Fun) kv.second = Id();
      }

      text_.assign(LIVEP_TEXT_SIZE, 0);
      data_.assign(LIVEP_DATA_SIZE, 0);
      stack_.assign(LIVEP_STACK_SIZE, 0);
      ok_ = true;
      line_ = 1;
      curLocals_ = 0;
      curAdjSlot_ = nullptr;
      curFuncActive_ = false;
      indentStack_ = {0};
      pendingDedents_ = 0;
      atLineStart_ = true;

      auto need = [&](const char* name) -> long {
        auto it = sym_.find(name);
        if (it == sym_.end() || it->second.Class != Sys) {
          err(std::string("print() requires the host to register '") + name + "'");
          return -1;
        }
        return it->second.Val;
      };
      printNumIdx_ = need("__print_num");
      printStrIdx_ = need("__print_str");
      printSpaceIdx_ = need("__print_space");
      printNlIdx_ = need("__print_nl");
      if (!ok_) return false;

      src_ = src;
      p_ = src_.c_str();
      e_ = text_.data();
      d_ = (char*)data_.data();

      next();
      while (ok_ && tk_ != Eof_) topLevelStmt();
      if (!ok_) return false;

      auto mainIt = sym_.find("main");
      if (mainIt == sym_.end() || mainIt->second.Class != Fun) {
        err("no main() function found");
        return false;
      }
      entryOff_ = mainIt->second.Val;
      return true;
    }
};
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

## 実機

（X 投稿の埋め込み）

（X 投稿の埋め込み）

## 関連コンテンツ

・[liveOS liveC](https://elchika.com/article/b4e1e0f3-5044-46c0-acd8-899735e12ec4/)

・[liveOS エディタ編](https://elchika.com/article/01717490-251c-4edf-8baf-d6957eb59c84/)

・[liveOS ch9350編](https://elchika.com/article/bb6690e3-5f79-46b9-9365-c44652c4fe31/)

・[liveOS LiveB/LiveP](https://elchika.com/article/d3597a49-cd38-4c91-9aea-a5441b0be2b6/)

・[liveOS LiveCO](https://elchika.com/article/580bf544-bf7b-48d3-b36e-4143c13ac1f8/)

・[liveOS LiveFO](https://elchika.com/article/831f9d55-fa82-4460-b5e7-d978f21e13c9/)

・[liveOS LinDOS](https://elchika.com/article/2ed7a532-7b75-4d49-991c-986aac7c614c/)

・[liveOS linaX](https://elchika.com/article/d9e5a3d8-984e-4ca6-9bfa-780411ca9243/)

・[liveOS porting m5](https://elchika.com/article/6e66fc48-daf9-4017-9e5f-cf50534abdb8/) ←いまここ

*このプロジェクトはArduinoフレームワーク上で開発しています。ボードはVicharak Shrike-Lite (RP2040) / Sony Spresense / cyd / m5tab5 / m5sticks3 5機種に対応。*

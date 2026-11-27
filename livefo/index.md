---
image: https://chrmlinux.github.io/linaLog/livefo/images/img01.jpg
---

# [#SPRESENSE 2026] FORTRAN(VM)を作ったにょ [LiveFO編]

2026年9月3日 13:06

![](images/img01.jpg)

# 【第5弾】組み込み向け軽量VM言語シリーズ「liveFORTRAN (liveFO.hpp)」を実装してみた

組み込み向け軽量言語処理系シリーズの第5弾として、FORTRAN-77ライクな言語インタプリタ **`liveFO.hpp`**（liveFORTRAN）を実装しました！

これまで作成してきた `liveC.hpp`（C言語風）、`liveB.hpp`（BASIC風）、`liveP.hpp`（Python風）、`liveF.hpp`（Forth風）と同じ仮想マシン（VM）アーキテクチャファミリーに基づいた、ヘッダーオンリーのC++実装です。

---

## 特長と設計思想

* **ヘッダーファイル1つで動作**: `liveFO.hpp` をインクルードするだけで利用可能。
* **依存関係ゼロ**: 他の `live*` シリーズや外部ライブラリから完全に独立。
* **省メモリ・安全設計**:
  * 固定サイズバッファ（Bytecode / Data / Stack）を採用し、動的メモリ確保（`std::vector`の自動再確保等）によるメモリ枯渇を防止。
  * `std::stol` 等の例外を投げる処理を排除し、C++例外に頼らないハンドリング。
  * ゼロ除算やスタックオーバーフロー・アンダーフローのガード。
* **標準出力の抽象化**: `std::cout` 等へ直接出力せず、ホスト側で登録したシステムコール（`__print_num`, `__print_str`, `__print_space`, `__print_nl`）を介して出力（液晶モジュールやシリアル出力、自作エディタ等への組み込みが容易）。

---

## デモコード（liveFORTRAN 記法）

以下のような FORTRAN-77 風の構文をサポートしています。

```fortran
PROGRAM DEMO
INTEGER X, Y, I, N
X = 5
Y = 10
PRINT *, 'X PLUS Y IS', X + Y
DO 10 I = 1, 5
  PRINT *, I
10 CONTINUE
IF (X .LT. Y) THEN
  PRINT *, 'X IS SMALLER'
END IF
PRINT *, 'PRESS A KEY'
N = GETCH()
STOP
END

```
## コード
```
//
// liveFO.hpp
//
// "liveFORTRAN": a FORTRAN-77-flavored language for the same embedded VM
// family as liveC.hpp / liveB.hpp / liveP.hpp / liveF.hpp. Independent
// of all four -- doesn't touch or depend on any of them.

/*

         PROGRAM DEMO
         INTEGER X, Y, I, N
         X = 5
         Y = 10
         PRINT *, 'X PLUS Y IS', X + Y
         DO 10 I = 1, 5
           PRINT *, I
         10 CONTINUE
         IF (X .LT. Y) THEN
           PRINT *, 'X IS SMALLER'
         END IF
         PRINT *, 'PRESS A KEY'
         N = GETCH()
         STOP
         END

*/

// Supported:
//   PROGRAM name (optional, just skipped)
//   INTEGER var, var, ...     (declares globals; also declare-on-first-
//                              assignment like liveBasic, so this is
//                              mostly documentation)
//   assignment: var = expr
//   PRINT *, item, item, ...  (list-directed output; string literals use
//                              'single quotes', space-separated, newline
//                              at the end)
//   IF (expr) THEN ... [ELSE ...] END IF     (block IF; no one-line
//                              "IF (expr) statement" form)
//   DO label var = start, stop [, step]  ...  label CONTINUE
//                              (label may be forward- or same-referenced;
//                              nested DO loops are supported up to a
//                              fixed depth -- see LIVEFO_MAX_DO_DEPTH)
//   GOTO label                (labels may be forward-referenced)
//   STOP / END                (both halt the program)
//   Numeric labels: any line may start with a label number; a line with
//     nothing but a label is a no-op landing point for GOTO
//   + - * / , comparisons via .LT. .GT. .EQ. .NE. .LE. .GE.
//   .AND. .OR. .NOT.          (short-circuit AND/OR; NOT is a prefix op)
//   .TRUE. .FALSE.            (1 / 0)
//   Parentheses, unary -
//   NAME(args) as a bare statement or inside an expression calls a
//     host-registered syscall (e.g. GETCH(), KBHIT(), N = GETCH())
//   ! trailing comments (to end of line) -- classic column-1 'C'
//     comments are NOT supported, use ! instead
//   All variables are GLOBAL (no SUBROUTINE/FUNCTION definitions in
//     this lightweight subset -- see "Not supported")
// Not supported:
//   REAL/floating point (integer-only, like every other Live* language
//     here), CHARACTER variables (string literals may only be used
//     directly as PRINT arguments), arrays/DIMENSION, user-defined
//     SUBROUTINE/FUNCTION, FORMAT/WRITE with format strings, one-line
//     arithmetic/logical IF, DATA statements, COMMON blocks
//
// Safety, matching liveC/liveB/liveP/liveF: fixed-size text/data/stack
// buffers (no unbounded growth), no C++ exceptions anywhere in the
// compiler (hand-rolled number parsing, not std::stol), bounds-checked
// stack push/pop, undefined-label/unknown-word errors go through
// onError() rather than silently doing something with a wrong default.
// All output goes through host-registered syscalls (__print_num /
// __print_str / __print_space / __print_nl) -- nothing is ever written
// directly to std::cout.
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

#ifndef LIVEFO_TEXT_SIZE
#define LIVEFO_TEXT_SIZE (16 * 1024)   // bytecode budget, in longs
#endif
#ifndef LIVEFO_DATA_SIZE
#define LIVEFO_DATA_SIZE (8 * 1024)    // string pool + variable storage, in longs
#endif
#ifndef LIVEFO_STACK_SIZE
#define LIVEFO_STACK_SIZE (1 * 1024)   // VM stack word count
#endif
#ifndef LIVEFO_MAX_DO_DEPTH
#define LIVEFO_MAX_DO_DEPTH 8          // max nested DO loops
#endif

class LiveFO {
  public:
    std::function<void(const std::string&)> onError =
    [](const std::string& s) {
      fprintf(stderr, "%s\n", s.c_str());
    };

    void registerSyscall(const std::string& name, std::function<long(LiveFO&, long*, int)> fn) {
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
           Program, Integer, Print, If, Then, Else, EndTok, Do, Continue, Goto, Stop,
           LtOp, GtOp, EqOp, NeOp, LeOp, GeOp, AndOp, OrOp, NotOp, TrueTok, FalseTok,
           Assign, Add, Sub, Mul, Div,
           NewlineTok, Eof_
         };
    enum { IMM = 1, JMP, BZ, BNZ, PUSH, LI, SI,
           LT, GT, EQ, NE, LE, GE, ADD, SUB, MUL, DIV,
           SYSC, HALT
         };
    enum { Glo = 1, Sys };

    struct Id { int Class = 0; long Val = 0; };

    std::string src_;
    const char* p_ = nullptr;
    int line_ = 1;
    bool ok_ = true;

    int tk_ = 0; long ival_ = 0; std::string lastName_;
    Id* curId_ = nullptr;

    std::vector<long> text_, data_, stack_;
    long* e_ = nullptr; char* d_ = nullptr;

    std::map<std::string, Id> sym_;
    std::vector<std::function<long(LiveFO&, long*, int)>> syscalls_;

    std::map<long, long> labelOffsets_;                  // FORTRAN label -> bytecode offset
    std::vector<std::pair<long*, long>> pendingJumps_;   // (operand slot to patch, target label)

    long printNumIdx_ = -1, printStrIdx_ = -1, printSpaceIdx_ = -1, printNlIdx_ = -1;

    struct PendingDo {
      long label;
      Id* varId;
      long stopAddr;
      long stepAddr;
      long loopTop;
      long* exitPatch;
    };
    std::vector<PendingDo> doStack_;
    long doStopAddr_[LIVEFO_MAX_DO_DEPTH];
    long doStepAddr_[LIVEFO_MAX_DO_DEPTH];

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
    // Identifiers/keywords are upcased, matching FORTRAN's traditional
    // case-insensitivity (and liveBasic's convention in this project).

    static bool isIdentStart(char c) { return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z'); }
    static bool isIdentChar(char c) { return isIdentStart(c) || (c >= '0' && c <= '9'); }

    void next() {
      for (;;) {
        char c = *p_;
        if (c == '\0') { tk_ = Eof_; return; }
        if (c == ' ' || c == '\t' || c == '\r') { p_++; continue; }
        if (c == '!') { while (*p_ && *p_ != '\n') p_++; continue; }
        if (c == '\n') { p_++; line_++; tk_ = NewlineTok; return; }
        p_++;

        if (isIdentStart(c)) {
          const char* start = p_ - 1;
          while (isIdentChar(*p_)) p_++;
          std::string name(start, p_ - start);
          for (auto& ch : name) ch = (char)toupper((unsigned char)ch);
          static const std::map<std::string, int> kw = {
            {"PROGRAM", Program}, {"INTEGER", Integer}, {"PRINT", Print},
            {"IF", If}, {"THEN", Then}, {"ELSE", Else}, {"END", EndTok},
            {"DO", Do}, {"CONTINUE", Continue}, {"GOTO", Goto}, {"STOP", Stop}
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
        if (c == '\'') {
          std::string s;
          while (*p_ && *p_ != '\'' && *p_ != '\n') s += *p_++;
          if (*p_ == '\'') p_++; else err("unterminated string literal");
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
        if (c == '.') {
          const char* opStart = p_;
          while (isIdentStart(*p_)) p_++;
          // Note: avoid the name `word` — Arduino defines it as a macro.
          std::string opWord(opStart, p_ - opStart);
          for (auto& ch : opWord) ch = (char)toupper((unsigned char)ch);
          if (*p_ != '.') { err("malformed '.' operator"); tk_ = Eof_; return; }
          p_++;
          if (opWord == "LT") { tk_ = LtOp; return; }
          if (opWord == "GT") { tk_ = GtOp; return; }
          if (opWord == "EQ") { tk_ = EqOp; return; }
          if (opWord == "NE") { tk_ = NeOp; return; }
          if (opWord == "LE") { tk_ = LeOp; return; }
          if (opWord == "GE") { tk_ = GeOp; return; }
          if (opWord == "AND") { tk_ = AndOp; return; }
          if (opWord == "OR") { tk_ = OrOp; return; }
          if (opWord == "NOT") { tk_ = NotOp; return; }
          if (opWord == "TRUE") { tk_ = TrueTok; return; }
          if (opWord == "FALSE") { tk_ = FalseTok; return; }
          err("unknown operator: ." + opWord + ".");
          tk_ = Eof_;
          return;
        }
        if (c == '=') { tk_ = Assign; return; }
        if (c == '+') { tk_ = Add; return; }
        if (c == '-') { tk_ = Sub; return; }
        if (c == '*') { tk_ = Mul; return; }
        if (c == '/') { tk_ = Div; return; }
        // ( ) , are returned as their raw char code
        tk_ = (unsigned char)c;
        return;
      }
    }

    // ---- Expressions --------------------------------------------------

    bool primaryIsStr_ = false;

    void primary() {
      primaryIsStr_ = false;
      if (tk_ == Num) { emit(IMM); emit(ival_); next(); return; }
      if (tk_ == Str) { primaryIsStr_ = true; emit(IMM); emit(ival_); next(); return; }
      if (tk_ == TrueTok) { emit(IMM); emit(1); next(); return; }
      if (tk_ == FalseTok) { emit(IMM); emit(0); next(); return; }
      if (tk_ == Sub) { next(); primary(); emit(PUSH); emit(IMM); emit(-1); emit(MUL); return; }
      if (tk_ == NotOp) { next(); primary(); emit(PUSH); emit(IMM); emit(0); emit(EQ); return; }
      if (tk_ == '(') { next(); exprOrImpl(); if (tk_ == ')') next(); else err("')' expected"); return; }
      if (tk_ == Id_) {
        std::string nm = lastName_; Id* id = curId_; next();
        if (tk_ == '(') {
          next();
          int argc = 0;
          while (ok_ && tk_ != ')') {
            exprOrImpl();
            emit(PUSH);
            argc++;
            if (tk_ == ',') next();
          }
          if (tk_ == ')') next(); else err("')' expected");
          if (id->Class != Sys) { err("unknown function: " + nm); return; }
          emit(SYSC); emit(id->Val); emit(argc);
          return;
        }
        if (id->Class != Glo) { err("undefined variable: " + nm); return; }
        emit(IMM); emit(id->Val); emit(LI);
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
        if (tk_ == LtOp) { next(); emit(PUSH); exprAdd(); emit(LT); }
        else if (tk_ == GtOp) { next(); emit(PUSH); exprAdd(); emit(GT); }
        else if (tk_ == EqOp) { next(); emit(PUSH); exprAdd(); emit(EQ); }
        else if (tk_ == NeOp) { next(); emit(PUSH); exprAdd(); emit(NE); }
        else if (tk_ == LeOp) { next(); emit(PUSH); exprAdd(); emit(LE); }
        else if (tk_ == GeOp) { next(); emit(PUSH); exprAdd(); emit(GE); }
        else break;
      }
    }

    void exprAnd() {
      exprCmp();
      while (tk_ == AndOp) {
        next();
        emit(BZ); long* falseLabel = e_; emit(0);
        exprCmp();
       *falseLabel = off(e_);
      }
    }

    void exprOrImpl() {
      exprAnd();
      while (tk_ == OrOp) {
        next();
        emit(BNZ); long* trueLabel = e_; emit(0);
        exprAnd();
        *trueLabel = off(e_);
      }
    }

    // ---- Statements -------------------------------------------------------

    Id& requireGlo(const std::string& nm, Id* id) {
      if (id->Class == 0) { id->Class = Glo; id->Val = (long)d_; d_ += sizeof(long); }
      (void)nm;
      return *id;
    }

    void emitDeferredLabelJump(long opcode, long targetLabel) {
      emit(opcode);
      long* slot = e_;
      emit(0);
      pendingJumps_.push_back({slot, targetLabel});
    }

    void printOneArg() {
      if (tk_ == Str) {
        long addr = ival_;
        next();
        emit(IMM); emit(addr);
        emit(PUSH);
        emit(SYSC); emit(printStrIdx_); emit(1);
      } else {
        exprOrImpl();
        emit(PUSH);
        emit(SYSC); emit(printNumIdx_); emit(1);
      }
    }

    void printStmt() {
      next(); // consume PRINT
      if (tk_ != Mul) { err("'*' expected after PRINT"); return; }
      next();
      if (tk_ == ',') next();
      bool any = false;
      while (ok_ && tk_ != NewlineTok && tk_ != Eof_) {
        if (any) { emit(SYSC); emit(printSpaceIdx_); emit(0); }
        printOneArg();
        any = true;
        if (tk_ == ',') { next(); continue; }
        break;
      }
      emit(SYSC); emit(printNlIdx_); emit(0);
    }

    // Assignment or a bare function-call statement (e.g. "CALL"-less
    // intrinsic like GETCH()). Mirrors liveB's approach: peek past the
    // identifier to see whether '=' follows.
    void assignOrCallStmt() {
      std::string nm = lastName_;
      Id* id = curId_;
      next();
      if (tk_ == Assign) {
        next();
        Id& target = requireGlo(nm, id);
        emit(IMM); emit(target.Val);
        emit(PUSH);
        exprOrImpl();
        emit(SI);
        return;
      }
      if (tk_ == '(') {
        next();
        int argc = 0;
        while (ok_ && tk_ != ')') {
          exprOrImpl();
          emit(PUSH);
          argc++;
          if (tk_ == ',') next();
        }
        if (tk_ == ')') next(); else err("')' expected");
        if (id->Class != Sys) { err("unknown function: " + nm); return; }
        emit(SYSC); emit(id->Val); emit(argc);
        return;
      }
      err("'=' or '(' expected after " + nm);
    }

    void gotoStmt() {
      next();
      if (tk_ != Num) { err("label expected after GOTO"); return; }
      emitDeferredLabelJump(JMP, ival_);
      next();
    }

    void ifStmt() {
      next(); // consume IF
      if (tk_ == '(') next(); else err("'(' expected");
      exprOrImpl();
      if (tk_ == ')') next(); else err("')' expected");
      if (tk_ == Then) next(); else err("THEN expected (one-line IF isn't supported)");
      if (tk_ == NewlineTok) next(); else err("end of line expected after THEN");

      emit(BZ); long* elsePatch = e_; emit(0);

      stmtBlockUntilElseOrEndIf();

      if (tk_ == Else) {
        emit(JMP); long* endPatch = e_; emit(0);
        *elsePatch = off(e_);
        next();
        if (tk_ == NewlineTok) next(); else err("end of line expected after ELSE");
        stmtBlockUntilElseOrEndIf();
        *endPatch = off(e_);
      } else {
        *elsePatch = off(e_);
      }

      // current line should be "END IF"
      if (tk_ == EndTok) {
        next();
        if (tk_ == If) next(); else err("IF expected after END");
      } else {
        err("END IF expected");
      }
    }

    bool atBlockTerminator() {
      return tk_ == Else || tk_ == EndTok || tk_ == Eof_;
    }

    void stmtBlockUntilElseOrEndIf() {
      while (ok_ && !atBlockTerminator()) {
        parseOneLine();
      }
    }

    void doStmt() {
      next(); // consume DO
      if (tk_ != Num) { err("label expected after DO"); return; }
      long label = ival_;
      next();
      if (tk_ != Id_) { err("loop variable expected"); return; }
      std::string nm = lastName_; Id* varIdRaw = curId_; next();
      Id& varId = requireGlo(nm, varIdRaw);
      if (tk_ != Assign) { err("'=' expected"); return; }
      next();

      emit(IMM); emit(varId.Val);
      emit(PUSH);
      exprOrImpl();
      emit(SI);

      if (tk_ != ',') { err("',' expected"); return; }
      next();

      if ((int)doStack_.size() >= LIVEFO_MAX_DO_DEPTH) { err("too many nested DO loops"); return; }
      int depth = (int)doStack_.size();
      long stopAddr = doStopAddr_[depth];
      long stepAddr = doStepAddr_[depth];

      emit(IMM); emit(stopAddr);
      emit(PUSH);
      exprOrImpl();
      emit(SI);

      emit(IMM); emit(stepAddr);
      emit(PUSH);
      if (tk_ == ',') {
        next();
        exprOrImpl();
      } else {
        emit(IMM); emit(1);
      }
      emit(SI);

      long loopTop = off(e_);
      // condition: step >= 0 ? var <= stop : var >= stop
      emit(IMM); emit(stepAddr); emit(LI);
      emit(PUSH); emit(IMM); emit(0); emit(GE);
      emit(BZ); long* negBranch = e_; emit(0);
      emit(IMM); emit(varId.Val); emit(LI);
      emit(PUSH); emit(IMM); emit(stopAddr); emit(LI);
      emit(LE);
      emit(JMP); long* condDone = e_; emit(0);
      *negBranch = off(e_);
      emit(IMM); emit(varId.Val); emit(LI);
      emit(PUSH); emit(IMM); emit(stopAddr); emit(LI);
      emit(GE);
      *condDone = off(e_);

      emit(BZ); long* exitPatch = e_; emit(0);

      PendingDo pd;
      pd.label = label; pd.varId = &varId; pd.stopAddr = stopAddr; pd.stepAddr = stepAddr;
      pd.loopTop = loopTop; pd.exitPatch = exitPatch;
      doStack_.push_back(pd);
    }

    // Called when a CONTINUE statement's line label matches the
    // innermost pending DO -- closes that loop (increment, test, jump
    // back, patch the exit). A CONTINUE with no matching pending DO (or
    // no label at all) is just a no-op statement, matching real Fortran
    // (CONTINUE is a valid statement anywhere).
    void closeMatchingDoLoops(long lineLabel, bool hasLabel) {
      while (hasLabel && !doStack_.empty() && doStack_.back().label == lineLabel) {
        PendingDo pd = doStack_.back();
        doStack_.pop_back();
        emit(IMM); emit(pd.varId->Val);
        emit(PUSH);
        emit(IMM); emit(pd.varId->Val); emit(LI);
        emit(PUSH); emit(IMM); emit(pd.stepAddr); emit(LI);
        emit(ADD);
        emit(SI);
        emit(JMP); emit(pd.loopTop);
        *pd.exitPatch = off(e_);
      }
    }

    void stmt() {
      if (tk_ == Integer) {
        next();
        while (tk_ == Id_) {
          requireGlo(lastName_, curId_);
          next();
          if (tk_ == ',') next(); else break;
        }
        return;
      }
      if (tk_ == Print) { printStmt(); return; }
      if (tk_ == If) { ifStmt(); return; }
      if (tk_ == Do) { doStmt(); return; }
      if (tk_ == Goto) { gotoStmt(); return; }
      if (tk_ == Continue) { next(); return; } // loop-closing handled by parseOneLine's label check
      if (tk_ == Stop || tk_ == EndTok) { next(); emit(HALT); return; }
      if (tk_ == Id_) { assignOrCallStmt(); return; }
      err("statement expected");
      next();
    }

    // Parses one physical line: [label] [statement] NEWLINE. Also
    // registers the label's bytecode offset (for GOTO) and, if this line
    // is "label CONTINUE", closes any matching pending DO loop(s).
    void parseOneLine() {
      while (tk_ == NewlineTok) next(); // skip blank lines
      // Only stop on EOF here. Else/EndTok must still be reachable as
      // top-level statements (END => HALT). Block contexts already avoid
      // calling parseOneLine when atBlockTerminator() is true.
      if (tk_ == Eof_) return;

      long lineLabel = 0;
      bool hasLabel = false;
      if (tk_ == Num) {
        lineLabel = ival_;
        hasLabel = true;
        if (labelOffsets_.count(lineLabel)) err("duplicate label: " + std::to_string(lineLabel));
        labelOffsets_[lineLabel] = off(e_);
        next();
      }

      bool wasContinue = (tk_ == Continue);

      if (tk_ != NewlineTok && tk_ != Eof_) stmt();

      if (wasContinue) closeMatchingDoLoops(lineLabel, hasLabel);

      if (tk_ == NewlineTok) next();
      else if (tk_ != Eof_ && !atBlockTerminator()) err("end of line expected");
    }

    // ---- VM exec ------------------------------------------------------

    bool exec() {
      long* pc = text_.data();
      long* sp = stack_.data() + stack_.size();
      long a = 0;
      long cycles = 0;
      const long MAX_CYCLES = 200L * 1000L * 1000L;
      long* textEnd = text_.data() + text_.size();

      while (pc < textEnd) {
        if (++cycles > MAX_CYCLES) { onError("execution aborted: cycle limit exceeded (possible infinite loop)"); return false; }
        long op = *pc++;
        if (op == 0) break;
        switch (op) {
          case IMM: a = *pc++; break;
          case JMP: pc = text_.data() + *pc; break;
          case BZ:  pc = a ? pc + 1 : text_.data() + *pc; break;
          case BNZ: pc = a ? text_.data() + *pc : pc + 1; break;
          case PUSH:
            if (sp <= stack_.data()) { onError("stack overflow"); return false; }
            *--sp = a;
            break;
          case LI: a = *(long*)a; break;
          case SI: *(long*)*sp++ = a; break;
          case LT: a = (*sp++ < a); break;
          case GT: a = (*sp++ > a); break;
          case EQ: a = (*sp++ == a); break;
          case NE: a = (*sp++ != a); break;
          case LE: a = (*sp++ <= a); break;
          case GE: a = (*sp++ >= a); break;
          case ADD: a = *sp++ + a; break;
          case SUB: a = *sp++ - a; break;
          case MUL: a = *sp++ * a; break;
          case DIV:
            if (a == 0) { onError("division by zero"); return false; }
            a = *sp++ / a;
            break;
          case SYSC: {
            long idx = *pc++;
            int argc = (int)*pc++;
            if (sp + argc > stack_.data() + stack_.size()) { onError("stack underflow"); return false; }
            long argsBuf[16];
            int n = argc < 16 ? argc : 16;
            for (int i = 0; i < n; i++) argsBuf[i] = sp[argc - 1 - i];
            sp += argc;
            if (idx < 0 || idx >= (long)syscalls_.size()) { onError("bad syscall index"); return false; }
            a = syscalls_[idx](*this, argsBuf, argc);
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
      // sym_ is NOT cleared here -- registerSyscall() calls made by the
      // host before run()/compileOnly() must survive. Stale Glo entries
      // from a hypothetical earlier compile on the SAME instance are
      // cleared below, without touching Sys entries.
      for (auto& kv : sym_) {
        if (kv.second.Class == Glo) kv.second = Id();
      }

      text_.assign(LIVEFO_TEXT_SIZE, 0);
      data_.assign(LIVEFO_DATA_SIZE, 0);
      stack_.assign(LIVEFO_STACK_SIZE, 0);
      labelOffsets_.clear();
      pendingJumps_.clear();
      doStack_.clear();
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
      if (!ok_) return false;

      src_ = src;
      p_ = src_.c_str();
      e_ = text_.data();
      d_ = (char*)data_.data();

      // Reserve hidden globals for DO-loop stop/step bookkeeping, one
      // pair per nesting level, before any user variable can claim these
      // addresses.
      for (int i = 0; i < LIVEFO_MAX_DO_DEPTH; i++) {
        doStopAddr_[i] = (long)d_; d_ += sizeof(long);
        doStepAddr_[i] = (long)d_; d_ += sizeof(long);
      }

      next();
      // Skip an optional leading "PROGRAM name" line.
      while (tk_ == NewlineTok) next();
      if (tk_ == Program) {
        next();
        if (tk_ == Id_) next();
        if (tk_ == NewlineTok) next();
      }

      while (ok_ && tk_ != Eof_) parseOneLine();
      if (!ok_) return false;

      if (!doStack_.empty()) { err("DO without matching CONTINUE"); return false; }

      for (auto& pj : pendingJumps_) {
        auto it = labelOffsets_.find(pj.second);
        if (it == labelOffsets_.end()) {
          err("undefined label: " + std::to_string(pj.second));
          continue;
        }
        *pj.first = it->second;
      }
      return ok_;
    }
};
```
## 実機
<blockquote class="twitter-tweet" data-theme="dark"><a href="https://twitter.com/chrmlinux03/status/2095366117791547678"></a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 関連コンテンツ
・[liveOS liveC](../livec/)
・[liveOS エディタ編](../livec-editor/)
・[liveOS ch9350編](../livec-ch9350/)
・[liveOS LiveB/LiveP](../liveb-livep/)
・[liveOS LiveCO](../liveco/)
・[liveOS LiveFO](../livefo/)
・[liveOS LinDOS](../lindos/)
・[liveOS linaX](../spresense-linax/)
・[liveOS porting m5](../spresense-linax-port/) ←いまここ

*このプロジェクトはArduinoフレームワーク上で開発しています。ボードはVicharak Shrike-Lite (RP2040) / Sony Spresense / cyd / m5tab5 / m5sticks3 5機種に対応。*

<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b" data-lyket-id="livefo"></div></div>

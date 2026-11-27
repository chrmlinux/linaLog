# [#SPRESENSE 2026] COBOLインタプリタ(VM)を作ったにょ [LiveCO編]

2026年9月2日 13:20

![](images/img01.jpg)

# LiveCOBOL — liveシリーズ第4弾

## 概要

**LiveCOBOL（LiveCO）** は、`liveC` から発展してきた **liveシリーズ第4弾**として開発した、軽量なCOBOL風スクリプト言語です。

従来のliveシリーズが持つ「小さな処理系で、組み込み環境でも動かせる」という方向性を引き継ぎながら、COBOLの文法を取り入れています。

LiveCOは単独のCOBOL処理系を目指すものではなく、**liveシリーズ共通のスタックVM上で動く、COBOL風の小型言語**という位置付けです。

## liveシリーズ

liveシリーズは、同じ組み込み向けのスタックVMという考え方をベースに、異なる言語スタイルへ展開しています。

- **liveP** — 初期の軽量スクリプト系
- **liveB** — BASIC系の文法
- **liveC** — C系の文法
- **LiveCO** — COBOL系の文法

LiveCOは、この流れの**第4弾**にあたります。

## LiveCOの特徴

### COBOL風の構成

プログラムをCOBOLでおなじみの構成に分けて記述できます。

- `IDENTIFICATION DIVISION`
- `DATA DIVISION`
- `WORKING-STORAGE SECTION`
- `PROCEDURE DIVISION`

そのため、COBOLに馴染みのある人が読みやすい構成になっています。

### WORKING-STORAGE

変数は `WORKING-STORAGE SECTION` に定義します。

数値データについては、COBOLの `PIC 9(n)` を意識した簡易的な型指定を利用できます。

例えば、

- `PIC 9`
- `PIC 9(3)`
- `PIC X`
- `PIC X(n)`

といった形式に対応しています。

### 基本的な処理

現在のLiveCOでは、以下のような基本命令を扱えます。

- `MOVE`
- `ADD`
- `DISPLAY`
- `IF`
- `GO TO`
- `CALL`
- `STOP RUN`

COBOLらしい記述を保ちながら、内部ではVMの命令列へコンパイルして実行します。

## 算術演算

`ADD` では、通常の加算だけでなく、COBOL風の `GIVING` も利用できます。

```text
ADD X TO Y GIVING Z
```

この場合、

```text
Z = X + Y
```

となり、`Y` 自体は変更されません。

## DISPLAY

`DISPLAY` を使って文字列、数値、変数などを出力できます。

```text
DISPLAY "RESULT Z IS" Z
```

LiveCO内部では、ホスト側に登録された出力用システムコールを利用して表示を行います。

## CALL とホスト機能

LiveCOの特徴の一つが、**ホスト側の機能を `CALL` から呼び出せること**です。

例えば、

```text
CALL "GETCH" GIVING K
```

のように記述できます。

`GETCH` のような機能はLiveCO本体に直接固定するのではなく、ホスト側からシステムコールとして登録できます。

これにより、実行環境に合わせて、

- キー入力
- 画面操作
- デバイス操作
- 独自の組み込み機能

などを追加できます。

つまり、LiveCOの言語部分を小さく保ったまま、組み込み環境側の機能を拡張できます。

## コンパイルと実行

LiveCOには、

- ソースをコンパイルして実行する
- コンパイルだけ行う

という2つの利用方法があります。

コンパイルされたプログラムは、Liveシリーズで共通する考え方の**スタックベースのVM**上で実行されます。

## 小型処理系としての位置付け

LiveCOは、本格的なCOBOL処理系を置き換えることを目的としたものではありません。

目的は、

> **COBOL風の読みやすい記述を、小さな組み込み向けVMで実行すること**

です。

そのため、COBOLの巨大な仕様をすべて実装するのではなく、組み込み用途で扱いやすい機能に絞っています。

## liveCからLiveCOへ

`liveC` ではC言語に近い記述を採用していました。

LiveCOでは、その実行基盤となる考え方を引き継ぎながら、表現方法をCOBOL風に変更しています。

つまり、

**言語の見た目は変わっても、軽量なVMで実行するという設計思想は共通**

しています。

この構成により、同じ組み込み向けの基盤から、

- C風
- BASIC風
- COBOL風

といった異なるプログラミングスタイルを展開できます。

## サンプル

最小限の計算例です。

```text
IDENTIFICATION DIVISION.
DATA DIVISION.
WORKING-STORAGE SECTION.
01 X PIC 9(3) VALUE 5.
01 Y PIC 9(3) VALUE 10.
01 Z PIC 9(3).

PROCEDURE DIVISION.
MAIN-LOGIC.
  ADD X TO Y GIVING Z.
  DISPLAY "RESULT Z IS" Z.
  STOP RUN.
```

この例では、`X` と `Y` を加算し、その結果を `Z` に格納して表示します。

## 今後の拡張

LiveCOは、liveシリーズ共通のVMを利用することで、言語機能を追加しながら拡張できます。

特に `CALL` によるホスト連携を利用すると、組み込み機器向けに用途を広げられます。

例えば、

- キーボード入力
- LCDなどへの表示
- GPIO操作
- シリアル通信
- センサー読み取り
- マイコン固有機能

などをホスト側のシステムコールとして追加する構成が考えられます。

## まとめ

**LiveCOBOL（LiveCO）は、liveCから発展したliveシリーズ第4弾のCOBOL風スクリプト言語です。**

COBOLらしい `DIVISION`、`WORKING-STORAGE`、`PIC`、`DISPLAY`、`ADD ... GIVING` などの記述を採用しながら、内部では軽量なスタックVMで実行します。

最大の特徴は、言語そのものを大きくせず、`CALL` によってホスト環境の機能を組み込める点です。

**「COBOLの読みやすさ」＋「liveシリーズの軽量VM」＋「組み込み環境との連携」**

を目指した、第4のlive言語です。

## コード
```
#pragma once

//
// LiveCO.hpp
//
// "LiveCOBOL": a lightweight COBOL-flavored scripting language for the
// same embedded stack-VM family as liveC.hpp / liveB.hpp / liveP.hpp.
//

/*

IDENTIFICATION DIVISION.
DATA DIVISION.
WORKING-STORAGE SECTION.
01 X PIC 9(3) VALUE 5.
01 Y PIC 9(3) VALUE 10.
01 Z PIC 9(3).
01 K PIC 9(3).
PROCEDURE DIVISION.
MAIN-LOGIC.
  ADD X TO Y GIVING Z.
  DISPLAY "RESULT Z IS" Z.
  CALL "GETCH" GIVING K.
  STOP RUN.

*/

#include <cctype>
#include <cstdio>
#include <cstdint>
#include <cstring>
#include <functional>
#include <map>
#include <string>
#include <vector>

#ifndef LIVECO_TEXT_SIZE
#define LIVECO_TEXT_SIZE (16 * 1024)   // bytecode budget, in longs
#endif
#ifndef LIVECO_DATA_SIZE
#define LIVECO_DATA_SIZE (8 * 1024)    // variable storage, in longs
#endif
#ifndef LIVECO_STACK_SIZE
#define LIVECO_STACK_SIZE (1 * 1024)   // VM stack word count
#endif

class LiveCO {
  public:
    std::function<void(const std::string&)> onError = [](const std::string& s) {
        fprintf(stderr, "%s\n", s.c_str());
    };

    void registerSyscall(const std::string& name, std::function<long(LiveCO&, long*, int)> fn) {
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
    enum {
        Num = 200, Str, Id_, DotTok,
        Pic, Value, Move, To, Add, Giving, Display, If, Then, EndIf, Go, TokTo, Stop, Run,
        Call, Using,
        Eof_
    };
    enum {
        IMM = 1, JMP, BZ, PUSH, LI, SI,
        EQ, NE, LT, GT, LE, GE, ADD, SUB, MUL, DIV, MOD,
        SYSC, HALT
    };
    enum { Glo = 1, Lab, Sys };

    struct Id {
        int Class = 0;
        long Val = 0;
        long MaxValue = 0; // limit for PIC 9(N), e.g. 1000 for 9(3)
    };

    std::string src_;
    const char* p_ = nullptr;
    int line_ = 1;
    bool ok_ = true;

    int tk_ = 0;
    long ival_ = 0;
    std::string lastName_;
    Id* curId_ = nullptr;

    std::vector<long> text_, data_, stack_;
    long* e_ = nullptr;
    char* d_ = nullptr;

    std::map<std::string, Id> sym_;
    std::vector<std::function<long(LiveCO&, long*, int)>> syscalls_;

    std::map<std::string, long> labelOffsets_;                 // paragraph name -> bytecode offset
    std::vector<std::pair<long*, std::string>> pendingJumps_;  // (slot to patch, target paragraph)

    long printNumIdx_ = -1, printStrIdx_ = -1, printSpaceIdx_ = -1, printNlIdx_ = -1;

    Id& idFor(const std::string& n) { return sym_[n]; }

    void err(const std::string& msg) {
        ok_ = false;
        onError("line " + std::to_string(line_) + ": " + msg);
    }

    long off(long* target) { return (long)(target - text_.data()); }

    void emit(long v) {
        if (e_ - text_.data() >= (long)text_.size()) {
            err("out of code memory");
            return;
        }
        *e_++ = v;
    }

    // ---- Lexer -------------------------------------------------------------
    static bool isIdentStart(char c) { return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z'); }
    static bool isIdentChar(char c) { return isIdentStart(c) || (c >= '0' && c <= '9') || c == '-'; }

    void next() {
        for (;;) {
            char c = *p_;
            if (c == '\0') { tk_ = Eof_; return; }
            if (c == ' ' || c == '\t' || c == '\r') { p_++; continue; }
            if (c == '\n') { p_++; line_++; continue; }

            // '*' at line start or after space is a COBOL comment
            if (c == '*') { while (*p_ && *p_ != '\n') p_++; continue; }
            p_++;

            if (c == '.') { tk_ = DotTok; return; }

            if (isIdentStart(c)) {
                const char* start = p_ - 1;
                while (isIdentChar(*p_)) p_++;
                std::string name(start, p_ - start);
                for (auto& ch : name) ch = (char)toupper((unsigned char)ch);

                static const std::map<std::string, int> kw = {
                    {"PIC", Pic},
                    {"VALUE", Value},                     {"MOVE", Move},
                    {"TO", To},                           {"ADD", Add},
                    {"GIVING", Giving},                   {"DISPLAY", Display},
                    {"IF", If},                           {"THEN", Then},
                    {"END-IF", EndIf},                    {"GO", Go},
                    {"STOP", Stop},                       {"RUN", Run},
                    {"CALL", Call},                       {"USING", Using}
                };
                // contextual check for GO TO vs ADD ... TO
                if (name == "TO" && tk_ == Go) { tk_ = TokTo; return; }

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
                while (*p_ && *p_ != quote && *p_ != '\n') s += *p_++;
                if (*p_ == quote) p_++; else err("unterminated string literal");
                if (d_ - (char*)data_.data() + (long)s.size() + 1 >= (long)(data_.size() * sizeof(long))) {
                    err("out of string memory");
                } else {
                    ival_ = (long)d_;
                    for (char sc : s) *d_++ = sc;
                    *d_++ = '\0';
                    size_t rem = ((size_t)d_ - (size_t)data_.data()) % sizeof(long);
                    if (rem != 0) d_ += (sizeof(long) - rem);
                }
                tk_ = Str;
                return;
            }
            if (c == '=') { tk_ = '='; return; }
            if (c == '<') { tk_ = '<'; return; }
            if (c == '>') { tk_ = '>'; return; }
            tk_ = (unsigned char)c;
            return;
        }
    }

    // ---- Expressions -------------------------------------------------------
    void expr() {
        if (tk_ == Id_) {
            std::string nm = lastName_; Id* id = curId_; next();
            if (id->Class != Glo) { err("undefined variable: " + nm); return; }
            emit(IMM); emit(id->Val); emit(LI);
        } else if (tk_ == Num) {
            emit(IMM); emit(ival_); next();
        } else {
            err("invalid expression"); return;
        }

        int op = tk_;
        if (op == '=' || op == '<' || op == '>') {
            next();
            emit(PUSH);
            if (tk_ == Id_) {
                std::string nm = lastName_; Id* id = curId_; next();
                emit(IMM); emit(id->Val); emit(LI);
            } else if (tk_ == Num) {
                emit(IMM); emit(ival_); next();
            }
            if (op == '=') emit(EQ);
            else if (op == '<') emit(LT);
            else if (op == '>') emit(GT);
        }
    }

    void emitClamp(Id* id) {
        if (id && id->MaxValue > 0) {
            emit(PUSH);
            emit(IMM); emit(id->MaxValue);
            emit(MOD);
        }
    }

    // ---- Statements --------------------------------------------------------
    void moveStmt() {
        next(); // consume MOVE
        bool isLit = false; long litAddr = 0;
        if (tk_ == Str) { isLit = true; litAddr = ival_; next(); }
        else if (tk_ == Num) { emit(IMM); emit(ival_); next(); }
        else if (tk_ == Id_) {
            std::string nm = lastName_; Id* id = curId_; next();
            if (id->Class != Glo) { err("undefined variable: " + nm); return; }
            emit(IMM); emit(id->Val); emit(LI);
        }

        if (tk_ == To) next(); else err("TO expected");
        if (tk_ != Id_) { err("target variable expected"); return; }
        std::string targetNm = lastName_; Id* targetId = curId_; next();
        if (targetId->Class != Glo) { err("undefined variable: " + targetNm); return; }

        if (isLit) {
            emit(IMM); emit(litAddr);
        } else {
            emitClamp(targetId);
        }
        emit(PUSH); emit(IMM); emit(targetId->Val); emit(SI);
    }

    void addStmt() {
        next(); // consume ADD
        if (tk_ == Id_) {
            std::string nm = lastName_; Id* id = curId_; next();
            if (id->Class != Glo) { err("undefined variable: " + nm); return; }
            emit(IMM); emit(id->Val); emit(LI);
        } else if (tk_ == Num) {
            emit(IMM); emit(ival_); next();
        } else {
            err("operand expected after ADD"); return;
        }

        if (tk_ != To) { err("TO expected"); return; }
        next();
        if (tk_ != Id_) { err("variable expected after TO"); return; }
        std::string targetNm = lastName_; Id* targetId = curId_; next();
        if (targetId->Class != Glo) { err("undefined variable: " + targetNm); return; }

        // stack currently holds the first operand; add the TO-variable's
        // current value to it.
        emit(PUSH);
        emit(IMM); emit(targetId->Val); emit(LI);
        emit(ADD);

        if (tk_ == Giving) {
            // "ADD a TO b GIVING c" -- b is left unmodified; the sum goes
            // into c instead.
            next();
            if (tk_ != Id_) { err("variable expected after GIVING"); return; }
            std::string givingNm = lastName_; Id* givingId = curId_; next();
            if (givingId->Class != Glo) { err("undefined variable: " + givingNm); return; }
            emitClamp(givingId);
            emit(PUSH); emit(IMM); emit(givingId->Val); emit(SI);
        } else {
            // "ADD a TO b" -- accumulate into b itself.
            emitClamp(targetId);
            emit(PUSH); emit(IMM); emit(targetId->Val); emit(SI);
        }
    }

    void displayStmt() {
        next(); // consume DISPLAY
        int count = 0;
        while (ok_ && tk_ != DotTok && tk_ != EndIf && tk_ != Eof_) {
            if (count > 0) { emit(SYSC); emit(printSpaceIdx_); emit(0); }
            if (tk_ == Str) {
                emit(IMM); emit(ival_); next();
                emit(PUSH); emit(SYSC); emit(printStrIdx_); emit(1);
            } else if (tk_ == Id_) {
                std::string nm = lastName_; Id* id = curId_; next();
                emit(IMM); emit(id->Val); emit(LI);
                emit(PUSH);
                if (id->MaxValue == 0) {
                    emit(SYSC); emit(printStrIdx_); emit(1);
                } else {
                    emit(SYSC); emit(printNumIdx_); emit(1);
                }
            } else if (tk_ == Num) {
                emit(IMM); emit(ival_); next();
                emit(PUSH); emit(SYSC); emit(printNumIdx_); emit(1);
            }
            count++;
        }
        emit(SYSC); emit(printNlIdx_); emit(0);
    }

    void gotoStmt() {
        next(); // consume GO
        if (tk_ == TokTo) next(); else err("TO expected");
        if (tk_ != Id_) { err("paragraph name expected"); return; }
        emit(JMP);
        long* slot = e_; emit(0);
        pendingJumps_.push_back({slot, lastName_});
        next();
    }

    void ifStmt() {
        next(); // consume IF
        expr();
        if (tk_ == Then) next();
        emit(BZ); long* skip = e_; emit(0);

        while (ok_ && tk_ != EndIf && tk_ != DotTok && tk_ != Eof_) {
            stmt();
        }
        if (tk_ == EndIf) next();
        *skip = off(e_);
    }

    // CALL "NAME" [USING var1 var2 ...] [GIVING result].
    // NAME must be a host-registered syscall (e.g. "GETCH", "KBHIT",
    // "GOTOXY"). Since string literals only carry their data-pool
    // address at this point (not their text, see the lexer above), the
    // name is read back out of the data pool at compile time.
    void callStmt() {
        next(); // consume CALL
        if (tk_ != Str) { err("string literal (program name) expected after CALL"); return; }
        std::string calleeName((const char*)ival_);
        next();

        std::vector<Id*> usingVars;
        if (tk_ == Using) {
            next();
            while (tk_ == Id_) {
                if (curId_->Class != Glo) { err("undefined variable: " + lastName_); return; }
                usingVars.push_back(curId_);
                next();
            }
        }

        Id* givingVar = nullptr;
        if (tk_ == Giving) {
            next();
            if (tk_ != Id_) { err("variable expected after GIVING"); return; }
            if (curId_->Class != Glo) { err("undefined variable: " + lastName_); return; }
            givingVar = curId_;
            next();
        }

        auto it = sym_.find(calleeName);
        if (it == sym_.end() || it->second.Class != Sys) {
            err("unknown CALL target: " + calleeName);
            return;
        }
        long sysIdx = it->second.Val;

        for (Id* v : usingVars) {
            emit(IMM); emit(v->Val); emit(LI);
            emit(PUSH);
        }
        emit(SYSC); emit(sysIdx); emit((long)usingVars.size());

        if (givingVar) {
            emitClamp(givingVar);
            emit(PUSH); emit(IMM); emit(givingVar->Val); emit(SI);
        }
    }

    void stmt() {
        if (tk_ == Move) moveStmt();
        else if (tk_ == Add) addStmt();
        else if (tk_ == Display) displayStmt();
        else if (tk_ == Go) gotoStmt();
        else if (tk_ == If) ifStmt();
        else if (tk_ == Call) callStmt();
        else if (tk_ == Stop) {
            next(); if (tk_ == Run) next(); else err("RUN expected");
            emit(HALT);
        } else {
            err("unknown statement: " + lastName_); next();
        }
    }

    // ---- Divisions ---------------------------------------------------------
    void parseDataDivision() {
        // Caller (compileInternal's dispatch loop) has already consumed
        // "DATA DIVISION." -- next up should be "WORKING-STORAGE SECTION."
        if (!(tk_ == Id_ && lastName_ == "WORKING-STORAGE")) return;
        next();
        if (tk_ == Id_ && lastName_ == "SECTION") next();
        if (tk_ == DotTok) next();

        while (ok_ && tk_ == Num) {
            long level = ival_; next();
            if (level != 1) { err("only level 01 is supported"); return; }
            if (tk_ != Id_) { err("variable name expected"); return; }
            std::string varNm = lastName_; Id& id = idFor(varNm); next();

            id.Class = Glo;
            id.Val = (long)d_;
            d_ += sizeof(long);

            if (tk_ == Pic) {
                next();
                // "9(3)" lexes as separate tokens: Num(9), '(', Num(3), ')'
                // -- NOT as one identifier -- since digits never start an
                // identifier in this lexer. Likewise a bare "X" (string
                // marker) lexes as Id_. Handle both shapes explicitly.
                if (tk_ == Num && ival_ == 9) {
                    next();
                    if (tk_ == '(') {
                        next();
                        if (tk_ != Num) {
                            err("expected a digit count in PIC 9(n)");
                        } else {
                            long digits = ival_;
                            next();
                            if (tk_ == ')') next(); else err("')' expected in PIC clause");
                            // Capped at 9 digits: this VM's `long` may be
                            // only 32 bits on an embedded target, and
                            // 10^9 is comfortably below that limit while
                            // still covering any realistic PIC size.
                            if (digits > 0 && digits <= 9) {
                                long maxVal = 1;
                                for (long i = 0; i < digits; i++) maxVal *= 10;
                                id.MaxValue = maxVal;
                            } else {
                                err("invalid PIC 9(n) size");
                            }
                        }
                    } else {
                        id.MaxValue = 10; // bare "PIC 9" -- one digit, 0-9
                    }
                } else if (tk_ == Id_ && lastName_ == "X") {
                    next();
                    id.MaxValue = 0; // marker for string pointer
                    if (tk_ == '(') { // "X(N)" -- size noted but not enforced
                        next();
                        if (tk_ == Num) next(); else err("expected a size in PIC X(n)");
                        if (tk_ == ')') next(); else err("')' expected in PIC clause");
                    }
                } else {
                    err("unsupported PIC clause (only 9, 9(n), X, X(n) are supported)");
                }
            }
            if (tk_ == Value) {
                next();
                if (tk_ == Num) {
                    *(long*)id.Val = ival_; next();
                } else if (tk_ == Str) {
                    *(long*)id.Val = ival_; next();
                }
            }
            if (tk_ == DotTok) next();
        }
    }

    void parseProcedureDivision() {
        // Caller has already consumed "PROCEDURE DIVISION."

        while (ok_ && tk_ != Eof_) {
            if (tk_ == Id_) {
                const char* savedP = p_; int savedTk = tk_; std::string savedName = lastName_;
                next();
                if (tk_ == DotTok) { // paragraph label check
                    if (labelOffsets_.count(savedName)) err("duplicate paragraph: " + savedName);
                    labelOffsets_[savedName] = off(e_);
                    next(); continue;
                } else { // rollback if not a label
                    p_ = savedP; tk_ = savedTk; lastName_ = savedName; curId_ = &idFor(lastName_);
                }
            }
            stmt();
            if (tk_ == DotTok) next();
        }
    }

    // ---- VM Exec -----------------------------------------------------------
    bool exec() {
        long* pc = text_.data();
        long* sp = stack_.data() + stack_.size();
        long a = 0; long cycles = 0;
        const long MAX_CYCLES = 200L * 1000L * 1000L;
        long* textEnd = text_.data() + text_.size();

        while (pc < textEnd) {
            if (++cycles > MAX_CYCLES) { onError("execution aborted: cycle limit exceeded"); return false; }
            long op = *pc++;
            if (op == 0) break;
            switch (op) {
                case IMM: a = *pc++; break;
                case JMP: pc = text_.data() + *pc; break;
                case BZ:  pc = a ? pc + 1 : text_.data() + *pc; break;
                case PUSH: if (sp <= stack_.data()) { onError("stack overflow"); return false; } *--sp = a; break;
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
                case DIV: if (a == 0) { onError("division by zero"); return false; } a = *sp++ / a; break;
                case MOD: if (a == 0) { onError("division by zero"); return false; } a = *sp++ % a; break;
                case SYSC: {
                    long idx = *pc++; int argc = (int)*pc++;
                    long argsBuf[16];
                    int n = argc < 16 ? argc : 16;
                    for (int i = 0; i < n; i++) argsBuf[i] = sp[argc - 1 - i];
                    if (idx < 0 || idx >= (long)syscalls_.size()) { onError("bad syscall index"); return false; }
                    a = syscalls_[idx](*this, argsBuf, argc);
                    sp += argc;
                    break;
                }
                case HALT: return true;
                default: onError("bad instruction"); return false;
            }
        }
        return true;
    }

    bool compileInternal(const std::string& src) {
        text_.assign(LIVECO_TEXT_SIZE, 0);
        data_.assign(LIVECO_DATA_SIZE, 0);
        stack_.assign(LIVECO_STACK_SIZE, 0);
        labelOffsets_.clear();
        pendingJumps_.clear();
        ok_ = true; line_ = 1;

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

        for (auto& kv : sym_) {
            if (kv.second.Class == Glo || kv.second.Class == Lab) kv.second = Id();
        }

        src_ = src; p_ = src_.c_str(); e_ = text_.data(); d_ = (char*)data_.data();

        next();
        while (ok_ && tk_ != Eof_) {
            if (tk_ == Id_ && lastName_ == "IDENTIFICATION") {
                next();
                if (tk_ == Id_ && lastName_ == "DIVISION") next();
                if (tk_ == DotTok) next();
                continue;
            }
            if (tk_ == Id_ && lastName_ == "ENVIRONMENT") {
                next();
                if (tk_ == Id_ && lastName_ == "DIVISION") next();
                if (tk_ == DotTok) next();
                continue;
            }
            if (tk_ == Id_ && lastName_ == "DATA") {
                next();
                if (tk_ == Id_ && lastName_ == "DIVISION") next();
                if (tk_ == DotTok) next();
                parseDataDivision();
                continue;
            }
            if (tk_ == Id_ && lastName_ == "PROCEDURE") {
                next();
                if (tk_ == Id_ && lastName_ == "DIVISION") next();
                if (tk_ == DotTok) next();
                parseProcedureDivision();
                continue;
            }
            next();
        }

        if (!ok_) return false;

        // resolve forward references
        for (auto& pj : pendingJumps_) {
            auto it = labelOffsets_.find(pj.second);
            if (it == labelOffsets_.end()) {
                err("undefined paragraph: " + pj.second); continue;
            }
            *pj.first = it->second;
        }
        return ok_;
    }
};

```

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


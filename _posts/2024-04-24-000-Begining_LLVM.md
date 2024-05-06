---
layout: post
title: 000 An Introduction to LLVM
date: 2024-04-24 05:24:00
description: LLVM
# tags: formatting
tags: LLVM
---
<!-- 0XX: LLVM Compiler optimization
1XX: Program analysis
2XX: Deep Neural Network -->



*In this tutorials, I will not go through all of theoritical concepts in comiler construction and optimization since the purpose of this article is to provide an introcution to LLVM and implement program analysis from scratch.*
*한국어 번역은 [여기]()에서 보실 수 있습니다.*

# LLVM

- SSA
With LLVM, you can create your own language, optimize a code before translating it into machine code.
LLVM IR (Intermediate Representation) is an representation in the middle of source code and machine so that it can be used for optimization.

## Compilation process
다음은 컴파일 과정을 도식화한 그림이다. LLVM은 AST 로 생성된 IR을 통해 최적화를 수행한다.
- Lexical analysis
- semantic analysis
- optimization

## Installation
- download: github
- build: 간단한 빌드

## Writing an LLVM Pass

1. Create a Pass folder at `./lib/Transformation`

- print hello world
<details open>
<summary>Code</summary>
<br>

```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

namespace {
    struct HelloLLVM : public FunctionPass {
        static char ID;

        HelloLLVM() : FunctionPass(ID) {}

        bool runOnFunction(Function &F) override {
            errs() << "Hello LLVM!";
            return false;
        }
    }; // end of struct
}  // end of anonymous namespace

char HelloLLVM::ID = 0;
static RegisterPass<HelloLLVM> X("PrintHello", "Print Hello LLVM!",
                            false /* Only looks at CFG */,
                            false /* Analysis Pass */);
```
</details>

We just printed 'Hello LLVM!' on your Terminal! But we don't know how it works yet.
- header
```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"
```
We will write an LLVM Pass and this pass run over functions in a program. To print 'Hello LLVM!' out, we need to specify which stream we use. In this case, the output stream is `errs()`

In llvm, a pass has an unique ID so we define `char ID` as static.

After defining struct for a pass, we need to register the pass: `static RigisterPass<YourPass> VarName("OptionName", "Description", "false", "false")`.
This way of registering a pass is called a *legacy pass*. Nowadays, New Pass Manager is used more often.

## LLVM structure
- Module
- Function
- BasicBlock
- Instruction

HelloLLVM 패스는 각 함수마다 최적화("Hello LLVM!" 출력하기)를 수행한다. 이렇듯 최적화 패스를 작성하기 위해서는 LLVM IR의 구조를 알아야 하는데 위 그림에서와 같이 크게 `Module`, `Function`, `BasicBlock`, `Instruction`으로 나눌 수 있다. `Instruction`이 모여 `BasicBlock`을 구성하고, `BasicBlock`이 모여 `Function`을 구성한다. 마지막으로 `Function`이 모여 `Module`을 구성한다. 각 구조는 `loop statement`를 통해 탐색할 수 있으며 이는 다음에 알아보도록 하자.


## Pass Manager
### Legacy pass manager
```cpp
char llvmPass000::ID = 0;
static RegisterPass<llvmPass000> X("command", "description", false, false);
```

### New pass manager
```cpp

```

## Analysis vs Transformation
**Program analysis** invests program whether they have undefined behavior such as misuse of pointer, unused variables, memeory reference, etc. This does not change a target source code but look at it and alarm to users.
On the other hand, **program transformation** not only search the source code but also optimize it by removing unnecessary code snipet or converting it into more efficient code.

A table shown below links to other pages for supplimental analysis and transformation. You can follow the links to dive into LLVM pass.

|Pass|Type|
|---|---|
|[Writing HelloLLVM Pass](../000-Begining_LLVM)|analysis|
|[Iterating over Module, Function, Basic block](../Iterating_over_Module_Function_BasicBlock)|analysis|
|[Count the number of insts, func calls](../002-Count_insts_calls)| analysis|
|[Insert func call](../003-Insert_func_call)|transformation|
|[Change Insts (obfuscation)](../004-Change_Insts_(obfuscation))|transformation|
|[Control flow graph](../005-Traverse_CFG)|transformation|

### Did you enjoy the LLVM tutorial? Now it's time to study **program analysis**. [here!](./2024-03-14-101-Program_analysis_with_LLVM%20copy.md)

## Reference
[1] Andrzej Warzyński. llvm-tutor. [github](https://github.com/banach-space/llvm-tutor)\
[2] Adrian Sampson. LLVM for Grad Students. [blog](https://www.cs.cornell.edu/~asampson/blog/llvm.html)\
[3] Keshav Pingali. CS 380C: Advanced Topics in Compilers. [blog](https://www.cs.utexas.edu/~pingali/CS380C/2020/assignments/assignment4/index.html)\
[4] Arthur Eubanks. The New Pass Manager. [blog](https://blog.llvm.org/posts/2021-03-26-the-new-pass-manager)

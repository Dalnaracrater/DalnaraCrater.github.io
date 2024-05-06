---
layout: post
title: 001 Iterating Module, Function, and Basicblock with LLVM
date: 2024-04-24 05:24:00
description: LLVM
# tags: formatting
tags: LLVM
---
<!-- 0XX: LLVM Compiler optimization
1XX: Program analysis
2XX: Deep Neural Network -->

# Iterating Function, Basicblock, and Instruction with LLVM

As we discussed previous [post](../000-Begining_LLVM(KOR)), LLVM Structure consists of Modules, functions, basicblocks, and instructions. Therefore, we can search for one of them through its super set. For example, if you are looking for an `add` instruction, you can find it by investigating BasicBlocks.

### Learning Objectives
- Learn how to iterate LLVM structures(Module, Function, BasicBlock, Instruction)

### LLVM

### Iterating over Functions

### Iterating over Basicblocks
`llvm/IR/BasicBlock.h` header is deprecated. So you may include `Function.h` or `Module.h` when you iterate BasicBlocks.

You can iterate BasicBlocks in a function F.

```cpp
for(BasicBlock &BB: F){
    errs() << "Start BB: " << BB.getName() << ' ' << BB.size() << '\n';
}
```

### Iterating over Instructions
You can iterate instructions in a BasicBlocks BB.

```cpp
for(Instruction &I: BB){
    errs() << "Start Inst: " << I << '\n';
}
```

```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Instructions.h"

#include "llvm/Support/raw_ostream.h"

using namespace llvm;

namespace {
    struct YJ00003BB : public FunctionPass{
        static char ID;
        YJ00003BB() : FunctionPass(ID){}

        bool runOnFunction(Function &F) override{
            // Function
            errs() << "Start Function: " << F.getName() << '\n';

            for(BasicBlock &BB: F){
                errs() << "Start BB: " << BB.getName() << ' ' << BB.size() << '\n';

                for(Instruction & I : BB){
                    errs() << "Start Inst: " << I << '\n';
                }
            }

            return false;
        }
    };
}

char YJ00003BB::ID = 0;
static RegisterPass<YJ00003BB> X("BasicBlock", "Print basic block", false, false);
```

|Pass|Type|
|---|---|
|[Writing HelloLLVM Pass](../000-Begining_LLVM(KOR))|analysis|
|[Iterating over Module, Function, Basic block](../Iterating_over_Module_Function_BasicBlock)|analysis|
|[Count the number of insts, func calls](../002-Count_insts_calls)| analysis|
|[Insert func call](../003-Insert_func_call)|transformation|
|[Change Insts (obfuscation)](../004-Change_Insts_(obfuscation))|transformation|
|[Control flow graph](../005-Traverse_CFG)|transformation|

---
## Reference
[1] Andrzej Warzyński. llvm-tutor. [github](https://github.com/banach-space/llvm-tutor)\
[2] Adrian Sampson. LLVM for Grad Students. [blog](https://www.cs.cornell.edu/~asampson/blog/llvm.html)\
[3] Keshav Pingali. CS 380C: Advanced Topics in Compilers. [blog](https://www.cs.utexas.edu/~pingali/CS380C/2020/assignments/assignment4/index.html)
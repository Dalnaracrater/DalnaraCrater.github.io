---
layout: post
title: 004 Change Instructions (obfuscation)
date: 2024-04-24 05:24:00
description: LLVM
# tags: formatting
tags: LLVM
---
<!-- 0XX: LLVM Compiler optimization
1XX: Program analysis
2XX: Deep Neural Network -->

*본 튜토리얼에서는 프로그램 분석 기법을 LLVM을 통해 구현해보는 것이 목적이기 때문에 모든 이론적 지식을 자세하게 다루지는 않을 것이다.*

### Learning Objectives
- learn a definition of obfuscation
- learn obfuscation technuques
- learn how to change instructions

## What is obfuscation?

## How can we change instructions?

## implementation
```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Instruction.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/IR/IRBuilder.h"

using namespace llvm;

namespace{
    struct YJ008MbaAdd : public FunctionPass{
        static char ID;
        YJ008MbaAdd() : FunctionPass(ID){};

        bool runOnFunction(Function& F) override {
            bool isModified = false;

            for(auto& BB: F){

                for(auto I = BB.begin(); I != BB.end(); ++I){
                    auto *op = dyn_cast<BinaryOperator>(I);

                    if (!op) continue;

                    if (op->getOpcode() != Instruction::Add || !op->getType()->isIntegerTy())
                        continue;
                    
                    IRBuilder<> Builder(op);
                    //a + b == (((a ^ b) + 2 * (a & b)) * 39 + 23) * 151 + 111
                    Instruction* v = BinaryOperator::CreateAdd(Builder.CreateMul(
                                                            Builder.CreateAdd(Builder.CreateXor(op->getOperand(0), op->getOperand(1)),
                                                                Builder.CreateAdd(Builder.CreateMul(ConstantInt::get(op->getType(), 2),
                                                                                    Builder.CreateMul(Builder.CreateAnd(op->getOperand(0), op->getOperand(1)),
                                                                                ConstantInt::get(op->getType(), 39))),
                                                                    ConstantInt::get(op->getType(), 23))),
                                                            ConstantInt::get(op->getType(), 151)),
                                                        ConstantInt::get(op->getType(), 111));
                    
                    ReplaceInstWithInst(op->getParent()->getInstList(), I, v);

                    isModified = true;
                }

            }

            return isModified;
        }
    };
}

char YJ008MbaAdd::ID = 0;
static RegisterPass<YJ008MbaAdd>X ("MbaAdd", "Replace add with MBA add", false, true);
```

```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

#include "llvm/Support/raw_ostream.h"

using namespace llvm;

namespace{
    struct YJ008MbaSub : public FunctionPass{
        static char ID;
        YJ008MbaSub() : FunctionPass(ID){}

        bool runOnFunction(Function& F) override {
            bool isModified = false;
            for(auto& BB : F){
                
                for(auto I = BB.begin(); I != BB.end(); ++I){
                    // errs() << I << '\n';
                    auto *op = dyn_cast<BinaryOperator>(I);
                    if(!op) continue;

                    // skip inst if opcode is not Sub or of Integer type
                    unsigned opcode = op->getOpcode();
                    if (opcode != Instruction::Sub || !op->getType()->isIntegerTy()) continue;

                    IRBuilder<> Builder(op);
                    // create inst: (a + ~b) + 1
                    Instruction *v = BinaryOperator::CreateAdd(
                        Builder.CreateAdd(op->getOperand(0), Builder.CreateNot(op->getOperand(1))),
                        ConstantInt::get(op->getType(), 1));

                    errs() << op << '\n';

                    // replace inst sub -> (a + ~b) + 1
                    ReplaceInstWithInst(op->getParent()->getInstList(), I, v);
                    // errs() << "HI";

                    isModified = true;
                }
            }

            return isModified;
        }
    };
}

char YJ008MbaSub::ID = 0;
static RegisterPass<YJ008MbaSub>X("MbaSub", "replace Sub with Mab", false, true);
```


|Pass|Type|
|---|---|
|[Writing HelloLLVM Pass](../000-Begining_LLVM(KOR))|analysis|
|[Iterating over Module, Function, Basic block](../Iterating_over_Module_Function_BasicBlock)|analysis|
|[Count the number of insts, func calls](../002-Count_insts_calls)| analysis|
|[Insert func call](../003-Insert_func_call)|transformation|
|[Change Insts (obfuscation)](../004-Change_Insts_(obfuscation))|transformation|
|[Control flow graph](../005-Traverse_CFG)|transformation|

## Reference
[1] Andrzej Warzyński. llvm-tutor. [github](https://github.com/banach-space/llvm-tutor)\
[2] Adrian Sampson. LLVM for Grad Students. [blog](https://www.cs.cornell.edu/~asampson/blog/llvm.html)\
[3] Keshav Pingali. CS 380C: Advanced Topics in Compilers. [blog](https://www.cs.utexas.edu/~pingali/CS380C/2020/assignments/assignment4/index.html)\
[4] Arthur Eubanks. The New Pass Manager. [blog](https://blog.llvm.org/posts/2021-03-26-the-new-pass-manager)

---
layout: post
title: 003 Insert function calls
date: 2024-04-24 05:24:00
description: LLVM
# tags: formatting
tags: LLVM
---
<!-- 0XX: LLVM Compiler optimization
1XX: Program analysis
2XX: Deep Neural Network -->

### Learning Objectives
- learn how a function is called in a memory
- learn how to generate function call with parameters



### How do CPU/compiler call a function?
#### printf


### Insert function call

```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/IRBuilder.h"

#include "llvm/Support/raw_ostream.h"

#include "llvm/IR/Module.h"

using namespace llvm;

namespace {
    struct YJ005InjectFuncCall : public ModulePass {
        static char ID;
        YJ005InjectFuncCall() : ModulePass(ID){}

        bool runOnModule(Module &M) override{
            LLVMContext & context = M.getContext();

            // creating a pointer Int8 type
            PointerType *PrintfArgTy = PointerType::getUnqual(Type::getInt8Ty(context));

            // step1: Inject declaration of printf
            FunctionType *PrintfTy = FunctionType::get(
                IntegerType::getInt32Ty(context),
                PrintfArgTy,
                true// variadic
            );

            FunctionCallee Printf = M.getOrInsertFunction("printf", PrintfTy);

            Function *PrintfF = dyn_cast<Function>(Printf.getCallee());
            PrintfF->setDoesNotThrow();
            PrintfF->addParamAttr(0, Attribute::NoCapture);
            PrintfF->addParamAttr(0, Attribute::ReadOnly);

            // step2: Inject global variable that will hold the printf format string
            Constant *PrintfFormatStr = ConstantDataArray::getString(
                context, "Function is %s, num of arg: %d\n");

            Constant *PrintfFormatStrVar = M.getOrInsertGlobal("PrintfFormatStr", PrintfFormatStr->getType());
            dyn_cast<GlobalVariable>(PrintfFormatStrVar)->setInitializer(PrintfFormatStr);

            // GlobalVariable *PrintfFormatStrVar = new GlobalVariable(M, PrintfFormatStr->getType(), true, // isConstant
            //     GlobalValue::PrivateLinkage, PrintfFormatStr, "PrintfFormatStr");

            bool modified = false;
            // step3: for each function in the module, inject a call to printf
            for(auto& F : M){
                if (!F.isDeclaration()){
                    // errs() << &*F.getEntryBlock().getFirstInsertionPt();

                    IRBuilder<> Builder(&*F.getEntryBlock().getFirstInsertionPt());

                    Value *FormatStrPtr = Builder.CreatePointerCast(PrintfFormatStrVar, PrintfArgTy, "formatStr");

                    auto funcName = Builder.CreateGlobalStringPtr(F.getName());
                    
                    // errs() << F.getName() << ' ' << Builder.getInt32(F.arg_size()) << " " << F.arg_size() << '\n';

                    Builder.CreateCall(Printf, {FormatStrPtr, funcName, Builder.getInt32(F.arg_size())});
                }


                for(auto& BB: F){
                    for(auto& I: BB){
                        // errs() << I << '\n';
                    }
                }

                modified = true;
            }

            return modified;
        }
    };
}

char YJ005InjectFuncCall::ID = 0;
static RegisterPass<YJ005InjectFuncCall> X("InjectFunc", "Inject function call", false, true);

```





|Pass|Type|
|---|---|
|[Writing HelloLLVM Pass](../000-Begining_LLVM(KOR))|analysis|
|[Iterating over Module, Function, Basic block](../Iterating_over_Module_Function_BasicBlock)|analysis|
|[Count the number of insts, func calls](../002-Count_insts_calls)| analysis|
|[Insert func call](../003-Insert_func_call)|transformation|
|[Change Insts (obfuscation)](../004-Change_Insts_(obfuscation))|transformation|
|[Control flow graph](../005-Traverse_CFG)|transformation|

#### Did you enjoy the LLVM tutorial? Now it's time to study **program analysis**. [here!](./2024-03-14-101-Program_analysis_with_LLVM%20copy.md)

## Reference
[1] Andrzej Warzyński. llvm-tutor. [github](https://github.com/banach-space/llvm-tutor)\
[2] Adrian Sampson. LLVM for Grad Students. [blog](https://www.cs.cornell.edu/~asampson/blog/llvm.html)\
[3] Keshav Pingali. CS 380C: Advanced Topics in Compilers. [blog](https://www.cs.utexas.edu/~pingali/CS380C/2020/assignments/assignment4/index.html)\
[4] Arthur Eubanks. The New Pass Manager. [blog](https://blog.llvm.org/posts/2021-03-26-the-new-pass-manager)

---
layout: post
title: 002 Count the number of instructions and function calls
date: 2024-04-24 05:24:00
description: LLVM
# tags: formatting
tags: LLVM
---
<!-- 0XX: LLVM Compiler optimization
1XX: Program analysis
2XX: Deep Neural Network -->

## To be continued...

### Learning Objectives
- Learn how to handle instructions
- learn how to generate instruction (codegen)
- learn the difference between static analysis and dynamic analysis

### isa<>, dyn_cast<>

### (static) Counting instructions
```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Instructions.h"

#include "llvm/Support/raw_ostream.h"

#include <map>
#include <string>

using namespace llvm;

namespace {
    struct YJ004OpC : public FunctionPass {
        static char ID;
        YJ004OpC() : FunctionPass(ID){}

        bool runOnFunction(Function& F){
            std::map<std::string, int> opcodeMap; // OpcodeCounter::Result in OpcodeCounter.h
            // iterate function
            for(auto &BB: F){
                for(Instruction & I : BB){
                    if (opcodeMap.find(I.getOpcodeName()) != opcodeMap.end()){
                        opcodeMap[I.getOpcodeName()] = opcodeMap[I.getOpcodeName()] + 1; 
                    }else{
                        opcodeMap[I.getOpcodeName()] = 1;
                    }
                }
            }

            errs() << "OpCode Info\n";
            for(auto Opcode : opcodeMap){
                errs() << Opcode.first << ' ' << Opcode.second << '\n';
            }

            return false;
        }
    };
}

char YJ004OpC::ID = 0;
static RegisterPass<YJ004OpC> X("OpCount", "Count OpCode", false, false);
```

### counting function calls
```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/CallSite.h"

#include "llvm/Support/raw_ostream.h"

using namespace llvm;

/*
In M, define a global var for count of func call with 0
for each F in M
    increment count by 1, so that the number of call in which function called is recorded

*/

namespace {
    struct YJ007StaticCallCounter : public ModulePass{
        static char ID;

        YJ007StaticCallCounter(): ModulePass(ID){}

        bool runOnModule(Module& M) override {
            LLVMContext& context = M.getContext();

            // llvm map
            int count = 0;

            bool isModified = false;
            for(auto& F: M){
                // add Inst into the beginning of the func
                // increment var
                IRBuilder<> Builder(&*F.getEntryBlock().getFirstInsertionPt());

                for(auto& BB: F){
                    for (auto& I : BB){
                        //dyn_cast<>, isa<>, cast<>
                        // if auto inst = I == call
                        // ++ count into map[F.getName()];
                        if (isa<CallInst>(I)){
                            ++count;
                        }
                    }
                }
            }

            if (isModified){
                // print map
            }

            return isModified;
        }
    };
}

char YJ007StaticCallCounter::ID = 0;
static RegisterPass<YJ007StaticCallCounter> X("StaticCallCounter", "Count Static Call", false, false);
```

### (dynamic) Counting instructions
```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Instruction.h"
#include "llvm/Support/raw_ostream.h"

#include "llvm/Transforms/Utils/ModuleUtils.h"

#include "llvm/IR/IRBuilder.h"

using namespace llvm;

namespace{
    struct YJ006DynCount : public ModulePass{
        static char ID;
        YJ006DynCount() : ModulePass(ID){}


        Constant *createGlobalVariable(Module& M, StringRef varName){

            auto& context = M.getContext();

            // define global variable
            Constant *gVar = M.getOrInsertGlobal(varName, Type::getInt32Ty(context));

            // get a reference through getNamedGlobal() if it exists.
            // set some properties: linkage, initial value
            GlobalVariable * gVarRef = M.getNamedGlobal(varName);
            gVarRef->setLinkage(GlobalValue::LinkageTypes::CommonLinkage);
            gVarRef->setAlignment(MaybeAlign(4));
            gVarRef->setInitializer(ConstantInt::get(context, APInt(32, 0)));
 
            return gVar;
        }

        bool runOnModule(Module &M) override{
            // global var
            LLVMContext &context = M.getContext();

            StringMap<Constant*> funcCountMap;
            StringMap<Constant*> funcNameMap;

            bool modified = false;
            for(auto& F: M){
                if(F.isDeclaration())
                    continue;

                IRBuilder<> Builder(&*F.getEntryBlock().getFirstInsertionPt());

                std::string varName = "Counter_" + std::string(F.getName());
                //define global variable via createGlobalVariable() method
                Constant *counter = createGlobalVariable(M, varName);

                funcCountMap[F.getName()] = counter;

                Constant *funcName = Builder.CreateGlobalStringPtr(F.getName());
                funcNameMap[F.getName()] = funcName;

                /*
                load counter
                add counter by 1
                store counter
                */
                LoadInst *instLoad = Builder.CreateLoad(IntegerType::getInt32Ty(context), counter);
                Value* instAdd = Builder.CreateAdd(Builder.getInt32(1), instLoad);
                Builder.CreateStore(instAdd, counter);

                modified = true;
            }

            if (!modified) return modified;

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
                context, "Function %s is called %d times\n");

            Constant *PrintfFormatStrVar = M.getOrInsertGlobal("PrintfFormatStr", PrintfFormatStr->getType());
            dyn_cast<GlobalVariable>(PrintfFormatStrVar)->setInitializer(PrintfFormatStr);

            FunctionType *PrintWrapperTy = FunctionType::get(Type::getVoidTy(context), {}, false);
            Function *PrintWrapperF = dyn_cast<Function>(M.getOrInsertFunction("PrintWrapper", PrintWrapperTy).getCallee());

            BasicBlock *retBlock = BasicBlock::Create(context, "enter", PrintWrapperF);
            IRBuilder<> retBuilder(retBlock);

            Value *FormatStrPtr = retBuilder.CreatePointerCast(PrintfFormatStrVar, PrintfArgTy, "formatStr");

            for(auto& name : funcCountMap){
                auto instLoad = retBuilder.CreateLoad(IntegerType::getInt32Ty(context), name.second);

                retBuilder.CreateCall(Printf, {FormatStrPtr, funcNameMap[name.first()], instLoad});
            }

            retBuilder.CreateRetVoid();

            appendToGlobalDtors(M, PrintWrapperF, 0);

            return true;
        }        
    };
}


char YJ006DynCount::ID = 0;
static RegisterPass<YJ006DynCount> X("DynCount", "Count Dynamic Call", false, false);
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

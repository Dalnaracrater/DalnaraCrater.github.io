---
layout: post
title: 000 An Introduction to LLVM(KOR)
date: 2024-04-24 05:24:00
description: LLVM
# tags: formatting
tags: LLVM
---
<!-- 0XX: LLVM Compiler optimization
1XX: Program analysis
2XX: Deep Neural Network -->

GPU의 등장으로 인공지능 연구에 엄청난 발전이 시작되었다. 스팸메일 탐지, MRI 영상 처리, 심지어는 잡초 이미지를 학습하여 농기구에 부착된 카메라를 통해 잡초를 인식하고 레이저로 제거하는 기술까지 모든 분야에서 인공지능 기술을 활용하고 있다. 또한 불과 2~3년 사이에 ChatGPT, LLaMa, Bard와 같은 트랜스포머(transformer) 기반의 모델은 이미 우리의 삶 어딘가에 자리잡기 시작했다. 잠깐! LLVM 튜토리얼 아니었나? 맞다! 근데 왜 갑자기 인공지능 이야기를 하는가? 왜냐하면 컴파일러 또한 인공지능과 관련이 있기 때문이다! 인공지능 모델 또한 결국 CPU, GPU, TPU에서 처리해야 하기 때문에 더 빠르고 최적화된 컴파일러를 필요로 한다. MLIR, XLA, Triton과 같이 특정 도메인에서 유용한 컴파일러 프레임워크가 등장하였으며, 이 프레임워크를 통해 딥러닝 모델의 학습을 가속화시킬 수 있다.\
LLVM은 좀 더 범용적으로 사용할 수 있는 컴파일러 프레임워크 중 하나이지만 다소 높은 진입 장벽을 갖고 있으며, 나 또한 이러한 장벽에 좌절을 했던 한 학생이었다. 하지만 이제는 제법 LLVM에 대해 잘 알고 있으며 이론으로만 배웠던 프로그램 분석 기법들이 어떻게 구현이 되고 적용되는지 알아냈다. 나는 나와 같은 어려움을 겪고 있는 학생들을 위해 간단한 튜토리얼을 작성해본다.

*본 튜토리얼에서는 프로그램 분석 기법을 LLVM을 통해 구현해보는 것이 목적이기 때문에 모든 이론적 지식을 자세하게 다루지는 않을 것이다.*

## LLVM이란?
LLVM은 프로그래밍 언어를 설계하고 최적화할 수 있게 해주는 컴파일러 도구이다. (Front-end) (Middle-end) 변환된 LLVM IR을 최적화하여 새로운 LLVM IR을 출력한다. (Back-end) 최적화된 IR 코드는 기계에 종속적인 실행 파일로 변환된다.

### Learning Objectives
- 컴파일 과정에 대해 이해한다
- SSA 형태가 무엇인지 공부한다
- LLVM 프로젝트를 설치하고 빌드하는 방법을 알아본다
- LLVM Pass를 작성하는 법을 살펴본다.
- LLVM IR로 변환된 프로그램의 기본 구조를 살펴본다.

### Compilation process
컴파일 과정을 이해하면 LLVM을 사용하는데 큰 도움이 된다. 흔히 빌드 과정과 컴파일 과정을 헷갈려하는데 이에 대해 간단하게 짚고 넘어가고자 한다.\
먼저 빌드 과정을 살펴보자. 소스코드 빌드는 보통 전처리 -> 컴파일 -> 어셈블 -> 링크 순서로 진행되어 최종적으로 프로그램이 실행될 기계에서 실행할 수 있는 실행파일이 생성된다. 우리가 앞으로 다루고자 하는 최적화는 빌드 과정의 컴파일에서 수행된다.

다음은 컴파일 과정을 도식화한 그림이다.

컴파일 과정은 크게 전단부와 후단부로 나뉘며 전단부를 좀 더 세밀하게 나누면 전단부와 중단부로 다시 나눌 수 있다. 전단부에서는 어휘분석, 구문분석, 의미분석을 수행하고, 중단부에서는 최적화를, 후단부에서는 기계어로의 변환 작업을 수행한다.

- Front-end: Convert a high level programming langauge into LLVM IR
    - Lexical analysis: Scan a source code and check whether it follows designated patterns by tokens. Return a series of tokens, otherwise omit an error.
    - syntax analysis: Given a series of tokens, create a Abstract Syntax Tree (AST) 
    - semantic analysis: analyze a source code whether it has semantic errors or not. (e.g. Type check, Arithmetics) Return Abstract Syntax Tree (AST)
- Middle-end: Optimize LLVM IR via LLVM Pass
    - optimization: optimize a source code.
- Back-end: Translate a optimized IR into a machine dependent assembly language.



### SSA
LLVM이 소스코드를 최적화하기 위해서는 gcc, llc와 같은 컴파일 도구를 이용해 소스코드를 LLVM IR로 변환해야 한다. 이때 LLVM IR은 Static Single Assignment (SSA) 형태를 따르는데, 이는 변수에 값이 오직 한번만 할당되고, 그 이후에는 그 변수를 읽는데만 사용할 수 있도록 한다. SSA 형태로 코드를 변환하면 코드 최적화에 도움이 되는데 이는 그래프 기반의 최적화와 같이 흐름 분석을 하는데 유용하기 때문이다.

- Fig


자, 이제 LLVM에 대한 기본적인 개념을 소개했으니 어떻게 사용하는지 알아보도록 하자.

### Installing LLVM
[LLVM 공식 홈페이지](https://llvm.org/docs/GettingStarted.html)에서 LLVM 설치에 대한 더 자세한 설명이 있으니 참고하면 좋을 것 같다.

#### required software
`CMake`: `brew install CMake`\
`Ninja`: `brew install Ninja`\
`Clang`: `brew install Clang`

#### Clone Git
```
git clone https://github.com/llvm/llvm-project.git llvm
cd llvm
```

#### Build LLVM project
```
mkdir build
cd build
ninja
```

Ninja를 통해 프로젝트를 처음 빌드하게 되면 대략 30분 정도의 시간이 소요된다. 하지만 이후에 다시 빌드를 하게 되면 몇 분 내외로 빌드가 완료되니 걱정하지 말자.

빌드된 llvm의 실행파일을 손쉽게 사용하기 위해 실행환경을 수정해 준다. *(프로그램이 실행되지 않을 때 뿐만 아니라 LLVM 패스를 작성할 때 라이브러리가 인식이 안되는 문제도 해결해준다.)*

```
export PATH=$PATH:~/llvm/build/bin/
```

#### Compile a source code into *.ll or *.bc
```
clang -emit-llvm -S test.cpp -o test.ll
```

### Writing an LLVM Pass

처음으로 만들어 볼 패스는 "Hello LLVM!"을 출력하는 Hello 패스이다.

1. 우선 LLVM 패스 파일을 위한 `HelloLLVM` 디렉토리를 `./llvm/llvm/lib/Transforms/`에 만들자.

```
mkdir HelloLLVM
cd HelloLLVM
```
###### *주의. 현재 우리가 위치한 디렉토리는 `~/llvm/llvm/lib/Transforms/HelloLLVM/`이고 패스 파일을 빌드하면 `~/llvm/build/lib/`에 빌드된 패스가 생성된다*

패스를 작성하기 위해서는 빌드를 위한 `CMakeLists.txt`와 LLVM 소스코드 파일, `HelloLLVM.cpp`이 필요하다.

```
touch CMakeLists.txt
touch HelloLLVM.cpp
```

`CMakeLists.txt` 파일에 들어갈 내용은 다음과 같다.
```
add_llvm_library(HELLO MODULE
    HELLO.cpp
    
    PLUGIN_TOOL
    OPT)
```

`Transforms` 디렉토리에 위치한 `CMakeLists.txt`에도 맨 마지막에 `add_subdirectory(HelloLLVM)`을 추가해주자.

자 이제 진정한 LLVM Pass를 작성해보자!

#### library header
우리가 작성하려는 **패스** 파일은 소스코드를 **함수 단위**로 분석하고 이때 "Hello LLVM!"을 **출력**하는 것이다. 따라서 이에 맞게 사용할 라이브러리는 다음과 같다.
```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"
```

llvm은 이름공간을 사용하므로 편의를 위해 다음을 선언해주자.
```cpp
using namespace llvm;
```

다음으로는 새로운 이름 공간 안에 FunctionPass를 상속받는 HelloLLVM 구조체를 선언하자. 이 구조체의 멤버 함수가 실질적인 최적화 작업을 수행할 것이다.

```cpp
namespace{
    struct HelloLLVM: public FunctionPass{
        // You will write a Pass here! 
    };
}
```

llvm은 패스마다 고유의 `ID`가 있기 때문에 `ID`를 스태틱 변수로 선언해준다. `FunctionPass`의 생성자를 통해 고유한 ID값을 갖게 하고 다른 패스들과 구분할 수 있게 만들어준다.

```cpp
static char ID;
HelloLLVM() : FunctionPass(ID) {}
```

소스코드를 함수 단위로 분석하기 위해 `bool runOnFunction()` 함수를 재정의(override)한다. 이 함수는 소스코드를 변경했으면 `true`를 아니면 `false`를 반환한다.
```cpp
bool runOnFunction(Function& F){
    // this code optimize a given source code over the function F.

    return false;
}
```

`C`언어의 `printf`, `C++`의 `cout`과 같이 llvm에서는 `errs()`를 출력 스트림으로 사용한다.
```cpp
errs() << "Hello LLVM!";
```

We will write an LLVM Pass and this pass run over functions in a program. To print 'Hello LLVM!' out, we need to specify which stream we use. In this case, the output stream is `errs()`

In llvm, a pass has an unique ID so we define `char ID` as static.

최적화를 위한 패스를 완성하였다. 이제는 이 패스를 등록하는 일만 남았다. 먼저 패스의 `ID`를 `0`으로 설정해주고(아무런 의미가 없기 때문), `RegisterPass`를 통해 패스를 등록한다. llvm에서는 패스가 고유의 ID를 갖지만, 이 고유성은 `ID`가 `0`값을 갖을 때 할당되는 메모리 공간을 통해 다른 패스들과 구분되게 한다.

```cpp
char HelloLLVM::ID = 0;
static RegisterPass<HelloLLVM> X("PrintHello", "Print Hello LLVM!",
                            false /* Only looks at CFG */,
                            false /* Analysis Pass */);
```

위 방식은 **legacy Pass Manager**를 통해 패스를 등록한 것이다. 이 방식은 스케줄링, 메모리 관리 등에 여러 불편함이 있었는데 이를 해소하기 위해 **New Pass Manager**가 등장했다. 그래서 최근에는 이 **New Pass Manger**를 통해 패스를 등록하는 추세이다.

자 이제 패스 파일을 완성했으니 빌드하고 실제 코드에 적용해보자.

먼저 `Ninja`를 통해 llvm을 빌드한다. 첫 빌드와 달리 단 몇 십 초만에 빌드가 끝난다.
```
cd ~/llvm/build/
ninja
```

`hello.cpp` 파일이 있는 `~/llvm/testcases/`로 이동하자. *(다른 곳에서 실행해도 된다.)*
```
cd ~/llvm/testcases/
clang -emit-llvm -S hello.cpp -o hello.ll
opt -load ../build/lib/HelloLLVM.so < hello.ll > /dev/null
```

혹시 `HelloLLVM.so`가 없는가? 운영체제에 따라 생성되는 파일이 다르니 다시 한번 살펴보자. 필자는 Mac OS를 사용하기 때문에 `*.dylib`가 생성된다.

```
Hello LLVM!
```

자 방금 "Hello LLVM!"을 터미널에 출력하였다. 이는 hello.ll 파일을 최적화 하는 과정에서 해당 메시지를 출력한 것이다. LLVM의 세계에 온 것을 환영한다!

#### Full code
```cpp
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


### LLVM structure
- Module
- Function
- BasicBlock
- Instruction

HelloLLVM 패스는 각 함수마다 최적화("Hello LLVM!" 출력하기)를 수행한다. 이렇듯 최적화 패스를 작성하기 위해서는 LLVM IR의 구조를 알아야 하는데 위 그림에서와 같이 크게 `Module`, `Function`, `BasicBlock`, `Instruction`으로 나눌 수 있다. `Instruction`이 모여 `BasicBlock`을 구성하고, `BasicBlock`이 모여 `Function`을 구성한다. 마지막으로 `Function`이 모여 `Module`을 구성한다. 각 구조는 `loop statement`를 통해 탐색할 수 있으며 이는 다음에 알아보도록 하자.

### Pass Manager
Legacy pass manager은 
#### Legacy pass manager
```cpp
char llvmPass000::ID = 0;
static RegisterPass<llvmPass000> X("command", "description", false, false);
```

#### New pass manager
```cpp

```

### Analysis vs Transformation
**프로그램 분석(Program analysis)**은 프로그램을 수정하지 않는 선에서 수행할 수 있는 분석 기법을 말한다. 이러한 분석은 정의하지 않는 행위(undefined behavior)를 찾아내는데 자주 사용되며, 포인터 오용(misuse of pointer), 사용하지 않는 변수 찾기(unused variable), 메모리 참조(memory reference) 등이 있다.\
반면에 **프로그램 변환(Program transform)**은 프로그램을 조사하는 것 뿐만 아니라, 프로그램을 수정하여 올바르게 동작하도록 하거나, 같은 동작을 수행하지만 좀 더 빨르게 해주는 최적화 작업에 사용한다. 죽은 코드 제거(dead code elimination), 상수 전파(constant propagation) 등이 프로그램 변환의 예시라 할 수 있다.

프로그램 분석을 위해서는 호출 그래프(call graph)를 탐색하거나, 때로는 명령어를 다른 명령어로 변환하는 작업을 수행해야 한다. 이를 위해서는 LLVM IR을 잘 다룰 수 있어야 하는데, 아래 표는 LLVM IR을 다룰 수 있는 여러 방법들을 설명해주고 있으니 좀 더 살펴보도록 하자.

|Pass|Type|
|---|---|
|[Writing HelloLLVM Pass](../000-Begining_LLVM(KOR))|analysis|
|[Iterating over Module, Function, Basic block](../001-Iterating_over_Module_Function_BasicBlock)|analysis|
|[Count the number of insts, func calls](../002-Count_insts_calls)| analysis|
|[Insert func call](../003-Insert_func_call)|transformation|
|[Change Insts (obfuscation)](../004-Change_Insts_(obfuscation))|transformation|
|[Control flow graph](../005-Traverse_CFG)|transformation|

LLVM 튜토리얼을 잘 즐기셨나요? 이제는 프로그램 분석에 대해 알아보도록 합시다! [여기](./101-Program_analysis_with_LLVM)로 오세요!

## Reference
[1] Andrzej Warzyński. llvm-tutor. [github](https://github.com/banach-space/llvm-tutor)\
[2] Adrian Sampson. LLVM for Grad Students. [blog](https://www.cs.cornell.edu/~asampson/blog/llvm.html)\
[3] Keshav Pingali. CS 380C: Advanced Topics in Compilers. [blog](https://www.cs.utexas.edu/~pingali/CS380C/2020/assignments/assignment4/index.html)\
[4] Arthur Eubanks. The New Pass Manager. [blog](https://blog.llvm.org/posts/2021-03-26-the-new-pass-manager)
[5] Changbie. Compilation Optimization: LLVM Code Generation Technology Details and Its Application in Databases. [post](https://www.alibabacloud.com/blog/compilation-optimization-llvm-code-generation-technology-details-and-its-application-in-databases_598408)
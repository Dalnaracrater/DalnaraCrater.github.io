---
layout: post
title: Program Synthesis
date: 2020-02-08 18:00:00 -0400
tags: Research Programming-language
featured: true
---

프로그램 합성 (Program synthesis)는 논리식으로 표현된 명세를 만족하는 프로그램을 생성하는 자동 생성 기법이다.

## 개요
많은 기술의 발전에도 불구하고, 아직까지 프로그래머들은 손으로 직접 코드를 작성해야만 한다. 비록 프로그래밍 언어들이 user-friendly하게 발전하였다 하더라도, 프로그래밍하는 방법을 배우는 것은 코딩하는데 있어서 상당한 노력을 기울여야한다.

>Despite the many advances in computing over the past decades, the actual process of writing computer software has not fundamentally changed — a programmer must manually code the exact algorithmic logic of a program in a step-by-step manner using a specialized programming language. Although programming languages have become much more user-friendly over the years, learning how to program is still a major endeavor that most computer users have not undertaken.

이러한 이유로 프로그래밍 자동 생성 기법에 대한 연구가 활발히 진행되고 있으며 대표적으로 program synthesis와 reduction이 있다.

## Program Synthesis?

프로그램 합성은 쉽게 말해 아래 그림과 같이 주어진 프로그램 공간(program space) 속에서 논리 명세를 만족하는 프로그램을 탐색하는 것이라고 생각할 수도 있다. 

![program_tree](https://user-images.githubusercontent.com/24788751/86518342-300b1700-be6c-11ea-8813-3780fdaacc6e.JPG)

프로그램 공간은 DSL (Domain specific language)를 따라 생성되는 후보 프로그램들로 구성된다. (DSL을 통해 구체적인 언어 설계를 알 필요가 없다.)

## Synthesis 과정
DSL에 따라 다르겠지만 일반적으로 프로그램 합성은 결정적이지 않다 (Non-deterministic). 그렇기 때문에 프로그램 공간의 크기는 무한이 될 수 있으며 후보 프로그램의 수는 이론적으로 n^n이다. 이러한 특징을 통해 프로그램 합성을 조합론적 문제 (combinatorial)라고도 한다.

## Approach
초기에는 deductive method와 transformational method이 중심이 되었다. 이러한 방법들은 지수적으로 증가하는 theorem-proving deductive inference tree와 correctness-preserving code rewrite rules를 기반으로 하고있다. 그러나 두 방법들은 제대로 동작하는 코드를 생성할 수는 있지만 non-deterministic nature of a theorem proving과 code rewriting loop는 효율성을 보장해주지 않을 뿐더러 종료도 제대로 되지 않을 수 있다. 이런 이유로 최근에는 clever domain specific heuristic for cutting down the derivation tree를 사용한다.

Deductive synthesis approach는 사용자의 의도에 맞게 formal specification이 제공된다고 가정하는데 이것은 프로그램을 직접 작성하는 것 만큼이나 복잡한 일이다. 이러한 이유 때문에 input-output 예제 등을 기반으로 한 inductive synthesis approach가 등장하게 된다. 최근 들어서는 사용자가 specification과 더불어 program space의 skeleton(grammer)를 제공할 수 있게 해준다. 이로 인해 검색 절차를 효율적으로 할 수 있도록 hypothesis space에 구조를 제공해주고, grammer로부터 파생되기 때문에 학습된 프로그램은 좀 더 이해하기 쉬워진다.   

## Search space
Search space는 표현력과 효율성 간의 균형을 잘 유지하여야 한다. Space는 domain과 다양한 프로그램들을 포함할 수 있을 만큼 크고, 표현력이 좋아야 하며, program의 space는 효율적인 탐색을 위해서 다소 제한적이어야 한다.

## Search Techniques
아래 탐색 기법의 목적은 결국 프로그램 공간의 크기를 줄이고, 탐색하려는 후보 프로그램의 수를 감소 시킴으로써 합성 시간을 단축하는 데 있다.
- Enumerative    
 Search space에 있는 프로그램들을 나열하여 constraint를 만족하는지 아닌지를 확인한다. 보통 일반적인 enumerative search는 scale하지 않기 때문에 ordering이나 pruning 등의 최적화 방법을 이용한다.

- Deductive    
 Deductive top-down searching은 divide-and-conquer technique을 이용한다. Enumerative searching과 달리 출력 값으로부터 역방향으로 programmatic path를 탐색한다. F(e1,e2)

- Constraint Solving    
 Constraint solving은 constraint generation과 constraint resolution 두 단계로 나뉜다.   
 Constraint generation은 logical constraint를 생성하는 과정으로 invariant-based, path-based, and input-based constraint generation이 있다.   
 Constraint resolution은 second-order unknown과 universal quantifier를 포함한다. 일반적으로는 먼저 second-order unknown을 first-order unknown으로 reduce한 뒤 universal quantifier를 제거한다. 그런 다음 SMT solver를 이용하여 first-order quantifier-free constraint를 풀어낸다.

- Conflict driven clause learning(CDCL)
CDCL 기법은 탐색 도중 conflict를 만들어내는 절 (clause)을 추가하여 (learning) 다음 탐색에서 동일한 문제로 탐색을 하지 않도록 시간을 단축시켜 주는 가지치기 (pruning) 기법이다.

- Statistical    
  - 머신러닝은 Enumerative search나 deduction에 확장하여 쓰일 수 있다. 이때 likelihood가 제공되어 함수로 input-output example에 쓰일 수 있다.

  - genetic programming    
biological evolution에 영감을 받았다.   
개별 프로그램의 수를 유지하고 이를 biological mutation과 crossover를 통해 다른 프로그램을 만들어낸다. mutation은 random changes를 만들어내는 반면 crossover는 진화된 프로그램들 간의 쓸만한 코드 조각들을 공유한다. user-defined fitness function을 통해 변이의 적합성을 판단한 뒤 선택된 코드들만을 evolution에 계속 참여시킨다. 이 함수를 잘 사용해야 성공함. mutual exclusion algorithms을 발견하고 imperative programs의 버그를 고치는데 사용된다.   

  - MCMC( Markov Chain Monte carlo ) sampling    
주어진 후보들로부터 원하는 프로그램을 찾는데 사용되며 cost metric for Boolean constraint에 의해 좌우된다. 


## Challenges
주요 문제점 : Program space의 intractability, diversity of user intent.   

### program space
 Moore’s law와 constraint solving의 발전으로 program space를 합리적인 시간 안에 큰 규모의 공간을 탐색할 수 있게 되었다. 그러나 현대의 프로그램 합성 기술은 작은 코드 밖에 생성해내지 못하는 단점을 갖고 있어 실제 산업에서 적용되는 경우는 적다.

### User Intent
두번째 과제는 사용자의 의도를 정확히 표현하는 것이다. 사용자 의도를 표현하는 방법에 있어서는 formal specification부터 informal nature-language description, input-output 예제까지 다양하게 존재한다.

프로그램 합성에 대한 실제 application domain은 formal 또는 informal specification으로 표현하기에는 꽤 복잡하다. 그래서 이런 specification이 생성된 프로그램에 비해 너무 구체적이거나, 사용자가 모든 범위를 생각하지 않는 경우가 많은 등의 어려움이 존재한다. 이러한 경우 프로그램 합성을 큰 산업에 적용하는 것을 human-computer interaction(HCI)로도 생각해 볼 수 있다.

 - 추가로 사용자의 의도가 원하는 프로그램의 범위를 제대로 포함하지 못한다면 그에 따른 정확도는 감소할 것이다.

### Scalability
프로그램 탐색에 대한 가지치기를 진행한다 하더라도 정수, 실수와 같은 수를 찾아내야할 경우 이에 대한 경우의 수가 무한하다. 또한 어떠한 한 구문을 생략할 수 있는 경우, 구문이 생략된 프로그램과 그렇지 않은 프로그램들이 존재하기 때문에 이 프로그램들이 가지치기 대상이 되어야하는지 확신하기 힘들다.

## Future work
2020년 GPT-3가 공개됨과 동시에 MS에서는 자연어를 통해 해당 문장에 대응되는 파이썬 소스코드를 생성하는 프로그램의 데모를 공개하였다. 이처럼 소프트웨어 산업에서 프로그램을 자동으로 생성하려는 노력은 계속되고 있으며, Excel의 FlashFill처럼 실제로 우리 곁에서 사용되고 있다. 또한 최근에는 프로그램 합성을 통해 소스코드의 오류를 자동으로 수정하는 자동 디버거 (프로그램 수정)가 개발되었다. 이러한 노력들을 통해 향후 5년 내에 프로그램 자동 생성이라는 분야가 크게 성장할 것이라고 기대한다.

## Reference
[1] Gulwani, Sumit, Oleksandr Polozov, and Rishabh Singh. "Program synthesis." Foundations and Trends® in Programming Languages 4.1-2 (2017): 1-119.\
[2] Armando. (2018). "Introduction to Program Synthesis." Retrieved from https://people.csail.mit.edu/asolar/SynthesisCourse/Lecture3.htm"\
[3] Feng, Yu, et al. "Program synthesis using conflict-driven learning." ACM SIGPLAN Notices 53.4 (2018): 420-435.\
[4] Y, Han. (2020). "Development of an automated debugger with program synthesis and fault localization." Retrieved from https://github.com/CSID-DGU/2020-1-CECD3-WAS-6

<!-- 
3. Dimensions in Program Synthesis
key dimensions
* 사용자의 의도를 표현할 constraints 종류
* 탐색할 program space
* 적용될 search technique

3.1 User Intent
사용자의 의도는 logical specification, examples, traces, natural language, partial program 등의 다양한 형태로 표현될 수 있고, 시나리오, 사용 기술, 사용자의 배경지식 등에 따라 선택하면 된다.   
Logical specification은 프로그램의 입출력 값 간의 논리적 관계를 말한다.   
Example based specification에 내재되어있는 모호함을 해소하기 위해, 사용자와 interactive loop를 이용할 수 있다.   
Trace는 주어진 입력 값에 대하여 한 단계씩 프로그램이 어떻게 동작해야 하는지를 묘사한 것이다. 또한 단순히 입력 값이 어떤 출력 값으로 매핑되는 것만 설명한 것이 아니라, 특정 입력 값이 어떤 출력 값으로 어떻게 바뀌는지, 그 과정을 설명하기 때문에 trace는 input-output 예제보다 더 자세한 묘사 방법이다. 이는 synthesizer의 관점에서 보면 input-output 예제보다 더 많은 정보를 제공하고, 사용자의 관점에서 더 자세한 묘사를 제공한다.   
since it also illustrates how a specific input should be transformed into the corresponding output as opposed to just describing what the output should be.   

3.2 Search Space
Search space는 표현력과 효율성 간의 균형을 잘 유지하여야 한다. Space는 domain과 다양한 프로그램들을 포함할 수 있을 만큼 크고, 표현력이 좋아야 하며, program의 space는 효율적인 탐색을 위해서 다소 제한적이어야 한다.

3.3 Search Techniques
- Enumerative
 Search space에 있는 프로그램들을 나열하여 constraint를 만족하는지 아닌지를 확인한다. 보통 일반적인 enumerative search는 scale하지 않기 때문에 ordering이나 pruning 등의 최적화 방법을 이용한다.

- Deductive
 Deductive top-down searching은 divide-and-conquer technique을 이용한다. Enumerative searching과 달리 출력 값으로부터 역방향으로 programmatic path를 탐색한다. F(e1,e2)

- Constraint Solving
 Constraint solving은 constraint generation과 constraint resolution 두 단계로 나뉜다.   
 Constraint generation은 logical constraint를 생성하는 과정으로 invariant-based, path-based, and input-based constraint generation이 있다.   
 Constraint resolution은 second-order unknown과 universal quantifier를 포함한다. 일반적으로는 먼저 second-order unknown을 first-order unknown으로 reduce한 뒤 universal quantifier를 제거한다. 그런 다음 SMT solver를 이용하여 first-order quantifier-free constraint를 풀어낸다.   

- Statistical
+ 머신러닝은 Enumerative search나 deduction에 확장하여 쓰일 수 있다. 이때 likelihood가 제공되어 함수로 input-output example에 쓰일 수 있다.

+ genetic programming
biological evolution에 영감을 받았다.   
개별 프로그램의 수를 유지하고 이를 biological mutation과 crossover를 통해 다른 프로그램을 만들어낸다. mutation은 random changes를 만들어내는 반면 crossover는 진화된 프로그램들 간의 쓸만한 코드 조각들을 공유한다. user-defined fitness function을 통해 변이의 적합성을 판단한 뒤 선택된 코드들만을 evolution에 계속 참여시킨다. 이 함수를 잘 사용해야 성공함. mutual exclusion algorithms을 발견하고 imperative programs의 버그를 고치는데 사용된다.   

+ MCMC( Markov Chain Monte carlo ) sampling
주어진 후보들로부터 원하는 프로그램을 찾는데 사용되며 cost metric for Boolean constraint에 의해 좌우된다.   

**Future Work.**
- Debuggability
명세를 개선하는데 도움을 줄 수 있다.   
The user would require active assistance to refine the specification.   
sensitive data, large amount data where the result are not easy to verify manually.   
- Multi-modal input
다각화된 입력을 통해서 단순히 문법적으로 올바른 명령어들을 구성하는 것을 뛰어 넘는다. Multi-modal natural input을 받아 명령형 프로그래밍을 할 수도 있고, examples, demonstrations, natural language, keywords, and sketches 등을 통해 intent를 표현할 수 있다.   
 - Adapability
과거에 합성되었던 데이터를 통해 사용자의 under specification에 대한 모호함을 좀 더 효과적으로 해결할 수 있다.   
- Statistical techniques
semantic knowledge나 다양한 연산자 속성에 좋지만, scalability challenge문제가 남아있다. 다양한 종류의 artifact를 예측하는데 있어 딥러닝 방법에 큰 발전이 있었다. 큰 규모의 코드를 합성하거나 추론하는데 사용될 수 있다.   
- Scaling
현재는 복잡한 논리를 갖는 작은 코드를 합성할 수 있다.   
- Knowledge transfer
하나의 도메인으로부터 얻은 것들을 다른 도메인으로 자동으로 일반화하고 전달할 수 있다.   
- Industrialization
대중화하는 것. 상대적으로 신기술이고 프로그램 분석과 달리 널리 알려져있지 않다. 프레임워크, solver-aided language, domain-specific search components의 분리에 대한 합성의 발전이 이러한 갭을 줄여줄 것이다.   

-->
---
layout: post
title: "Lazy Evaluation"
date: 2020-07-12 05:24:00 -0400
categories: Research Programming-language
---
# Lazy Evaluation: 지연 계산법 (느긋한 계산법)

지연 계산법 (Lazy evaluation)은 표현식 (Expression)의 계산을 바로 하지 않고 (지연, Lazy), 파일 출력, 화면 출력 등 실제로 표현식에 대한 값이 필요할 때 계산하는 방법이다. 이와 반대대는 개념으로 위키피디아에서 조급한 계산법이라 번역된 eager evaluatino이 있으며, 이는 우리가 평소에 아는 것처럼 표현식을 즉시 계산하는 방법이다.

Z3, Yice와 같은 SMT solver나 Haskell, Scala와 같은 함수형 언어에서 성능 향상의 목적으로 많이 사용된다.

## 장점
1. 표현식이 바로 연산되지 않고 실제로 필요한 시점에 계산되기 때문에 더 적은 명령어로 표현식을 계산할 수 있다.
2. 필요할 때에만 연산이 이루어지기 때문에 메모리를 절약할 수 있다.

## 단점
1. 표현식의 계산이 지연됨으로써 쌓이는 표현식들이 존재하게되고, 이를 한번에 처리할 때 오버헤드가 발생할 수 있다.
2. 기존 코드보다 코드 구조가 복잡해져 유지보수에 어려움을 겪을 수 있다.
3. 무조건 성능 향상으로 이어지지 않는다.

## 구현
thunks라는 개념을 이용하여 지연 계산법을 구현할 수 있다.

---
어떻게 보면 단점이 더 뚜렸해 보이지만 엄청난 성능 향상을 보인다. (추후에 이를 뒷받침해주는 예제 코드를 업데이트하겠습니다... **I AM LAZYㅜㅜ**)

## Reference
[1] David Evans. (2008). CS150: Book. Retrieved from https://www.cs.virginia.edu/~evans/cs150/book/
[2] Jonathan. (2017). "Lazy evaluation of function arguments in C++." Retrieved from https://foonathan.net/2017/06/lazy-evaluation/
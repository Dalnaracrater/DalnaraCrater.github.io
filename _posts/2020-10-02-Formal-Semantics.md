---
layout: post
title: Formal Semantics
date: 2020-10-02 05:24:00
tags: Programming-language
featured: true
---
# Formal Semantics

Formal Semantics는 프로그램에 대한 계산적 의미(computational meaning)를 모델링한다. 이때의 모델에 대한 묘사는 이미 명확한 의미를 갖는 기호들로 쓰여진 경우 formal하다고 한다.

우리가 잘 알고있는 문법같은 경우에는 프로그램의 구문(syntax)을 보여주고 있다. 구문의 경우에는 단순히 프로그램의 구조만을 보여준다는 점에서 이 프로그램이 어떠한 의미를 목적으로 설계된 프로그램인지 알기 어렵다. 그러나 의미론(semantics)는 프로그램이 동작하는 과정을 보여주고 있기 때문에 프로그램을 이해하는데 도움을 준다.

- static semantics: 구문이 context-free grammar일 경우 well-formedness constraint를 만족하는지 확인한다. type checking이 static semantics에 해당된다.
- dynamic semantics: 프로그램이 실행될 때, 확인할 수 있는 행동들에 대해서 확인한다. (단, static semantics에 의해 프로그램이 이미 well-formedness를 만족한다고 가정한다.)
- equivalence: 프로그램 실행에 관한 특징들을 추상적으로 모델링하여 두 프로그램이 의미론적으로 동치(semantically equivalent)인지 확인한다. (up to isomorphism)

dynamic semantics에는 여러 방법이 있으며 대표적으로 operational semantics, denotational semantics, axiomatic semantics가 있다.

## Operational Semantics
프로그램 계산이 명확히 모델링된다.

### Bindings
identifier를 특정 값에 bind 해주며, binding map 또는 environment는 현재 identifier에 binding된 값을 나타낸다.

State = Exp X Env
위 식은 (E,p)->(E',p) p가 변하지 않으므로 pㅏE->E'으로 나타낼 수 있다.

### Stores
Assignment는 어떤 장소에 저장된 값에 영향을 주고, Binding은 identifier에 bind된 위치에 영향을 준다.

Assignment와 binding을 잘 구분하면 aliasing 유용하게 사용할 수 있다. (how?)

State = Exp X Env X Store
p ㅏ E,o -> E',o'


### Big-step

### Small-step


## Denotational Semantics
프로그램의 계산적 의미를 형성하는 부분에 대해서만 모델링된다.

## Axiomatic Semantics
변수들의 pre-condition과 post-condition에 대한 관계가 모델링된다.


---
layout: post
title: "Fault Localization"
date: 2020-07-05 05:24:00 -0400
categories: Research Programming-language
---
# Fault Localization: 결함위치식별

Fault Localization (결함위치식별)은 소스코드 내에 존재하는 오류의 위치를 찾아내는 기법으로, spectrum, slice, statistical, machine learning, SMT 등 다양한 방법들이 적용되어 연구되고 있다.

## Coverage-based Fault Localization
coverage 기반의 결함위치식별 기법은 주어진 테스트케이스를 실행하여 결함이 발생하기 까지 실행되는 부분(cover)을 확인하고, 이를 'suspiciouness', '의심도'라는 확률 값으로 나타내어 의심도 값이 높은 코드를 결함 코드라고 추정하는 기법이다. 커버리지 기반의 결함위치식별 기법을 적용한 모델로는 Tarantula가 대표적이다.

### 단점
그러나 이 방법의 경우 동일한 커버리지를 갖는 테스트가 여러 개 존재한다면, 이로 인해 해당 커버리지에 해당하는 코드에 대한 의심도가 증가하여 정확한 의심도 계산이 어려워진다. 따라서 이를 해결하기 위한 방법으로 **같은 커버리지를 갖는 테스트케이스를 하나의 집합으로 묶음**으로써 위와 같은 단점을 해결할 수 있다.

또한 적용되는 의심도 계산 식에 따라 결함위치식별의 정확도가 달라지므로 여러 식을 함께 사용하고 이를 혼합하여 정확도를 향상시킬 수 있다.

## Reference
추후 업데이트하겠음.

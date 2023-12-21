---
layout: post
title: (DreamHack) rev-basic-1
date: 2021-03-03 05:24:00
tags: Security CTF
---

## 풀이

IDA를 통해 주어진 실행 파일을 디스어셈블링하고 string view를 확인해 보았다.\
플래그로 추정되는 문자열은 찾을 수 없어 비교 연산을 하는 코드가 있는지 찾아보았다. 아래 그림과 같이 문자 하나씩 비교해가며 확인하는 프로세스가 존재하였다.

![image](https://user-images.githubusercontent.com/24788751/110456181-9a1b4080-810c-11eb-8cff-dffde77c3eaa.png)

비교되는 문자들을 아스키코드를 참고하여 변환하면 'Compar3_the_ch4ract3r'이라는 문자열을 얻을 수 있다.

![image](https://user-images.githubusercontent.com/24788751/110456857-6391f580-810d-11eb-9e58-76b7f9a709d4.png)
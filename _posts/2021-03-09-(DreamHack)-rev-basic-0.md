---
layout: post
title: (DreamHack) rev-basic-0
date: 2021-03-03 05:24:00
tags: Security CTF
---

## 풀이

IDA를 통해 주어진 실행 파일을 디스어셈블링하였다. 우전 string view를 확인하여 플래그처럼 보이는 문자열을 확인하였다.

![image](https://user-images.githubusercontent.com/24788751/110455212-7dcad400-810b-11eb-9eca-e1f488af9cd1.png)

사용자 입력값이 'Compar3_the_str1ng'과 일치하는지 비교하는 어셈블리 코드를 확인할 수 있었다.

![image](https://user-images.githubusercontent.com/24788751/110455373-b5d21700-810b-11eb-95a9-9114ba67de5e.png)

예상한대로 찾아낸 문자열을 입력하였더니 Correct가 출력되었다.
![image](https://user-images.githubusercontent.com/24788751/110455684-15302700-810c-11eb-91a9-2bdd331ff53d.png)


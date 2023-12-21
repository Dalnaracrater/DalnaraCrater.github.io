---
layout: post
title: "(DreamHack) rev-basic-2"
date: 2021-03-03 05:24:00 -0400
categories: Security CTF
---

## 풀이

IDA를 통해 주어진 실행 파일을 디스어셈블링하고 string view를 확인해 보았다. 플래그로 추정되는 문자열을 얻을 수 없어 어셈블리 코드를 살펴보았다.

![image](https://user-images.githubusercontent.com/24788751/110457204-bbc8f780-810d-11eb-974f-1ab363388b01.png)

프로세스가 반복되고 있음을 알 수 있지만 아직 어떤 동작을 하는지 정확하게 해석할 수 없었다. 다만 aC의 주소를 옮겨 사용한다는 것은 알 수 있었고, aC의 값이 담긴 스택을 확인해 보았다.

![image](https://user-images.githubusercontent.com/24788751/110457996-9be60380-810e-11eb-8c47-5015bdc1928a.png)

'Comp4re_the_arr4y'가 플래그임을 유추할 수 있었다.

![image](https://user-images.githubusercontent.com/24788751/110458317-f97a5000-810e-11eb-8cad-bd93d2f02769.png)

어떨결에 플래그를 찾아내긴 했지만 디스어셈블된 어셈블리 코드를 해석할 수 있었다면 더욱 좋았을 것 같다.
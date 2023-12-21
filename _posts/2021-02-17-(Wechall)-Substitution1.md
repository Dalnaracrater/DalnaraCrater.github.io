---
layout: post
title: (Pwnable) blukat
date: 2021-02-17 05:24:00
tags: Security CTF
---

## 문제 풀이
ls -l 명령어를 통해 어떠한 파일이 있고 어떤 코드로 구성되어 있는지 확인하였다.

![image](https://user-images.githubusercontent.com/24788751/108188675-34621700-7153-11eb-8888-c31cadcaf51f.png)

평소와 같으면 바로 실행 파일을 실행하고, c 파일을 살펴 보았겠지만 자세히 살펴보니 password 파일의 소유 그룹이 눈에 띄었다.

![image](https://user-images.githubusercontent.com/24788751/108188748-49d74100-7153-11eb-9d3b-43561af1b9ee.png)

id 명령어를 통해 내 계정의 uid와 gid를 확인하였다. 확인 결과 내 계정이 속한 그룹이 password 파일에 접근할 수 있음을 알았다.

![image](https://user-images.githubusercontent.com/24788751/108188766-5065b880-7153-11eb-8300-a8e4b5b7ce5f.png)

password 파일 내용 출력 성공!

(주의: 자칫 잘못하면 c 소스코드를 분석하여 시간을 낭비할 수 있음.)
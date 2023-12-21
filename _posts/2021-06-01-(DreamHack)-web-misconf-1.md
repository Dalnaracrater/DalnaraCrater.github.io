---
layout: post
title: "(DreamHack) web-misconf-1"
date: 2021-06-01 05:24:00 -0400
description: web-misconf
# tags: formatting
categories: Security CTF
---

네트워크 장비, CCTV 등 여러 전자제품은 구매를 하면 초기 비밀번호가 설정되어 있다. 그런데 구입 후 이를 변경해주지 않으면 이를 악용할 수 있는 가능성이 존재한다. 해당 문제도 비슷한 시각에서 출제된 문제라고 생각한다.

## 풀이

접속하면 다음과 같은 화면이 출력된다.

![image](https://user-images.githubusercontent.com/24788751/120296923-91996700-c303-11eb-8b0b-61b4b4fb4fd8.png)

회원가입 기능이 따로 없어 admin/admin으로 로그인 시도를 해보았는데 성공하였다.

문제에서 flag가 Organization에 있다고 했으므로 설정을 확인하여 플래그를 찾을 수 있었다.
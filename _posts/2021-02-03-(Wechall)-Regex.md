---
layout: post
title: (Wechall) Regex 1
date: 2021-02-03 05:24:00 -0400
tags: Security CTF
---

# (Wechall) Training Regex write up

Regex 정규표현식 문법을 이용하여 empty string을 매칭하는 정규 표현식 찾기


답
```java
/^$/
```
출처: http://www.wechall.net/challenge/training/regex/index.php

# (Wechall) Training Caesar 2 write up

 시저 암호는 고전 암호 알고리즘 중 하나로 Key만큼 문자열을 치환시켜 암호문을 얻는다. (복호화 과정은 그 반대)
 C = (P + K) % 26

 이러한 치환 암호의 문제점은 알파벳의 출현 빈도 수가 그대로 노출되어 암호문에 남아있다는 것이다. 이떄 통계적 성질을 이용하여 암호를 풀어낼 수 있다.
 보통 알파벳의 경우 e, t, a, s, i, n, ... 순으로 출현 빈도가 나타난다.

보통 치환 암호는 암호문과 암호키 간의 관계를 숨기는 혼돈(confusion)의 성질을 만족시킨다.

 이번 문제에서는 아스키 코드를 사용할 수 있도록 하여 키의 크기가 26에서 128로 늘어났다는 점에 주의한다.

```python
ciphertext = "63 0B 0B 00 20 06 0B 7E 48 20 15 0B 11 20 0F 0B\
 08 12 01 00 20 0B 0A 01 20 09 0B 0E 01 20 7F 04\
 7D 08 08 01 0A 03 01 20 05 0A 20 15 0B 11 0E 20\
 06 0B 11 0E 0A 01 15 4A 20 70 04 05 0F 20 0B 0A\
 01 20 13 7D 0F 20 02 7D 05 0E 08 15 20 01 7D 0F\
 15 20 10 0B 20 7F 0E 7D 7F 07 4A 20 73 7D 0F 0A\
 43 10 20 05 10 5B 20 4D 4E 54 20 07 01 15 0F 20\
 05 0F 20 7D 20 0D 11 05 10 01 20 0F 09 7D 08 08\
 20 07 01 15 0F 0C 7D 7F 01 48 20 0F 0B 20 05 10\
 20 0F 04 0B 11 08 00 0A 43 10 20 04 7D 12 01 20\
 10 7D 07 01 0A 20 15 0B 11 20 10 0B 0B 20 08 0B\
 0A 03 20 10 0B 20 00 01 7F 0E 15 0C 10 20 10 04\
 05 0F 20 09 01 0F 0F 7D 03 01 4A 20 73 01 08 08\
 20 00 0B 0A 01 48 20 15 0B 11 0E 20 0F 0B 08 11\
 10 05 0B 0A 20 05 0F 20 00 02 0C 05 02 05 09 0A\
 03 05 0F 04 4A"
cipher = ciphertext.split(" ")

for j in range(0, 127):
	for i in cipher:
		print(chr((int("0x" + i, 0)+j)%128), end="")
	print("")
```

위 코드를 실행시키면 결과물 중 아래와 같은 문장을 얻을 수 있다.

Goodjob,yousolvedonemorechallengeinyourjourney.Thisonewasfairlyeasytocrack.Wasn'tit?128keysisaquitesmallkeyspace,soitshouldn'thavetakenyoutoolongtodecryptthismessage.Welldone,yoursolutionisdfpifimngish.

출처: http://www.wechall.net/challenge/training/crypto/caesar2/index.php

# (Wechall) Training Transposition 2 write up
전치 암호는 key에 따라 문자의 순서를 바꾸는 암호화 방식이다.

전치 암호는 암호문과 평문의 관계를 숨기는 확산(Difussion) 성질을 만족시킨다.

'oWdnreuf'에서 암호화 키가 [2, 1]임을 유추할 수 있다.

```python
cipher = "oWdnreuf.lY uoc nar ae dht eemssga eaw yebttrew eh nht eelttre srax enic roertco drre.Ihtni koy uowlu dilekt oes eoyrup sawsro don: wc lfoediorbp.e"
keysize = 2
key = [2, 1]

for i in range(0, int(len(cipher)/keysize)):
	snippet = cipher[i*keysize:i*keysize+keysize]
	for k in key:
		print(snippet[k-1], end="")
```

Wonderful. You can read the message way better when the letters are in correct order. I think you would like to see your password now: cfleoidropbe.

출처: http://www.wechall.net/challenge/training/crypto/transposition1/index.php

# (Wechall) Training PHP LFI (Local File Inclusion) write up

LFI 취약점은 공격 대상 서버에 있는 파일을 포함시켜 공격한다. 보통 include 부분에 입력값 검증 등 적절한 보안 조치가 없을 경우에 발생한다.
cf. RFI (Remote File Inclusion)

filename에 대한 검증 없이 include를 해 취약점 발생
```php
$filename = 'pages/'.(isset($_GET["file"])?$_GET["file"]:"welcome").'.html';
include $filename;
```


?file=../../solution.php 를 통해 해당 파일을 불러오고 %00을 통해 끝에 .html이 붙지 않도록 한다. (Null byte injection)
```php
https://www.wechall.net/challenge/training/php/lfi/up/index.php?file=../../solution.php%00
```

출처: http://www.wechall.net/challenge/training/php/lfi/up/index.php
---
layout: post
title: (Wechall) A Black Hats Tale
date: 2021-02-10 05:24:00
tags: Security CTF
---

## 문제 풀이
War driving을 통해 은행 본사의 무선 통신을 스니핑하여 통신 데이터 일부를 얻었다 (wpa_psk.cap). 은행은 WPA를 통해 통신을 암호화 하고 있어 스니핑된 데이터는 암호화되어 있다. 따라서 WPA의 키를 알아야 평문을 얻을 수 있다.

문제에서는 dictionary attack을 사용하여 WPA의 패스워드를 알아내라고 힌트를 주었다.

해당 문제를 풀기 위해 aircrack-ng 프로그램을 사용하였으며, dictionary list는 따로 다운로드 받았다. (kali linux에는 /pentest/wireless/aircrack-ng/test/password.lst 에 저장되어 있다고 한다.)

aircrack-ng를 사용하면 다음과 같이 WPA에 사용되는 키가 "jennifer"임을 알 수 있다.

<img width="625" alt="wpa_aircrack" src="https://user-images.githubusercontent.com/24788751/107494877-acb85d80-6bd2-11eb-9976-f5061bc705c3.png">

------
## WPA란?
WPA는 Wifi Protected Access의 약자로 WEP를 보안하기 위해 고안되었다.

### WEP
Wired Equivalent Privacy는 IV(Initial Vector)와 사전에 공유된 키를 통해 난수를 생성하여 이를 키 스트림으로 사용한다. 또한 암호문에 대해 CRC-32 알고리즘을 사용하여 ICV(Integrity Check Value)를 생성함으로써 무결성을 확인한다.

#### 단점
- IV의 길이가 24비트로 짧기 때문에 반복되어 사용될 경우 IV 값을 찾아내기 쉬움
- 고정키를 사용하기 때문에 패킷 수집을 통해 암호분석을 통해 암호키를 찾아내기 쉬움
- 암호화 알고리즘으로 RC4를 사용하고 있어 자체적으로 취약함

### WPA
WPA는 WEP을 대체하기 위해 TKIP(Temporal Key Integrity Protocol)을 사용한다. TKIP는 RC4를 사용하긴 하지만 **48비트의 확장된 IV 사용, IV의 순차적 증가를 통한 재생 공격 보호**, CRC-32보다 안전한 **MIC(Message Integrity Check) 알고리즘** 사용을 통해 위 목적을 실현하려고 하였다.

#### 단점
- RC4의 취약점이 그대로 남겨짐

### WPA2
WPA의 TKIP를 보안하기 위해 AES를 사용하는 CCMP(Counter mode with CBC-MAC Protocol) 암호화 방식을 적용하였다. 128비트의 대칭키와 48비트의 IV를 사용하고 무결성 보장과 replay attack를 방지할 수 있다.

### WPA-PSK(Wi-Fi Protected Access Pre-Shared Key)
인증 서버가 설치되지 않은 소규모 망에서 사용되는 방식이다. 동일한 공유키를 갖고 있는지 확인하기 위해 802.1x에 규정된 EAPoL-Key 프레임을 활용하여 4-way handshake를 통해 인증을 수행한다. 암호화키가 1만개의 패킷마다 바뀌는 것이 특징이다.
- 암호키가 짧거나 추측하기 쉬운 값일 경우 사전 공격 (dictionary attack)에 취약함 (사전의 성능에 좌우됨)
<img width="1226" alt="wpa_wireshark" src="https://user-images.githubusercontent.com/24788751/107497481-fce4ef00-6bd5-11eb-9ba2-d2556f117e3c.png">
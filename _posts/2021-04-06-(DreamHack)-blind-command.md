---
layout: post
title: (DreamHack) blind-command
date: 2021-04-06 05:24:00
tags: Security CTF
---

Blind command injection은 입력 값을 통해 os의 명령어를 실행시키는 취약점이다. 보통은 입력 값을 검증하고 (';', '#'과 같은 값 포함 여부 검증), 사용자 계정 권한 설정을 통해 예방할 수 있다. 특히 root 권한은 부여하지 않도록 주의한다.

## 풀이

주어진 소스코드 에서는 GET 메소드를 통해 요청을 받으면 아무런 동작을 하지 않은 뒤 cmd를 반환한다. 그러나 GET 메소드가 아닌 경우에는 cmd에서 지정한 시스템 함수를 호출할 수 있다.

``` python
@app.route('/' , methods=['GET'])
def index():
    cmd = request.args.get('cmd', '')
    if not cmd:
        return "?cmd=[cmd]"
    print(cmd)

    if request.method == 'GET':
        ''
    else:
        os.system(cmd)
    return cmd

app.run(host='0.0.0.0', port=8000)
```


먼저 로컬 호스트에서 HEAD 메소드를 사용하여 `host:port?cmd=dir`을 요청하게 되면 app.py 파일이 존재하는 디렉토리의 파일들을 출력하는 것을 콘솔창에서 확인할 수 있다.

![ctf2-1](https://user-images.githubusercontent.com/24788751/113688584-6845cb80-9704-11eb-8938-3c095897e2fa.PNG)


그러나 주어진 컨테이너에서 플래그 값을 호출하지 못하였다. (어떻게 하는지 알려주세요ㅜㅜ)

### HTTP 메소드

|name|description|
|----|------------------------------------------------------|
|GET|자료를 요청함|
|HEAD|GET 메소드 요청과 동일한 응답을 요구, body를 포함하지 않음|


### Reference
[1] Heino Sass Hallik. (2020). "How to Prevent Blind Command Injection." Retrieved from https://www.rangeforce.com/blog/how-to-prevent-blind-command-injection
[2] MDN. (2021). "HTTP request methods." Retrived from https://developer.mozilla.org/ko/docs/Web/HTTP/Methods

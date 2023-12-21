---
layout: post
title: (DreamHack) simple-ssti
date: 2021-04-06 05:24:00
tags: Security CTF
---


SSTI (Server Side Template Injection) 취약점은 동적으로 생성되는 웹페이지의 특성으로 인해 발생하는 취약점으로써 템플릿에 RCE (Remote Code Execution)가 존재할 경우 발생되는 취약점이다.

## 풀이

flag.txt에 저장된 값은 FLAG와 app.secret_key에 저장된다. 그러나 FLAG는 전역변수가 아니기 때문에 Error404 메소드에서 사용할 수 없다. 

```python
try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

app.secret_key = FLAG
```

그러나 flask는 Flask 객체의 config 속성에 애플리케이션의 실행 환경에 대한 설정 값을 저장한다. 그렇기 때문에 FLAG 값 또한 config에 설정이 되어 있다.

```python
@app.errorhandler(404)
def Error404(e):
    template = '''
    <div class="center">
        <h1>Page Not Found.</h1>
        <h3>%s</h3>
    </div>
''' % (request.path)
    return render_template_string(template), 404

app.run(host='0.0.0.0', port=8000)
```

`%s`를 통해 ssti 취약점이 발생될 수 있다. 따라서 config를 출력하도록 `host:port/{{config}}`를 요청하면 다음과 같이 플래그 값을 얻을 수 있다.

![image](https://user-images.githubusercontent.com/24788751/113693098-35520680-9709-11eb-94b1-eb0910fc6c2f.png)

### Reference
[1] Armin Ronacher. (2012)."설정 다루기." Retrieved from https://flask-docs-kr.readthedocs.io/ko/latest/config.html
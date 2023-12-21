---
layout: post
title: "(DreamHack) command-injection-1"
date: 2021-03-30 05:24:00 -0400
categories: Security CTF
---

command injection은 말 그대로 명령어를 주입하여 해당 명령어를 실행시키는 취약점이다.

ping 페이지에는 다음과 같이 host 주소를 입력하는 폼이 있으며, 소스코드에는 host에 3번 icmp 패킷을 전송하도록 되어있다.

``` python
@APP.route('/ping', methods=['GET', 'POST'])
def ping():
    if request.method == 'POST':
        host = request.form.get('host')
        cmd = f'ping -c 3 "{host}"'
        try:
            output = subprocess.check_output(['/bin/sh', '-c', cmd], timeout=5)
            return render_template('ping_result.html', data=output.decode('utf-8'))
        except subprocess.TimeoutExpired:
            return render_template('ping_result.html', data='Timeout !')
        except subprocess.CalledProcessError:
            return render_template('ping_result.html', data=f'an error occurred while executing the command. -> {cmd}')

    return render_template('ping.html')
```

따라서 입력 폼에 명령어를 삽입하면 될 것 같다는 생각이 들었다. 하지만 입력 폼은 알파벳 대소문자, 숫자를 5 ~ 20번까지만 사용할 수 있도록 되어있다.

```html
<input type="text" class="form-control" id="Host" placeholder="8.8.8.8" name="host" pattern="[A-Za-z0-9.]{5,20}" required>
```

`127.0.0.1; cat flag.py`을 입력하여 보았다.\
다음과 같이 에러 메시지가 출력된다. 자세히 살펴보니 명령어에 큰 따옴표가 삽입되어 있다. 

![image](https://user-images.githubusercontent.com/24788751/112983371-93766b00-9198-11eb-8617-18bfa7932aed.png)

이를 처리하기 위해 중간에 큰 따옴표를 삽입하였다.\
`127.0.0.1"; cat "flag.py`를 입력하였더니 정상적으로? 명령어가 삽입되었으며, 플래그 값을 얻을 수 있었다.


![image](https://user-images.githubusercontent.com/24788751/112982615-a3417f80-9197-11eb-9e4e-146df2ac6bb1.png)

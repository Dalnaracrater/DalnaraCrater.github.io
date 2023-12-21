---
layout: post
title: "(DreamHack) csrf-1"
date: 2021-03-30 05:24:00 -0400
categories: Security CTF
---

CSRF(Cross Script Request Forgery)는 클라이언트의 권한을 이용해서 의도하지 않은 요청을 하는 취약점이다.


![image](https://user-images.githubusercontent.com/24788751/112967909-b7c94c00-9186-11eb-9328-f4d1ad066342.png)

flag라는 페이지에 위와 같이 입력을 할 수 있는 폼이 있었다. FLAG를 얻기 위해서는 '/admin/notice_flag'에서 host가 127.0.0.1, userid가 admin이어야 함을 알 수 있다.\
그렇게 되면 memo_text가 전역변수로 사용되어 있기 때문에 해당 변수에 FLAG 값을 저장한 뒤 memo 페이지를 호출하면 memo_text 값이 메모 페이지에 출력된다.


```python
@app.route('/flag', methods=['GET', 'POST'])
def flag():
    if request.method == 'GET':
        return render_template('flag.html')
    elif request.method == 'POST':
        csrf = request.form.get('csrf', '')
        if not read_url(csrf):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'

def read_url(url, cookie={'name': 'name', 'value': 'value'}):
    cookie.update({'domain':'127.0.0.1'})
    try:
        options = webdriver.ChromeOptions()
        for _ in ['headless', 'window-size=1920x1080', 'disable-gpu', 'no-sandbox', 'disable-dev-shm-usage']:
            options.add_argument(_)
        driver = webdriver.Chrome('/chromedriver', options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get('http://127.0.0.1:8000/')
        driver.add_cookie(cookie)
        driver.get(f'http://127.0.0.1:8000/csrf?csrf={urllib.parse.quote(url)}')
    except:
        driver.quit()
        return False
    driver.quit()
    return True
```

```python
@app.route('/admin/notice_flag')
def admin_notice_flag():
    global memo_text
    if request.remote_addr != '127.0.0.1':
        return 'Access Denied'
    if request.args.get('userid', '') != 'admin':
        return 'Your not admin'
    memo_text += f'[Notice] flag is {FLAG}\n'
    return 'Ok'

memo_text = ''
@app.route('/memo')
def memo():
    global memo_text
    text = request.args.get('memo', None)
    if text:
        memo_text += text.replace('<', '&lt;') + '\n'
    return render_template('memo.html', memo=memo_text)
```

주어진 텍스트 박스에 아래 스크립트를 입력하였다.
```javascript
<link rel="stylesheet" href="/admin/notice_flag?userid=admin">
```

memo 페이지를 조회해보니 다음과 같으 플래그 값을 얻을 수 있었다.
![image](https://user-images.githubusercontent.com/24788751/112967836-a5e7a900-9186-11eb-9630-f79e843cfe8d.png)
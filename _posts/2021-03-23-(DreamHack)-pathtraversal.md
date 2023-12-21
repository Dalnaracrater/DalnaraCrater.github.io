---
layout: post
title: (DreamHack) pathtraversal
date: 2021-03-23 05:24:00
tags: Security CTF
---

Path traversal은 URL의 경로를 임의로 조작하여 의도하지 않는 api 실행 및 파일 접근을 가능하게 하는 취약점입니다. (cf. directory listing)

주어진 소스코드를 살펴보면 사용자 입력으로 userid를 받아 {API_HOST}/api/user/{userid}의 경로로 POST 요청을 보내는 것을 알 수 있습니다.

``` python
@app.route('/get_info', methods=['GET', 'POST'])
def get_info():
    if request.method == 'GET':
        return render_template('get_info.html')
    elif request.method == 'POST':
        userid = request.form.get('userid', '')
        info = requests.get(f'{API_HOST}/api/user/{userid}').text
        return render_template('get_info.html', info=info)
```

Path traversal 취약점을 통해 flag.txt 값이 저장된 FLAG 변수를 반환하는 /api/flag가 호출될 수 있도록 '../flag'를 기존 url에 추가해주기로 합니다.

``` python

try:
    FLAG = open('./flag.txt', 'r').read() # Flag is here!!
except:
    FLAG = '[**FLAG**]'

@app.route('/api/flag')
@internal_api
def flag():
    return FLAG
```

사실 주어진 입력란에 ../flag를 입력하여도 일시적으로 undefined라고 표시된 뒤 아무런 변화가 없다. 이는 자바스크립트 코드에서 users 변수가 'guest'는 0, 'admin'은 1로 고정되어 있기 때문이다.

``` javascript
<script>
  const users = {
    'guest': 0,
    'admin': 1
  }
  function user(evt){
  	document.getElementById('userid').value = users[document.getElementById('userid').value];
    return true;
  }
  window.onload = function() {
    document.getElementById('form').addEventListener('submit', user);
  }
</script>
```


따라서 이러한 문제를 해결하기 위해 다음 그림과 같이 Chrome의 console을 이용하여 값을 고정시켜준 뒤 실행해주면 flag를 얻을 수 있습니다.

![image](https://user-images.githubusercontent.com/24788751/112127348-b3da7e80-8c08-11eb-8933-4b989c8626a5.png)

![image](https://user-images.githubusercontent.com/24788751/112127358-b63cd880-8c08-11eb-87b8-3029b68e3235.png)
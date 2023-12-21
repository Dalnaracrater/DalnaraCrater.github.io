---
layout: post
title: "(DreamHack) file-download-1"
date: 2021-03-23 05:24:00 -0400
categories: Security CTF
---

파일 다운로드 취약점은 ../과 같이 url의 경로를 조작하여 파일을 읽거나 다운로드 하는 취약점이다.

주어진 소스코드를 살펴보면, /upload에서 POST 메소드를 통해 filename, content를 사용자 입력으로 받아 filename으로 된 파일을 읽고 content를 쓰는 작업을 한다. 그러나 이때 filename 속에 '..'문자가 있는지 확인하기 때문에 filename을 통해 취약점을 익스플로잇하려고 할 경우 다음과 같은 화면이 출력된다.

![image](https://user-images.githubusercontent.com/24788751/112131875-5c8add00-8c0d-11eb-9387-e557305bd1ea.png)

``` python
@APP.route('/upload', methods=['GET', 'POST'])
def upload_memo():
    if request.method == 'POST':
        filename = request.form.get('filename')
        content = request.form.get('content').encode('utf-8')

        if filename.find('..') != -1:
            return render_template('upload_result.html', data='bad characters,,')

        with open(f'{UPLOAD_DIR}/{filename}', 'wb') as f:
            f.write(content)

        return redirect('/')

    return render_template('upload.html')
```

/read에서는 별다른 사용자 입력 없이 name의 값을 filename으로 읽어들여 해당 파일의 데이터를 읽어들이는 것을 확인할 수 있다.

``` python
UPLOAD_DIR = 'uploads'

@APP.route('/read')
def read_memo():
    error = False
    data = b''

    filename = request.args.get('name', '')

    try:
        with open(f'{UPLOAD_DIR}/{filename}', 'rb') as f:
            data = f.read()
    except (IsADirectoryError, FileNotFoundError):
        error = True


    return render_template('read.html',
                           filename=filename,
                           content=data.decode('utf-8'),
                           error=error)
```

다음과 같이 host:port/read?name=flag.py 파라미터를 추가하면 flag.py 파일이 존재하지 않는다고 나온다.

![image](https://user-images.githubusercontent.com/24788751/112131799-44b35900-8c0d-11eb-9ebd-ef69958ff57a.png)

(만약 /upload에서 리다이렉트 된 페이지에서 filename을 flag.py, content를 Hello라고 입력한 후 업로드 할 경우 host:port/read?name=flag.py에서 Hello가 출력된다.)

다시 위에 코드를 살펴보면 파일의 주소가 {UPLOAD_DIR}/{filename}으로 설정된 것을 볼 수 있으며, UPLOAD_DIR은 uploads이기 때문에 이를 벗어나 주어야 한다.\
따라서 host:port/read?name=../flag.py를 입력하면 flag 값을 얻을 수 있다.

![image](https://user-images.githubusercontent.com/24788751/112131785-3feea500-8c0d-11eb-96b1-751191e4e218.png)
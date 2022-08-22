---
published: true
title: "[Flask] Bcrypt를 이용한 비밀번호 암호화 & JWT 인증 기반 로그인"
author:
  name: 0da
categories:
  - Web
tags:
  - [Flask] 
  - [Bcrypt]
  - [JWT]

date: 2022-08-23
last_modified_at: 2022-08-23
---
<br>

# Bcrypt

비밀번호를 저장하는 목적으로 설계된 단방향 해시 함수이다. 단방향 해시 함수는 암호화는 가능하지만 복호화가 불가능하다. 회원가입 시 사용자가 입력한 비밀번호를 그대로 DB에 저장하는 것은 안전하지 않기 때문에 단방향 암호화를 거친 뒤 DB에 저장해야 한다.

<br>

### 암호화가 왜 필요할까?

- 사용자가 입력한 비밀번호가 DB에 그대로 평문 저장된다면 DB가 해킹을 당할 경우 유저의 비밀번호가 그대로 노출된다.

- 외부 해킹이 아니더라도 내부 인력이 유저들의 비밀번호를 볼 수 있게 된다. 

- 비밀번호를 단방향 암호화하여 DB에 저장하면 해커가 DB를 획득하더라도 복호화할 수 없기 때문에 평문 비밀번호 획득이 불가능하다.

<br>

### Bcrypt 암호화 진행 과정

![6](https://user-images.githubusercontent.com/40850499/185931220-99e8a54f-e26c-44c1-8c47-c1561833015f.png)

<br/>

### Bcrypt 비밀번호 암호화 실습

#### HTML

register.html

```html
<body>
    <form method="POST" id="registerForm">
      <h2>회원가입</h2>
      <br />
      <div class="form-group">
        <label for="user_id">아이디</label>
        <input type="text" class="form-control" id="user_id" name="user_id" />
      </div>
      <div class="form-group">
        <label for="user_pw">비밀번호</label>
        <input
          type="password"
          class="form-control"
          id="user_pw"
          name="user_pw"
        />
      </div>
      <button type="submit" class="btn btn-primary">회원가입</button>
   </form>
</body>
```

<br>

login.html

```html
<body>
    <form method="POST" id="loginForm">
      <h2>로그인</h2>
      <br />
      <div class="form-group">
        <label for="user_id">아이디</label>
        <input type="text" class="form-control" id="user_id" name="user_id" />
      </div>
      <div class="form-group">
        <label for="user_pw">비밀번호</label>
        <input
          type="password"
          class="form-control"
          id="user_pw"
          name="user_pw"
        />
      </div>
      <button type="submit" class="btn btn-primary">로그인</button>
   </form>
</body>
```

<br>

#### .env

민감한 정보가 코드에 노출되는 것을 방지하기 위하여 민감 정보는 .env에 설정하고, app.py에서 os.getenv()를 통해 접근할 수 있도록 하였다.

```
HOST=localhost
USER=mysql ID
PASSWD=mysql ID의 암호
DB=접속할 데이터베이스
```



#### app.py

- python version 3.8.10

- bcrypt version 3.2.2

```python
from flask import Flask, render_template, redirect, request, make_response
import os
import pymysql
import bcrypt

app = Flask(__name__)

HOST = os.getenv('HOST')
USER = os.getenv('USER')
PASSWD = os.getenv('PASSWD')
DB = os.getenv('DB')

db = pymysql.connect(host=HOST, user=USER, passwd=PASSWD, db=DB, charset="utf8")

@app.route('/', methods = ['GET', 'POST'])
def register():
    if request.method == 'GET':
        return render_template('register.html')
    else:
        cur = db.cursor()
        id = request.form.get('user_id')
        pw = request.form.get('user_pw')
        pw = pw.encode('utf-8')
        bcrypt_hash = bcrypt.hashpw(pw, bcrypt.gensalt())
        decode_hash = bcrypt_hash.decode('utf-8')
        sql = "INSERT INTO user (id, pw) values ('%s','%s')" % (id, decode_hash)
        cur.execute(sql)
        db.commit()
        cur.close()

        return redirect('/login')

@app.route('/login', methods = ['GET','POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        id = request.form.get('user_id')
        pw = request.form.get('user_pw')
        cur = db.cursor()
        cur.execute('SELECT * FROM user WHERE id = %s', id)
        user_id = cur.fetchone()

        if user_id:
            cur.execute('SELECT pw FROM user WHERE id = %s', id)
            select_pw = cur.fetchone()
            user_pw_hash = select_pw[0]
            pw = pw.encode('utf-8')
            user_pw_hash = user_pw_hash.encode('utf-8')
            check = bcrypt.checkpw(pw, user_pw_hash)

            if check == True:
                resp = make_response('로그인 성공')
                return resp
            else:
                print("잘못된 비밀번호입니다.")                
        else:
            print("존재하지 않는 계정입니다.")
        return render_template('login.html')


if __name__=="__main__":
    app.run(debug=True)
```

<br>

문자열인 비밀번호를 bcrypt 라이브러리에서 사용하기 위해서는 utf-8 방식의 바이트 인코딩이 필요하다.

```
pw = pw.encode('utf-8')
```

<br>

인코딩한 비밀번호는 아래와 같이 해싱한다. gensalt()를 통해 salt 값을 생성하고, hashpw()에 비밀번호와 salt를 인자로 받아 암호화된 비밀번호를 생성한다.

```python
bcrypt_hash = bcrypt.hashpw(pw, bcrypt.gensalt())
```

<br>

해싱된 비밀번호를 DB에 저장할 때는 현재 바이트인 해싱 비밀번호를 다시 유니코드로 바꿔줘야 한다.  이를 위해 인코딩 때와 동일하게 utf-8 방식으로 디코드 해준다.

```python
decode_hash = bcrypt_hash.decode('utf-8')
```

<br>

회원가입을 마치고 DB 데이터를 확인해보면 다음과 같이 입력한 비밀번호가 bcrypt 암호화를 거쳐 저장된 것을 볼 수 있다.

![1](https://user-images.githubusercontent.com/40850499/185931378-c46d2766-df1c-493c-ab32-f0b0a09a1509.png)

<br>

입력한 비밀번호와 DB에 저장되어 있는 bcrypt 암호화된 비밀번호가 일치하는지 확인한다. 일치 여부는 True / False로 반환된다.

```python
check = bcrypt.checkpw(pw, user_pw_hash)
```

<br>

<br>

# JWT (Json Web Token)

사용자가 로그인을 하면 서버에서 발행해주는 토큰을 가지고 브라우저의 저장소에 토큰을 유지시키는 방식을 토큰 방식이라고 한다. 여기에서 사용되는 토큰이 바로 JWT이다.

<br>

### 구조

![3](https://user-images.githubusercontent.com/40850499/185931437-188f150e-3338-4336-a1aa-d7c1b1ea0bb6.png)

JWT는 위 그림과 같이 .으로 구분하여 3파트로 나누어진다.

- Header
  
  - 토큰의 유형
  
  - 해시 암호화 알고리즘 (HMAC, SHA256, RSA)

- Payload
  
  - 토큰에 담을 클레임(claim) 정보
    
    - Payload에 담는 정보의 한 '조각'을 클레임이라 부르고, name / value의 한 쌍으로 이루어져 있다. 토큰에는 여러개의 클레임을 넣을 수 있다.
    
    - 아이디, 비밀번호 등의 개인정보가 아닌 토큰의 발급일과 만료일자 등의 내용을 담는다.

- Signature
  
  - Header와 Payload를 합친 후 서버에서 지정한 secret key로 암호화시켜 토큰을 변조하기 어렵게 만들어준다.

<br>

### 인증 과정

![2](https://user-images.githubusercontent.com/40850499/185931503-bad1e7ef-8242-447c-b055-63fb677bb1be.png)

<br>

### 세션 방식

세션 방식은 서버의 메모리, 데이터베이스와 같은 서버의 자원들을 사용해서 사용자의 정보를 유지시키는 방식이다.

<br>

#### 세션 방식 인증 과정

1. 사용자 로그인

2. 서버에서는 계정정보를 읽어 사용자를 확인한 후, 사용자의 고유한 ID값을 부여하여 세션 저장소에 저장하고, 이와 연결되는 세션ID를 발행한다.

3. 사용자는 서버에서 해당 세션ID를 받아 쿠키에 저장한 후, 인증이 필요한 요청마다 쿠키를 헤더에 실어 보낸다.

4. 서버는 쿠키 내부의 세션ID를 통해 세션 내부에 일치하는 유저 정보를 가져와 처음 로그인한 유저가 맞는지 확인한다.

<br>

#### 세션 방식의 문제점

세션은 서버의 메모리 내부에 저장되기 때문에 세션의 양이 많아질 수록 메모리에 부하가 걸릴 수 있으며 쿠키와 세션의 정보가 유출되면 보안의 위험이 있다. 이러한 문제점을 해결하고자 고안된 것이 JWT이다.

<br>

### JWT 인증 기반 로그인 실습

#### .env

JWT 토큰 생성에는 payload와 secret key가 필요하다. 본인은 "RAINBOW"로 secret key를 설정하였다.

```
HOST=localhost
USER=mysql ID
PASSWD=mysql ID의 암호
DB=접속할 데이터베이스
SECRET_KEY=RAINBOW
```

<br>

#### app.py

- PyJWT version 2.4.0

```python
from flask import Flask, render_template, redirect, request, make_response
from datetime import timedelta
from datetime import datetime
import pymysql
import bcrypt
import jwt

app = Flask(__name__)

HOST = os.getenv('HOST')
USER = os.getenv('USER')
PASSWD = os.getenv('PASSWD')
DB = os.getenv('DB')
SECRET_KEY = os.getenv('SECRET_KEY')

db = pymysql.connect(host=HOST, user=USER, passwd=PASSWD, db=DB, charset="utf8")

@app.route('/', methods = ['GET', 'POST'])
def register():
    if request.method == 'GET':
        return render_template('register.html')
    else:
        cur = db.cursor()
        id = request.form.get('user_id')
        pw = request.form.get('user_pw')
        pw = pw.encode('utf-8')
        bcrypt_hash = bcrypt.hashpw(pw,bcrypt.gensalt())
        decode_hash = bcrypt_hash.decode('utf-8')
        sql = "INSERT INTO user (id, pw) values ('%s','%s')" % (id, decode_hash)
        cur.execute(sql)
        db.commit()
        cur.close()

        return redirect('/login')

@app.route('/login', methods = ['GET','POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        id = request.form.get('user_id')
        pw = request.form.get('user_pw')
        cur = db.cursor()
        cur.execute('SELECT * FROM user WHERE id = %s', id)
        user_id = cur.fetchone()

        if user_id:
            cur.execute('SELECT pw FROM user WHERE id = %s', id)
            select_pw = cur.fetchone()
            user_pw_hash = select_pw[0]
            pw = pw.encode('utf-8')
            user_pw_hash = user_pw_hash.encode('utf-8')
            check = bcrypt.checkpw(pw, user_pw_hash)

            if check == True:
                payload = {
                    'id': id,
                    'exp': datetime.utcnow() + timedelta(days=1)
                }
                access_token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
                resp = make_response('로그인 성공')
                return resp
            else:
                print("잘못된 비밀번호입니다.")                
        else:
            print("존재하지 않는 계정입니다.")
        return render_template('login.html')


if __name__=="__main__":
    app.run(debug=True)
```

<br>

payload를 설정해준다. exp는 토큰의 유효 시간을 나타낸다. 아래 코드는 토큰 만료 시간을 하루로 지정하였음을 의미한다.

```python
payload = {
    'id': id,
    'exp': datetime.utcnow() + timedelta(days=1)
}
```

<br>

jwt.encode() 함수에 설정한 payload와 secret key를 넣고 사용할 암호화 방식을 선택하여 토큰을 생성한다. JWT에 주로 사용되는 암호화 알고리즘으로는 HS256(대칭키 방식)과 RS256(공개키 방식)이 있으며, 해당 코드에서는 HS256 알고리즘을 사용하였다.

```python
access_token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
```

<br>

# 참고

[비밀번호 암호화 Bcrypt (gensalt, hashpw, checkpw) : 네이버 블로그](https://blog.naver.com/PostView.naver?blogId=wldks79&logNo=222272099807)

https://velog.io/@hxyxneee/bcrypt를-통해-비밀번호-암호화하기

[인증 &amp; 인가 [Bcrypt, JWT] | Youngseo](https://youngseokim-kr.github.io/backend/backend01/)

[JWT (JSON Web Token) 이해하기와 활용 방안 - Opennaru, Inc.](http://www.opennaru.com/opennaru-blog/jwt-json-web-token/)

https://velog.io/@kingth/서버-인증-방식세션쿠키-토큰

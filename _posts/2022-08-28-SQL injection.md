---
title: "SQL Injection "
author: 이정호 
  name: 이정호

categories:
  - Web
tags:
  - [Blog, SCP, SQL, SQL injection, ] 

date: 2022-08-28
last_modified_at: 2022-08-29
---


SQL Injection  
=============
# 1. SQL Injection 공격이란?  

SQL: DB에 명령 내릴 때 쓰는 언어  
Injection: 삽입  
임의의 SQL문을 삽입하고 실행시켜 데이터 베이스가 비정상적인 동작을 하도록
조작하는 행위이다.   

***
<br>

# 2. SQL Injection 공격의 목적

#### 1. DB에 임의의 SQL명령 실행
#### 2. DB의 데이터를 추출
#### 3. 인증 우회 
#### 4. DB 데이터 변조
#### 5. 악성 파일 업로드
***
<br>

# 3. SQL Injection 공격의 종류

### 1. Error based SQL Injection   
다양한 SQL Injection 공격 기법 중에서도 가장 많이 사용되며, 대중적으로 잘 알려진 공격 기법이다.
```
SELECT * FROM users
	WHERE auth = 'admin'
	AND id = ' id '
```
서버에서 요청을 보낸 사용자가 관리자 권한이 있는지 확인하기 위와 같은 sql문으로 확인한다고 했을 때,
권한이 admin이고 올바른 id값을 가진 사용자가 있는지 조회해 결과가 나온다면 관리자로 인정받는다.    
그러나 id의 인자로 아래와 같이 삽입 된다면,
```
' OR '1' = '1
```
싱글쿼터와 OR 1=1 라는 구문이 WHERE절을 모두 참으로 만들어 사실상 무조건적인 admin권한을 획득하는 것 이다.
이 뿐만 아니라
```
SELECT * FROM users
	WHERE auth = 'admin'
	AND id = ''; DROP TABLE users; --'   
````    
<span style="color:red">';</span>
 으로 마무리 하고, <span style="color:red">DROP TABLE users;</span> 새구문으로 사용자 테이블을 삭제하고 <span style="color:red">--</span> 뒷 내용을 주석 처리한다면
사용자 테이블을 전체 삭제하는 구문으로 서버 자체를 운영하지 못하게 할 수도 있는 공격 기법이다.
***
### 2. Union based SQL Injection
원래의 요청에 추가 쿼리를 삽입하여 정보를 얻어내는 기법으로 
2개 이상의 쿼리를 요청하여 결과를 얻는 Union 명령어를 이용여 공격으로 하나의 테이블로 결과를 얻는다.  
단, Union 하는 두 테이블의 컬럼 수가 같아야 하고, 데이터 형이 같아야 한다.

다음과 같은 게시글 검색 쿼리문에서
```
SELECT * FROM Board 
	WHERE title LIKE '%INPUT%' OR contents '%INPUT%'
```
INPUT 안에
```
'UNION SELECT null,id,password FROM Users--    
```
를 넣어주게 되면 두 쿼리문이 합쳐져 하나의 테이블로 보여지게 된다.
즉, Board 테이블의 id, password가 게시글과 함께 화면에 나타난다.
***

### 3. Blind based SQL Injection   
DB로부터 특정 데이터 전달받지 않고 즉, 정보를 직접적으로 알 수 없는 상황이더라도 참과 거짓 정보만을 통해 알아내는 기법이다.
웹 서버 보안으로 인해 기존 SQL Injection 공격 기법을 통한 정보가 출력되지 않는 경우 해당 기법을 시도할 수 있다.
***

### 4. Stored Procedure based SQL Injection   
일련의 쿼리들을 모아 하나의 함수처럼 사용하기 위한 기법으로 공격자는 시스템 권한을 획득 해야 하는 
공격난이도가 높은 공격이지만 공격에 성공한다면, 서버에 직접적인 피해를 가할 수 있다.
공격에 사용되는 대표적인 저장 프로시저는 MS-SQL 에 있는 xp_cmdshell로 윈도우 명령어를 사용할 수 있게 된다.
***

<br>

# 4. SQL Injection 대응방안

* 입력값 검증   
   * 화이트리스트 방식으로 지정된 형식의 문자만 검증한다.
   * 특수문자 등의 에러 발생 유무 확인한다.  
   * db_name()으로 DB명 추출 여부 확인한다.
   
   <br>

* SQL 오류 발생시 에러 메세지 표시하지 않기
  * 별도로 처리해주지 않을 경우 에러가 발생한 쿼리문과 함께 에러에 관한 내용을 반환한다.    
   (이 때 테이블 명, 컬럼명, 쿼리문이 노출 될 수 있기 때문에 메세지가 표시되지 않도록 처리해 주어야 한다.)  
    <br>

* 서버 보안   
   * 최소 권한 유저로 DB를 운영한다.   
   * 목적에 따라 쿼리 권한을 수정한다.   
   * 지속적인 관리를 통해 신뢰할 수 있는 네트워크, 서버에 대해서만 접근 허용한다.
   * 웹 방화벽을 사용한다.   
--------------------------

<br>

# 5. SQL Injection 이해하기
SQL Injection 공격을 이해하기 위한 간단한 실습을 진행해보자  
운영 체제는 kali linux이며, 웹 모의 해킹 실습 서버 dvwa를 통해 실습하였다.        
![스크린샷(677)](https://user-images.githubusercontent.com/102301788/187028966-6c569fdc-9922-4aae-acb0-db6b717840b8.png)   
user ID를 입력하는 창이 존재한다.      
<br>    
![스크린샷(679)](https://user-images.githubusercontent.com/102301788/187028981-a57cb4c3-bc6d-4eef-bf7b-a2c3743ff0d6.png)  
![스크린샷(672)](https://user-images.githubusercontent.com/102301788/187028985-b79ebf8b-09c2-403e-ad88-d36281515c37.png)   
1을 입력하였더니 ID가 1인 사용자의 정보가 출력되었다.   
<br>
소스를 확인해보면   
![스크린샷(687)](https://user-images.githubusercontent.com/102301788/187029208-95fc44f1-cf90-4c2f-adee-fe41a06040d4.png)
![스크린샷(678)](https://user-images.githubusercontent.com/102301788/187029256-5fe03a93-ba42-4d5c-92bb-0e353478b4cd.png)   
이 소스코드의 경우 user ID에 사용자가 입력한 값이
$id라는 변수로 들어가기 때문에 직접 입력한 값으로 인해 sql 쿼리 문이 바뀌기 때문에 이는 sql 쿼리문을 조작할 수 있다는 의미이다.    
<br>
![스크린샷(684)](https://user-images.githubusercontent.com/102301788/187029014-a2be982b-a021-4456-8c81-e3cac8a76cd0.png)  
![스크린샷(685)](https://user-images.githubusercontent.com/102301788/187029016-94a548ec-c90e-4f8d-bd8c-297e1c1f1984.png)   
위와 같이 입력하니 테이블의 모든 정보가 출력되었다. 이는 조건문이 항상 참이 되도록 입력됐기 때문이다. 다음으로 Union을 사용해보자면 위에서 설명한 것과 같이 Union은 두 테이블의 컬럼 수가 같아야 하기 때문에 컬럼 수를 확인해보기 위해 다음과 같이 입력해본다.    
<br>
![스크린샷(690)](https://user-images.githubusercontent.com/102301788/187029832-e51a942c-e774-49d6-b818-d6a0e66b2714.png)   
select 뒤의 상수 값을 주게 되면 그것을 그대로 출력시킬 수 있다고 한다. 
<br>  
![스크린샷(691)](https://user-images.githubusercontent.com/102301788/187029955-a04f9230-c044-474e-a61e-8dbd61eecd8b.png)   
결과를 보니 컬럼의 수가 맞지 않다는 에러가 나왔다.    
<br>
![스크린샷(694)](https://user-images.githubusercontent.com/102301788/187030045-9e3b876b-a64b-47ff-9994-5383a4ecd622.png)    
![스크린샷(695)](https://user-images.githubusercontent.com/102301788/187030051-731fdf6b-6a6c-4250-aa6e-447e6a29be75.png)    
이렇게 select구문의 컬럼이 2개가 사용되었다는 것을 알 수 있다고 한다. 이제 DB정보를 알아보자면   
<br>
![스크린샷(698)](https://user-images.githubusercontent.com/102301788/187030394-29a70fad-9ee1-4552-8967-b5a8dfff31fe.png)     
1'union select schema_name,1 from information_schema.schemata #
DB명 조회와 관련된 구문이다.     
<br>
![스크린샷(699)](https://user-images.githubusercontent.com/102301788/187030406-832fbbc0-3d11-4fd7-9fe9-3a4aa197c9b6.png)   
다음과 같이 모든 DB이름을 확인 할 수 있다.

<br>

# 6. 참고
https://noirstar.tistory.com/264   
https://doh-an.tistory.com/27   
버그잡는해커 유튜브


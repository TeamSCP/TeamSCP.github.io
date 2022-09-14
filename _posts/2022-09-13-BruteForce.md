```
---
title:"Brute Force"
author:
  name: 이제혁

categories:
  - Web
tags:
  - [Brute Force] 
  - [Web Hacking]

date: 2022-09-13
last_modified_at: 2022-09-13
---
```

# Brute Force 이란?

영어로 Brute 는 "짐승 같은,난폭한"이라는 뜻이고, Brute Force는 "난폭한 힘,폭력"이라는 뜻으로 조합 가능한 모든 문자열을 하나씩 대입하여 암호를 해독하는 방법이다.

## Brute Force 장점
이론적으로 가능한 모든 경우의 수를 다 검색해 보는 것이라 정확도 100%가 항상 보장된다. 또한 브루트 포스의 특 장점은 거의 완벽하게 병렬작업이 가능하다는 점 이다. 이 때문에 병렬 프로그래밍 기법을 사용하거나, GPGPU를 이용하기도 하며, 여러 대의 컴퓨터를 연결해서 동시에 작업할 수도 있다. 이렇게 하면 투자 자원에 비례해서 문제 해결하는 시간을 줄일 수 있다.

## Brute Force 단점
브루트 포스에는 치명적인 단점이 있다. 보통 비밀번호로 자주 쓰이는 자릿수가 최대 8자리인 영문 대소문자, 숫자를 충족하는 사전파일을 만들면 그 텍스트 파일의 용량은 34^8바이이며, 이것을 GB로 환산해보면 대략 1663GB가 나온다. 조합 가능한 모든 문자열을 대입하기 때문에 아무리 컴퓨터가 빠르다고 하더라도 시간이 매우 많이 걸린다. 당장 영문 소문자 + 숫자 조합만 쳐도 n자리 암호를 뚫는 데에는 36^n의 시간이 걸린다. 즉 10 글자만 되어도 36^10 = 3,656,158,440,062,976 가지가 된다. 이 정도면 쉬운 암호 알고리즘이라 해도 초당 1억 번 계산 기준 대충 14개월 걸리는 수준이다.물론 암호가 더 복잡하거나 길이가 더 길어지면 수백 ~ 수천 년은 기본으로 기다려야 한다.

## Brute Force 공격예시
아래아 한글, MS워드 등 현대에 많이 쓰는 문서 포맷은 데이터를 모두 암호화했기 때문에 헥스 에디터로 뜯어도 암호를 통과하는 방법 따위는 존재하지 않고, 암호를 하나씩 대입해서 풀어보는 것 이외에는 우회 방법 따위는 존재하지 않는다. 따라서 이런 파일의 암호를 풀어주는 간단한 프로그램들은 그냥 브루트 포스를 구핸하는 프로그램으로 봐도 무방하며, 해커들이 웹페이지에 로그인을 시도할 때에도 자주 사용된다

## Python 으로 Brute Force 구현하기
Webhacking.kr 4번 문제가 Brute Force를 사용하여 해결하는 문제이기 때문에 Webhacking.kr 4번 문제를 이용하여 구현해 보자

<br>

![image](https://user-images.githubusercontent.com/44721784/188077516-0427d183-b096-4c1c-9bfc-c6e5d2e393f1.png)

<br>

4번 문제를 들어가면 이런식으로 문자열이 나오는데 이것을 보고 sha-1로 암호화 되어있다는 것을 알수 있다.

<br>

<img width="1899" alt="image" src="https://user-images.githubusercontent.com/44721784/188078266-f83956f3-77e2-4c1a-a515-284ac458ec26.png">

<br>

3번째 줄을 보면 ,chall4라는 세션값이 존재하고 만약 사용자로부터 POST형식으로 전송받은 KEY값이 세션의 chall4와 일치하면 문제가 풀린다는 것을 알 수 있다.
4번째 줄에 의해, $hash 값은 10000000 ~ 99999999 사이의 숫자 뒤에 'salt_for_you'를 붙인 문자열이 된다.
6번째 줄에서는 $hash를 sha1해시 함수를 사용하여 500번 암호화 시켰다는것을 알수 있다.

<br>

<img width="637" alt="image" src="https://user-images.githubusercontent.com/44721784/188079418-c737c3a2-09a1-42b2-9a00-e1d535dfa1c2.png">

<br>

스레드를 이용하여 시간을 단축 시키고 범위 id와 계산 범위를 지정해준뒤 def work로 넘겨주면 

<br>

<img width="538" alt="image" src="https://user-images.githubusercontent.com/44721784/188080222-fcd18c33-cf49-4bdd-8bf9-2fa7c6b53f97.png">

<br>

1 ~ 20 까지의 파일을 생성하고 그 파일에 계산값에 salt_for_you를 붙인뒤 sha1로 500번 복호화 한 값이 1 ~ 20 파일 안에 저장이 된다. 

<br>

<img width="118" alt="image" src="https://user-images.githubusercontent.com/44721784/188080512-af18bf4e-cdd3-4959-b2fe-1d9a23469224.png">

<br>

이런식으로 파일이 만들어지고 이 안에 결과 값이 저장이 되면 파일을 열어 검색기능으로 처음 나온 문자열과 같은 문자열을 찾아 해결할수 있다.

## Brute Force 대응방안
1. 암호는 최소 10자리 이상을 사용하자. 암호가 12자리를 넘어간다면 슈퍼컴퓨터를 가져와도 안전하다. 숫자만으로 이러어져 있다 해도 암호 한 자리당 경우의 수가 10배씩 늘어난다. 12자리면 10^12이며 영문 대소문자 섞은 12자리라면 경우의 수는 62^12 = 3226266762400000000000가 된다.
2. 특수 문자를 사용하자
3. CAPTCHA를 사용한다. Brute Force를 우리가 비밀번호를 여러번 틀리면 CAPTCHA를 사용하여 사람인지 확인 하는것 처럼 Brute Force를 막기 위해 비밀번호가 연속적으로 틀리게 된다면 CAPTCHA를 사용하여 Brute Force를 막을수 있다.

-나무위키
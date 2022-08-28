---
title: "스택(Stack)과 스택 프레임(Stack Frame)"
author:
  name: 전유경
  
categories:
  - Reversing
tags:
  - [stack, stack  frame]
    
date: 2022-08-28
last_modified_at: 2022-08-28

---

# **스택(Stack)**

## **스택(Stack)의 개념**

**LIFO(Last In First Out)**  또는  **FILO(First In Last Out)**  형태로 동작하는 선형 자료 구조

![stack1](https://user-images.githubusercontent.com/85911868/187039170-6b20f25e-adf2-4b44-a3bc-be1de791f8e0.png)

스택(Stack)의 사전적 의미는  **"더미", "쌓다"**로 자료 구조의 스택도  **데이터를 순차적으로 나열한 구조**이다.

가장 최근에 보관한 자료를 꺼내는 방식으로 동작한다.
<br>

스택의 연산은  **Push**와  **Pop**이 있다.

**Push는 삽입 연산**으로 가장 위에 원소를 추가하고,  **Pop은 삭제 연산**으로 가장 위의 원소를 반환한다.

**Top**은 스택의  **맨 위에 위치**해 있는 원소를 가리킨다.

**Push와 Pop은 Top으로 지정된 곳에서만 가능**하다.

따라서 스택의 접근(Push와 Pop)은 한 곳(Top)에서만 일어난다.

<br>

스택의 예시로는 아래와 같이 접시를 쌓았을 때  **맨 위의 제일 마지막으로 쌓은**  접시부터 쓰게 되는 것을 생각하면 이해하기 쉽다.


![plates](https://user-images.githubusercontent.com/85911868/187039344-d48d1777-ff84-4b5a-a4e4-62b4e02f3ad5.jpg)

<br>

## **스택(Stack)의 특징**

-   제일 위의 데이터만 알 수 있다.
-   스택이 담을 수 있는 크기를 초과하여 자료를 Push 하면  **스택 오버플로우(Stack Overflow)**가 발생한다.
-   스택이 비었을 때 Pop을 하면  **스택 언더플로우(Stack Underflow)**가 발생한다.
  
## **스택(Stack)의 활용 사례**

**※ 주로 왔던 길을 되돌아갈 때 유용**

-   웹 브라우저의 뒤로 가기
-   문서 작업의 Ctrl + z (Undo)
-   역순 문자열 생성
-   후위 표기법 계산
-   함수 호출 시 복귀 저장
-   재귀 함수 호출 시 복귀 저장

## **스택의 연산**

-   **push(item)**  : 스택에 데이터(item) 추가
-   **pop()**  : 스택의 맨 위 원소를 제거하고 반환
-   **peek() / top()**  : 스택의 맨 위의 값을 반환 (제거하지 않음)
-   **clear()**  : 스택 안 모든 값 강제 초기화 혹은 비어있는지 여부 확인
-   **empty() / isEmptyStack()**  : 스택이 비어있는지 여부 확인 (비어있다면  _True_, 비어있지 않다면  _False_)
-   **full() / isFullStack()**  : 스택이 가득 찼는지 여부 확인 (차 있다면 _True_, 차있지 않다면  _False)_
-   **size()**  : 현재 스택에 들어있는 원소의 개수 반환 (크기 확인)

---
<br>

# **Python으로 스택 구현하기**

## **ADT Stack**

> **ADT(Abstract Data Type)** 란?  
> 프로그래밍 언어에서 사용되는 데이터형을 정의함에 있어서 그 데이터형에 적용 가능한 연산 형식과 제약 조건 등만을 보여주고 실제로 그 연산이 어떻게 구체적으로 표현되어 있는지는 알 수 없게 하는 기능.  
> 출처 :  **추상 데이터형**  [abstract data type, 抽象-型] (컴퓨터인터넷IT용어대사전, 전산용어사전편찬위원회)  
>   
> → 간단히 말해,  **구현 방법은 알리지 않고 실제 기능이나 형식(e.g. 함수)을 정의해 놓은 것.**
> 

-   **push(item)** : 스택에 데이터(item) 추가
-   **pop()** : 스택 맨 위 원소를 제거하고 반환
-   **peek()** : 스택의 맨 위의 값을 반환 (제거하지 않음)
-   **clear()** : 스택 안 모든 값 강제 초기화
-   **isEmptyStack()** : 스택이 비어있는지 여부 확인  (비어있다면 _True_, 비어있지 않다면 _False_)
-   **isFullStack()**  : 스택이 가득 찼는지 여부 확인  (차 있다면 _True_, 차있지 않다면 _False)_
-   **size()**  : 현재 스택에 들어있는 원소의 개수 반환 (크기 확인)
-   **display()** : top 위치에 있는 원소부터 차례대로 출력
  
<br>

## **스택 구현하기 - 부분 설명**

### **1. 클래스 생성과 __init__**

```python
class STACK:
    def __init__(self, max_size=5):
        self.stack = []
        self.top = -1
        self.max = max_size-1
```
STACK이라는 클래스를 생성해주었다.

**__init__ 함수**

1.  원소를 넣어줄 리스트 생성  
    이 리스트가 원소가 들어갈 스택 역할을 하는 것이다.
2.  top 변수를 선언해준다.  
    top가 -1인 이유는 스택이 비어있는 상태로 설정해주기 위함이고, 리스트는 0번지부터 시작하기 때문이다.
3.  스택의 최대 크기를 정해준다.  
    위에서 max_size를 정해주었는데, 이는 5개의 원소를 넣을 수 있다는 의미로 정해주었다.  
    max_size-1을 해준 이유는 첫 번째 원소의 인덱스 값이 0이기 때문에 총 5칸을 만들어주기 위함이다.
    
<br>

### **2. isEmptyStack()과 isFullStack()**
```python
    def isEmptyStack(self):
        if self.top == -1:
            return True
        else:
            return False

    def isFullStack(self):
        if self.top == self.max:
            return True
        else:
            return False
```
push와 pop을 해주기 위해서는 먼저 스택이 완전히 비워져 있는지, 혹은 꽉 차있는 지를 알아야 한다.

따라서 스택의 상태를 검사해줄 함수 isEmptyStack()과 isFullStack()을 먼저 선언해줄 것이다.

  

**isEmptyStack 함수**

1.  만약 top이 -1(스택이 완전히 비워져 있음)이라면 True를 반환하고, 아니라면 False를 반환해준다.

**isFullStack 함수**

1.  만약 top이 스택의 최대 크기와 일치(스택이 완전히 차있음)한다면 True를 반환하고, 아니라면 False를 반환해준다.

<br>

### **3. peek()**
```python
    def peek(self):
        if self.isEmptyStack() == True:
            print("Stack is empty.")
        else:
            return self.stack[self.top]
```
먼저, 스택이 비어있는지 검사해주어 비어있다면 "Stack is empty."라는 메시지를 띄워준다.

비어있지 않다면 스택 맨 위의 값인 top을 반환해준다.

  <br>

### **4. clear()**
```python
    def clear(self):
        self.stack = []
        self.top = -1
```
stack 변수를 다시 빈 리스트로 선언해주고, top의 값도 -1로 저장해준다.

  <br>

### **5. size()**
```python
    def size(self):
        return len(self.stack)
```
len()을 이용해 스택의 크기를 반환해준다.

  <br>

### **6. display()**
```python
    def display(self):
        for a in reversed(self.stack):
            print(a)
```
스택 안 값들을 top을 기준으로 출력해준다. (top을 제일 먼저 출력)

  <br>
  
### **7. push()**
```python
    def push(self, item):
        if self.isFullStack() == True:
            print("Stack is full.")
        else:
            self.stack.append(item)
            self.top += 1
```
스택이 가득 차 있는지 검사를 해주고 가득 차 있다면 "Stack is full"을 출력해준다.

가득 차 있지 않다면 stack 안에 item 인자 값을 추가해주고, top의 위치를 1 증가시켜준다.

  <br>
  
### **8. pop()**
```python
    def pop(self):
        if self.isEmptyStack() == True:
            print("Stack is empty.")
        else:
            pop_item = self.stack[self.top]
            del self.stack[self.top]
            self.top -= 1
            return pop_item
```
스택이 비어있는지 검사를 해주고 비어있다면 "Stack is empty."를 출력해준다.

완전히 비어있지 않다면 pop_item 변수에 스택의 가장 위 원소를 저장해주고, 그 값을 스택에서 삭제해준다.

top의 위치 값도 1 감소시켜준다.

마지막으로 pop_item(스택 맨 위의 값)을 반환해준다.
  
<br>

## **전체 코드**
```python
class STACK:
    def __init__(self, max_size=5):
        self.stack = []
        self.top = -1
        self.max = max_size-1

    def isEmptyStack(self):
        if self.top == -1:
            return True
        else:
            return False

    def isFullStack(self):
        if self.top == self.max:
            return True
        else:
            return False

    def peek(self):
        if self.isEmptyStack() == True:
            print("Stack is empty.")
        else:
            return self.stack[self.top]

    def clear(self):
        self.stack = []
        self.top = -1

    def size(self):
        return len(self.stack)

    def display(self):
        for a in reversed(self.stack):
            print(a)

    def push(self, item):
        if self.isFullStack() == True:
            print("Stack is full.")
        else:
            self.stack.append(item)
            self.top += 1

    def pop(self):
        if self.isEmptyStack() == True:
            print("Stack is empty.")
        else:
            pop_item = self.stack[self.top]
            del self.stack[self.top]
            self.top -= 1
            return pop_item
   ```
---
<br>

# **스택 프레임(Stack Frame)**

## **스택 프레임(Stack Frame)이란?**

ESP(스택 포인터)가 아닌  **EBP(베이스 포인터) 레지스터를 사용하여**
**스택 내의 로컬 변수, 파라미터, 복귀 주소에 접근**하는 기법

➡️ 함수의 매개 변수, 함수 반환 주소값, 지역 변수 등
**메모리의 스택 영역에 순서대로 저장되는 함수의 호출 정보**를 스택 프레임이라고 한다.

  <br>

## **짚고 넘어가기 - 스택**

  

### **(1) Push**

<img width="533" alt="push" src="https://user-images.githubusercontent.com/85911868/187040366-15be39a7-4789-4970-81da-2c59155f9225.png">

스택에 Push를 하여 값을 넣으면 스택의 바닥(EBP)에서 스택 탑(ESP)을 향해 데이터가 쌓인다.

  

### **(2) Pop**

<img width="534" alt="pop" src="https://user-images.githubusercontent.com/85911868/187040375-0e0e2194-04eb-4cec-9b1d-613ac964881c.png">

스택에 Pop을 하여 값을 빼면 스택의 탑(ESP)에서 스택의 바닥(EBP)을 향해 데이터가 줄어든다.

➡️ 즉,  **데이터가 늘어날수록 낮은 주소에 저장**이 된다.

데이터가 늘어날수록 낮은 주소에 저장이 되는 이유는 커널(Kernel)과 만나지 않도록 하기 위함이다.

<br>

## **함수의 호출 단계**

1.  함수가 사용할 매개 변수를 넣고 함수 시작 지점으로 점프한다. ➡️  **함수 호출**
2.  함수 내에서 사용할 스택 프레임을 설정한다. ➡️  **함수 프롤로그**
3.  함수의 내용을 수행한다. ➡️  **함수의 본체**
4.  수행을 끝내면 처음 호출한 지점으로 돌아가기 위해 스택을 복원한다. ➡️  **함수 에필로그**

<br>
  
## **스택 프레임의 구조**
```
PUSH EBP           # 함수시작 (EBP를 사용하기 전에 기존의 값을 스택에 저장) 
MOV EBP, ESP       # 현재의 ESP를 EBP에저장 

                   # 함수 본체 
    ...            # 여기서 ESP가 변경되더라도 EBP가 변경되지 않으므로 
                   # 안전하게 로컬 변수와파라미터를 엑세스할 수 있음 

MOV ESP, EBP      # ESP를정리(함수 시작했을때의 값으로 복원시킴) 
POP EBP            # 리턴되기 전에 저장해 놓았던 원래 EBP 값으로 복원
RETN               # 함수 종료
```
  <br>

### **(1) 함수 프롤로그**
```
PUSH EBP           # 함수시작 (EBP를 사용하기 전에 기존의 값을 스택에 저장) 
MOV EBP, ESP       # 현재의 ESP를 EBP에저장
```
함수 수행을 마치면 다시 제자리로 돌아가기 위해 EBP의 값을 스택 PUSH 하여 저장해놓는다.

EBP에 ESP를 저장해줌으로써 스택 프레임을 생성해준다.

  <br>

### **(2) 함수 에필로그**
```
MOV ESP , EBP      # ESP를정리(함수 시작했을때의 값으로 복원시킴) 
POP EBP            # 리턴되기 전에 저장해 놓았던 원래 EBP 값으로 복원
RETN               # 함수 종료
```
함수 수행을 마치면 호출을 했던 지점으로 돌아가기 위해 처음 함수가 시작되었을 때의 값으로 복원시킨다.

그다음, 프롤로그에서 PUSH 해놓았던 값을 다시 POP 하여 EBP에 넣어준다.

  <br>

## **실습하기**

### **C 코드**

```c
#include <stdio.h>

int sub(int a, int b) {
	int x = a, y = b;
    
	return (x - y);
}

int main() {
	int a = 5, b = 1;
	printf("%d\n", sub(a, b));

	return 0;
}
```
위의 C 코드를 디버거로 열어보았다.

  

### **main() 함수**

![1](https://user-images.githubusercontent.com/85911868/187040912-0dd946a1-0168-4522-8222-2cd4c1ed1278.png)


### **sub() 함수**

![2](https://user-images.githubusercontent.com/85911868/187040933-fbf6eccc-6684-4564-86e4-f7d9ab77d3fa.png)

  <br>

### **(1) main() 함수 시작 & 스택 상태**

⬇️main() 함수를 실행했을 때의 스택 상태

<img src="https://user-images.githubusercontent.com/85911868/187043267-74ad0a0e-a31e-4ad2-ac8a-57f9b7e48740.png" width="255" height="70">

![4](https://user-images.githubusercontent.com/85911868/187043307-9626f281-154c-4ee6-a5ae-202f90ceff13.png)

EBP의 값을 스택에 집어넣는 과정은 EBP가 이전에 가지고 있던 값을 스택에 백업해두기 위함이다.

<br>

### **(2) main() 함수 스택 프레임 생성**

![5](https://user-images.githubusercontent.com/85911868/187043351-f7f88870-7656-4a20-9a99-2e18013dc692.png)

<img src="https://user-images.githubusercontent.com/85911868/187043348-76eccc40-1573-41e4-9397-6400961e15ba.png" width="255" height="70">

![7](https://user-images.githubusercontent.com/85911868/187043350-9fd890f9-2d73-4288-a39d-570be2563555.png)

EBP에 ESP의 값을 옮겨줌으로써  **서로 같은 값(**012FFBB8)을 가지게 된다. 12FFBB8의 주소에는 12FFC00이라는 값이 저장되어 있다. 12FFC00은 **main() 함수의 시작 때 EBP가 가지고 있던 초기 값**이다.

main() 함수가 끝날 때까지 EBP 값은 고정된다.

따라서 스택에 저장된 함수의 매개 변수와 지역 변수들은 EBP를 통해 접근하게 된다.

➡️ 2F10A0과 2F10A1 주소의 두 명령어를 통해 main() 함수에 대한 스택 프레임이 생성되었다.

  <br>
  
### **(3) 지역 변수 설정**

⬇️해당 C 코드
```c
int a = 5, b = 1;
```

![8](https://user-images.githubusercontent.com/85911868/187043442-b7b2d6ab-a7b6-4790-a0a6-dd8f7628559d.png)

위의 C 코드를 보면 int형 변수 2개를 선언해 주었다.

이 두 변수를 스택에 저장하려면 공간을 마련해주어야 하는데, int는 4바이트니 총 8바이트가 필요하다.

이 과정을 ESP에 8을 빼줌으로써 필요한 공간을 확보해 주었다.

![9](https://user-images.githubusercontent.com/85911868/187043445-d5308c53-f4e7-4c37-bc3f-5a99e491e6a2.png)

이제 만들어둔 공간에 각각 5와 1을 넣어줄 차례이다.

[EBP-8]의 주소에 5를 넣어주고, [EBP-4]의 주소에 1을 넣어준다.

따라서 [EBP-8]은 변수 a이고, [EBP-4]는 변수 b임을 알 수 있다.

  

⬇️지금까지 실행한 후의 스택 상태

<img src="https://user-images.githubusercontent.com/85911868/187043447-94d7798f-0018-4a46-843c-7217e0cb25dd.png" width="255" height="70">

<img src="https://user-images.githubusercontent.com/85911868/187043448-477b4288-574a-4f7b-839f-13fb7ed50c08.png" width="400" height="110">

5와 1이 들어 있는 모습을 볼 수 있다.

  <br>
  
### **(4) sub() 함수 매개 변수 입력 & sub() 함수 호출**

⬇️해당 C 코드
```c
printf("%d\n", sub(a, b));
```
![](https://blog.kakaocdn.net/dn/cMlRwK/btrKGSwMKwW/Q4yq1jfbiUtH5Oh7KBtUE0/img.png)

EAX에는 [EBP-4] 즉, 1을 옮겨주고 스택에 넣는다.

ECX에는 [EBP-8] 즉, 5를 옮겨주고 스택에 넣는다.

이는 변수 a, b를 스택에 넣는 것이다.

주목할 내용은 파라미터가 C언어의 입력 순서와는 반대로 스택에 저장된다는 것이다.

이를 **"함수 파라미터의 역순 저장"**이라고 한다

  

⬇️지금까지 실행한 후의 스택 상태

![](https://blog.kakaocdn.net/dn/74PM3/btrKHxeAxRH/3kOtvHuZn3eVBTbrIc96zk/img.png)

![](https://blog.kakaocdn.net/dn/DNg0L/btrKHcPl34P/0wW6XOkESOJfxePyLWPPK0/img.png)

  <br>
그 이후,  sub() 함수를 호출한다.

<br>

### **(5) sub() 함수 시작 & 스택 프레임 생성**

⬇️해당 C 코드
```c
int sub(int a, int b) {
```
![](https://blog.kakaocdn.net/dn/biIDPK/btrKHdgqS6E/nzIc8DolGBYt1IA5UuXsNK/img.png)

main() 함수와 마찬가지로 sub() 함수도 자신만의 스택 프레임을 생성한다.

스택 프레임의 생성 과정은 main() 함수와 같다.

  <br>
⬇️지금까지 실행한 후의 스택 상태

<img src="https://blog.kakaocdn.net/dn/W12Tx/btrKJ34U06z/X19OLRpVopI5p8Sva5di8K/img.png" width="255" height="70">

![](https://blog.kakaocdn.net/dn/t0SNQ/btrKF1VgZhv/6P4sYp938cZEzVBEA3NOd1/img.png)

![](https://blog.kakaocdn.net/dn/lG3oq/btrKIxypJuq/PlZvPB9JOnUX1FUtFCv9y0/img.png)

main() 함수에서 사용되던 EBP 값(12FFBB8)을 스택에 백업한 후 EBP는 12FFBA0로 새롭게 세팅된 것을 확인할 수 있다.

<br>

### **(6) sub() 함수의 지역 변수 설정**

⬇️해당 C 코드
```c
	int a = 5, b = 1;
```
![](https://blog.kakaocdn.net/dn/bUnx3a/btrKHU8txuf/9OC8Zyr0OiM9tYZtIAi3dk/img.png)

sub() 함수의 지역 변수 x, y에 각각 매개 변수 a, b를 대입을 해주는 부분이다.

이 역시 int형 변수 x와 y에 8바이트의 공간을 확보해준다.

  
![](https://blog.kakaocdn.net/dn/bdqFdv/btrKHaxbt7f/dg6DzGhxhjG4IrComoRGu1/img.png)

sub() 함수에서 스택 프레임이 생성되면서 EBP의 값이 변했다.

따라서 [EBP+8], [EBP+C]가 각각 매개 변수 a, b를 가리킨다. (C는 16진수)

[EBP-8], [EBP-4]는 각각 sub() 함수의 지역 변수 y, x를 의미한다.

  <br>

⬇️지금까지 실행한 후의 스택 상태

![](https://blog.kakaocdn.net/dn/xOtd5/btrKHYC2dDR/jNsx5iAkMnOCgRJ1s9CFNk/img.png)

![](https://blog.kakaocdn.net/dn/dbDM3A/btrKHYJOQJU/4SFR3pvsLPP5u0dALVlIuk/img.png)

  
<br>

### **(7) sub() 연산**
```c
	return (x - y);
```
위의 코드의 연산 부분이다.

![](https://blog.kakaocdn.net/dn/bWlyQ5/btrKJ7lW8sn/NzurGIMKv9Smmjtz3Vp2Ck/img.png)

EAX에 [EBP-4] 즉, x를 옮겨준다.

그다음, 5의 값이 들어있는 EAX에 y의 값이 들어있는 [EBP-8]를 빼준다.

이 과정을 쉽게 연산으로 표현한다면

5 - 1

이 된다. 따라서 EAX의 값이 4가 되었다.

  <br>

### **(8) sub() 함수 스택 프레임 해제 & 함수 종료(리턴)**
```c
	return (x - y);
}
```
위의 코드의 해당하는 부분이다.

![](https://blog.kakaocdn.net/dn/dart92/btrKIr54tyW/F17O2Rk31iphzMG1Zhd9jK/img.png)

EBP 값을 ESP에 대입해준다. 이는 sub() 함수가 시작될 때의 ESP값을 EBP에 넣어 두었다가 함수가 종료될 때 ESP를 원래대로 복원시키는 목적으로 사용하는 것이다.

  <br>

![](https://blog.kakaocdn.net/dn/bH2vyI/btrKGpaz4GY/6IzSO5I0n0HZ6uKkKfQLX0/img.png)

sub() 함수를 시작하면서 스택에 미리 저장해둔 EBP 값을 복원해준다.

이 명령어는 앞의 PUSH EBP 명령에 대응하는 것이다.

main() 함수의 EBP값을 복원함으로써 sub() 함수의 스택 프레임이 해제된다.

  <br>

⬇️지금까지 실행한 후의 스택 상태

![](https://blog.kakaocdn.net/dn/UAw72/btrKJ7fdMMu/11Ph2BqTFa3IoPBtuJkrs1/img.png)

![](https://blog.kakaocdn.net/dn/ctPGV0/btrKJ5n9WMk/Mn6MtFnUkOZ4EJT8mYeHX0/img.png)

![](https://blog.kakaocdn.net/dn/cqYttV/btrKHZhF58f/FC5edWD42YsmXCRzrCifd1/img.png)

ESP가 12 FFBA4이고, 이 주소의 값은 F110C1이다.

이 값은 CPU가 스택에 입력한 복귀 주소이다.

  <br>

![](https://blog.kakaocdn.net/dn/bgy02D/btrKGRrbA2l/KSW8mQkkAQtocKtD4iYnNK/img.png)

RETN 명령어가 실행되면 스택에 저장된 복귀 주소로 리턴한다.

  <br>

⬇️지금까지 실행한 후의 스택 상태

![](https://blog.kakaocdn.net/dn/IaaHp/btrKHcV8iJQ/ldof3YTgcIQPWRLhH072y0/img.png)

![](https://blog.kakaocdn.net/dn/WTefj/btrKHb33bIj/kdkNljAc3ZZoSLRwxQE061/img.png)

스택 상태를 확인해보면, sub() 함수를 호출하기 전의 상태로 되돌아왔다.

**(4) sub() 함수 매개 변수 입력 & sub() 함수 호출**의 sub() 함수 호출 전의 스택 상태를 확인해보면

완벽히 일치한다.

  <br>

프로그램은 이런 식으로 스택을 관리하기 때문에 함수의 호출이 아무리 많더라도 스택이 깨지지 않고 잘 유지되는 것이다.

하지만 함수의 매개 변수, 복귀 주소값 등을 한 번에 보관하기 때문에  **스택 버퍼 오버 플로우**(Stack Buffer Overflow)에 취약하다.

  

  

<br>

### **(9) sub() 함수 매개 변수 제거(스택 정리)**

![](https://blog.kakaocdn.net/dn/bbkrwJ/btrKIsqsmvD/SEF8LnGwg7RPNUXQT1PR2k/img.png)

sub() 함수에서 다시 main() 함수로 돌아왔다.

'ADD' 명령으로 ESP에 8을 더하는 이유는 더 이상 사용하지 않는 sub() 함수에게 넘겨준 매개 변수인 a, b를 정리해주기 위함이다.

  <br>

'ADD' 명령 실행 전에 다음 아래의 스택 상태를 잘 보면 12FFBA8과 12FFBAC 주소에 5와 1, 즉 매개 변수 a, b가 있는 것을 알 수 있다.

![](https://blog.kakaocdn.net/dn/WTefj/btrKHb33bIj/kdkNljAc3ZZoSLRwxQE061/img.png)

![](https://blog.kakaocdn.net/dn/cJ4VVU/btrKHbQvECC/wUDmTxotKkkegPXsHIszNk/img.png)

  <br>

⬇️실행 후 스택 상태

<img src="https://blog.kakaocdn.net/dn/dntiXP/btrKIrFdnhj/6mfyH5yk5n1i80NUzFnpqk/img.png" width="255" height="70">

![](https://blog.kakaocdn.net/dn/bMGLsb/btrKF0h2t0T/wFyx3Im3K6uJbmsfLs1IOk/img.png)

<br>

### **(10) printf() 함수 호출**

⬇️해당 C 코드
```c
	printf("%d\n", sub(a, b));
```
![](https://blog.kakaocdn.net/dn/b5mOlz/btrKHvH7IlP/1rQNtOpRo44Lq4ebMOzhAk/img.png)

**PUSH EAX**  : EAX에는 sub() 함수에서 저장된 리턴 값인 4가 들어있다.

**PUSH OFFSET ...**  : "%d\n"

**CALL stackstu.printf**  : printf() 함수 호출

**ADD ESP, 8** : 매개 변수 정리

  <br>

### **(11) 리턴 값 설정**

⬇️해당 C 코드
```c
	return 0;
```
![](https://blog.kakaocdn.net/dn/cdqutF/btrKLfqSWDO/8JSDMi4jZyBZ33xwelSoW1/img.png)

main() 함수의 리턴 값(0)을 설정해주는 코드이다.

XOR은 배타적 논리합으로, 비교하는 값들이 서로 같으면 0을 반환한다.

같은 레지스터를 비교했으므로 0이 된다.

XOR을 사용하는 이유는 'MOV' 명령어보다 실행 속도가 빠르기 때문에 레지스터를 초기화시킬 때 많이 사용된다.

<br>

### **(11) 스택 프레임 해제 & main() 함수 종료**

⬇️해당 C 코드
```c
	return 0;
}
```
![](https://blog.kakaocdn.net/dn/n66Ht/btrKIqTYuwH/DRzauRjKggYI3Okz8K0g71/img.png)

main() 함수의 스택 프레임을 해제한다.

main() 함수의 지역 변수 역시 유효하지 않게 되었다.

  <br>

POP EBP까지 실행하고 나면 아래의 이미지와 같이  **main() 함수가 시작할 때의 스택 모습과 현재의 스택 프레임이 동일해진다.**

<img src="https://blog.kakaocdn.net/dn/drupAv/btrKJ61AdJJ/aYABesdwU1Xa9HOXvctTJ0/img.png" width="255" height="70">

  
<br><br>
  

![](https://blog.kakaocdn.net/dn/ch6QXq/btrKGob0xJ0/1AAY7QGHHVa8V0di9CeDc0/img.png)

이제 메인 함수가 'RETN' 명령어를 실행 하면서 리턴 주소로 점프한다.

그 이후에는 프로세스 종료 코드가 실행되며 최종적으로 프로그램이 종료가 된다.

 <br><br>
 ---
 ### **참고 자료**
 
 이승원. 『리버싱 핵심 원리』. 인사이트, 2012
 
 [정보통신기술용어해설] - 스택 http://www.ktword.co.kr/test/view/view.php?m_temp1=1306



---
title: "[전파해킹] HackRF 소개와 환경 구축 (우분투)"
author:
  name: 노무승

categories:
  - HackRF
tags:
  - [HackRF, 전파해킹] 

date: 2022-08-22
last_modified_at: 2022-08-22
---

## 1. HackRF One이란?

<img src="https://user-images.githubusercontent.com/12112214/185939994-f2e0eba6-b11f-4cf3-952d-da58fb9a24d0.jpg" title="" alt="1290025.jpg" width="400">

 **"HackRF One"**은 Great Scott Gadgets라는 사람이 만든 SDR 장치다. SDR(Software Defined Radio) 장치는 단어 그대로 소프트웨어로 제어할 수 있는 라디오(FM 라디오 보다 폭 넓은 개념) 장치를 말한다. 현재 한국에서는 23만원 정도에 팔고 있는 것으로 확인되는데 알리 익스프레스나 이베이 등 해외직구를 통하면 더욱 싸게 구매할 수 있다.



 해당 장치는 <u>1Mhz ~ 6Ghz 주파수 대역에 전파를 송수신</u>할 수 있으며, <u>반이중 통신(Half-Duplex)</u>을 지원한다. 반이중 통신은 데이터를 송신할 때는 수신받지 못하고, 수신할 때는 송신하지 못하는 전송 방식이다. (무전기를 생각하면 편하다)



 참고로 가용 주파수가 제한적인 RTL-SDR 장치를 제외하면 전파를 송신할 수 있는 SDR 장치 중에서는 가장 저렴하다. HackRF의 상위 SDR 개념인 BladeRF 장치는 전이중 통신(Full-Duplex)을 지원해 송수신을 동시에 수행할 수 있지만 가격 차이가 2배 정도 난다.



---

## 2. HackRF 도구 설치 (우분투)

```
apt update
apt install -y hackrf libhackrf-dev libhackrf0
```

 터미널 창에 위와 같이 입력하여 HackRF 도구를 설치해준다.



<img title="" src="https://user-images.githubusercontent.com/12112214/185940072-fa0bc817-aed1-467f-9db0-0dffcd515218.png" alt="image.png" width="525">

 HackRF 장치를 PC에 연결하고 HackRF USB 디바이스를 VM 안으로 인식시켜 준다. 현재 버추얼박스 가상머신을 쓰고 있어 VMWARE에서는 또 다를 수 있지만 큰 맥락은 같다.



<img src="https://user-images.githubusercontent.com/12112214/185940138-7445d8b9-be1c-44e0-813e-53f45caeeafa.png" title="" alt="image (1).png" width="527">

 'hackrf_info' 명령을 입력하면 정상적으로 HackRF One 장치가 인식된 모습을 볼 수 있다.



---

## 3. GNU Radio 설치 (우분투)

<img title="" src="https://user-images.githubusercontent.com/12112214/185940215-0506ae4a-407e-446d-827b-6c6126f2b140.png" alt="1200px-GNU_Radio_Companion_(3.8.1.0)_Screenshot.png" width="364">

 HackRF 도구로도 충분히 간단한 RF Replay Attack을 수행할 수 있지만, 조금 더 상세한 작업을 하기 위해서는 GNU Radio라는 도구가 필요하다.



```
apt update
apt install -y gnuradio gr-osmosdr
```

위와 같이 입력하여 GNU Radio 도구를 설치해준다. 처음에 GNU Radio만 설치했을 때 osmocom 모듈이 보이지 않아 삽질했던 기억이 있는데, gr-osmosdr 모듈을 apt로 같이 설치해주면 정상적으로 보이게 된다.



<img title="" src="https://user-images.githubusercontent.com/12112214/185940296-ba29a5e9-8282-4944-9441-926997f43883.png" alt="image (2).png" width="607">

 'gnuradio-companion' 명령어를 입력하여 GNU Radio를 실행시킬 수 있다. 참고로 GNU Radio 프로그램은 root 권한으로 실행하는걸 추천한다. root 권한으로 실행하는 방법으로 하나는 'sudo -s'로 root 계정으로 전환한 후, 실행하는 방법이 있고, (위 사진과 같다) 하나는 명령 행 앞에 'sudo'를 붙여 'sudo gnuradio-companion'로 실행하는 방법이 있다.



---

## 4. 참고 자료

[원문] [HackRF One SDR장비 구매 및 소개]

https://blog.naver.com/nms200299/221400205899 (본인 블로그)




[원문] [HackRF 및 GNU Radio 우분투 리눅스 환경 구축]

https://blog.naver.com/nms200299/222801381670 (본인 블로그)

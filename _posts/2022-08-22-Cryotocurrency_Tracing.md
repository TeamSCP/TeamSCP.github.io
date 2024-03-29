---
title: "가상화폐 자산추적"
author:
  name: 유재겸

categories:
  - Blockchain
tags:
  - [blockchain,bitcoin,utxo] 

date: 2022-08-22
last_modified_at: 2022-08-22
---
<br>

# 가상화폐 자산추적
***
비트코인(bitcoin)은 블록체인(blockchain)을 기반으로 만들어진 암호화폐이다.<br> 블록체인은 누구나 거래내역을 조회 및 보관할 수 있는 투명한 구조이지만, 거래 당사자의 인증 대신 문자열로 이루어진 거래 주소를 통해 거래가 진행되기 때문에 익명성을 띈다.<br>
이러한 익명성을 이용하여 암호화폐는 범죄에 이용되곤 한다. 2017년 갑자기 등장하여 전세계를 강타한 워너크라이의 랜섬웨어나, 불법 성착취물을 제작하여 유통한 N번방 사건에서도 대가로 암호화폐를 요구하였다.
<br><br>

## 자산추적의 의미
***

블록체인 네트워크는 익명성을 띄지만, 암호화폐를 이용하여 본인인증 또는 개인정보를 필요로 하는 물건구매 혹은 환전행위를 하게된다. 이 때문에 지속적인 추적을 할 경우, 소유주의 개인정보 확인이 가능해진다.<br>
암호화폐 자산추적에 대한 연구는 국내외로 지속되고있다. 본 포스팅에서는 '박순태, 신용희, 강홍구.(2020).사이버 범죄에 악용되는 암호화폐 불법거래 추적.정보과학회지,38(9),40-47.' 해당 논문에서 제시한 방안을 참고 및 응용하여  <span style="color:red">'**암호화폐  추적 솔루션**'<span style="color:black">을 제작하였다.
<br><br>

## UTXO란?
***
UTXO는 Unspent Transaction output의 약자로, 미사용 트랜잭션 결과물을 의미한다.
비트코인이나 큐텀의 경우에는 누군가에게 송금받은 금액을 UTXO로 저장한다.
비트코인에서는 이더리움의 계좌잔고모델(Account Transaction Model)과 달리 계정이나 잔고가 없고, UTXO를 통해 코인의 존재여부를 확인한다.
<br>
<img src="https://user-images.githubusercontent.com/37801624/185780593-a1bb337f-f4ed-47ac-820e-d71328ddc5ea.png"  width="30%" height="30%" alt="UTXO image"/><br>
(그림1)_UTXO 이미지
<br><br>
위 그림과 같이 사용자의 잔고 총액은, 가지고 있는 UTXO의 총합으로 이루어진다.
이제 가상화폐의 거래과정을 통해 UTXO를 보다 더 정확히 알아보자.
<br><br>

## 가상화폐에서의 거래과정
***
<img src="https://user-images.githubusercontent.com/37801624/185781992-41b9b434-3bc8-4b0c-a605-a9ccb19832e9.png"  width="100%" height="100%" alt="가상화폐 거래과정"/>
(그림2)_가상화폐 거래과정
<br><br>
그림1의 사용자 A가, 사용자 B에게 2BTC를 송금 완료한 경우의 그림이다. 블록체인에서의 거래는 Transaction(이하 TX)이라고 불리는 거래내역에 기록된다. TX입력부에는 이전에 가지고 있었던 5BTC에 대한 UTXO 정보가 기재되어, 다음 거래에서 이전의 거래내역을 역추적할 수 있다. (TX #1의 utxo정보는 그림1의 utxo #1을 가리키고 있다.)
<br><br>
<img src="https://user-images.githubusercontent.com/37801624/185782292-4a22964c-f8f9-43e6-8579-e4d214c0e47f.png"  width="100%" height="100%" alt="역추적 흐름도"/><br>
(그림3)_역추적 흐름도
<br><br>
TX를 나열하면 다음과 같은 그림으로 나타낼 수 있다. 이를 통해 마지막 트랜잭션부터 이전의 거래내역까지 거슬러 올라갈 수 있다. 위 과정을 전처리 후 작업을 거쳐 정방향으로 암호화폐의 흐름을 추적할 수 있는 솔루션을 제작하고자 한다.
<br><br>

## 개발 구조도
***
개발 구조도는 다음과 같다
<br>
<img src="https://user-images.githubusercontent.com/37801624/185782487-4e20c0b7-5fc9-4a4a-9e46-529ac4a345ad.png"  width="100%" height="100%" alt="개발 구조도"/><br>
(그림4)_개발 구조도
<br><br>
1. 블록체인 네트워크(해당 글에서는 비트코인 테스트 네트워크를 사용하였다.)에서 'TX 수집기'가 모든 TX와 UTXO 정보에 관하여 수집한다.
2. 수집한 TX와 UTXO정보를 json형식으로 TX DB에 저장한다.
3. 사용자가 추적하길 원하는 TX정보를 입력한다.
4. 탐색기에 의하여 해당 TX이후의 거래내역을 추적하여 결과물을 출력한다.
<br><br>

## TX수집기 개발
***
<br><img src="https://user-images.githubusercontent.com/37801624/185793113-d3cd5ba9-eacf-470a-bfbc-9fbf04f1b834.png"  width="100%" height="100%" alt="수집기 원리"/><br>
(그림5)_TX수집기 원리
<br><br>
TX수집기와 탐색기는 모두 Go언어로 작성되었다.
먼저 TX수집기는 Bitcoin API를 활용하여, 블록체인 네트워크 내의 모든 TX를 수집한다. 이후 해당 TX의 이전 거래내역인 UTXO 정보 또한 수집하여, 데이터베이스에 저장시켜 놓는다.
<br><br>
블록체인 네트워크 내의 TX수집

```golang
func loadblock(blocknumber string) []string {
	var blockhash []byte

	blockhash = nodecmd("getblockhash", blocknumber)
	//원하는 블록의 해시값을 가져옴

	var blockinfo []byte
	blockinfo = nodecmd("getblock", string(blockhash))
	//해당 블록의 정보를 불러옴

	matchtxs, _ := regexp.Compile("\\[[^]]*\\]")
	txs := matchtxs.FindString(string(blockinfo))
	matchtx, _ := regexp.Compile("[^\"]*")
	txlist := matchtx.FindAllString(txs, -1)
	//정규표현식 파싱을 통하여 블록의 트랜잭션 값만 불러옴
	txlist = txlist[1:]
	//파싱을 통한 값 첫번째에 공백이 들어가 있어서 해당부분 제거
	return txlist
}
```
<br>

UTXO 정보 수집 후, json형식으로 저장
```golang
func savetx(txlist []string, blocknumber string) {

	data := make([]TxData, (len(txlist) / 2))
	for i := 0; i < (len(txlist) / 2); i++ {
		getrawtx := string(nodecmd("getrawtransaction", txlist[2*i]))
		//getrawtransaction 으로 해당 트랜잭션의 정보를 hex값으로 얻어옴
		txinfo := string(nodecmd("decoderawtransaction", getrawtx))
		//얻어온 hex값 해독
		vinmatch, _ := regexp.Compile("\\[[^]]*\\]")
		vin := vinmatch.FindString(txinfo)
		//vin 파싱
		pointermatch, _ := regexp.Compile("[a-z0-9]{63,64}")
		pointer := pointermatch.FindAllString(vin, -1)
		data[i].Info = txlist[2*i]
		data[i].Pointer = pointer
	}
	doc, _ := json.Marshal(data) // data를 JSON 문서로 변환

	err2 := ioutil.WriteFile("../txinfo/"+blocknumber+".json", doc, os.FileMode(0644)) // articles.json 파일에 JSON 문서 저장
	if err2 != nil {
		fmt.Println(err2)
	}
}
```


위의 두 코드는 TX수집기의 코드 일부이다. 블록체인 네트워크 내의 TX와 UTXO정보를 모두 불러와서, 정규 표현식을 통한 파싱을 진행한다. 이후, TX DB에 저장시키는 역할을 한다.
<br><br>

## 탐색기 개발
***

<br>
<img src="https://user-images.githubusercontent.com/37801624/185792635-3a70396e-4eb1-4715-9ea7-aeb186cc62b3.png"  width="100%" height="100%" alt="탐색기 원리"/><br>
(그림6)_탐색기 원리
<br><br>
위의 그림과 같이 사용자 A는 추적하고 싶은 TX값을 입력하게 된다. 값을 입력받은 탐색기는, TX DB에 있는 UTXO정보를 탐색하여 일치하는 값을 찾는다. 이후 탐색한 UTXO값과 대응하는 TX값을 반환한다.
TX DB에 들어있는 TX에 대응하는 UTXO정보에는 이전 거래를 가리키고 있기 때문에, UTXO정보에 대응하는 TX값은 다음 거래를 가리킨다.
<br>


 ```golang
func searchnexttx(searchtxinfo string, blocknumber int, root *gtree.Node) {
	var list []string
	for s := 1; s < blocknumber; s++ {
		stostring := strconv.Itoa(s)
		b, err := ioutil.ReadFile("../txinfo/" + stostring + ".json")
		if err != nil {
			fmt.Println(err)
		}
		var data []TxData
		json.Unmarshal(b, &data)
		for t := 0; t < len(data); t++ {
			for u := 0; u < len(data[t].Pointer); u++ {
				if data[t].Pointer[u] == searchtxinfo {
					list = append(list, data[t].Info)
				}
			}
		}
	}

	if list != nil {
		for i := 0; i < len(list); i++ {
			newRoot := root.Add(list[i])
			searchnexttx(list[i], blocknumber, newRoot)
		}

	}
}
 ```
 
위의 코드는 입력한 값과 대응되는 UTXO를 찾고, 해당 UTXO값에 대응되는 TX값을 반환하는 코드이다. 이후 재귀함수를 통해서 마지막 트랜잭션에 도달하여, 더 이상 거래내역이 존재하지 않을 때까지 거래를 추적한다.
<br>
<br>

## 출력결과
***
테스트 네트워크 환경 구성 후 트랜잭션 '742718dc9625c6aac6eca4b57d93051208540cd3d65773a41131ce77eee16291'에 관한 추적을 해보았다.
<br><br>
출력결과

 ```
 784c7115114ff74d49ad2624f4d520c2f60fb3ebb8480834c7c51c1427dcffa2
└── 9a1573f71f7ca2b32b17b6d3b09e5634258fbedac65552ac3a90eb5b1d40a880
    ├── 55d91986122006d4537fee88f5ed042e5cd23141dc3db5765d2e72a96ef1f851
    │   ├── dcd7c5f1e3fb2af005b3e48e52d24a73dbdb1c0a6fcd233e7f5ae1f1b699cf09
    │   │   ├── 9dca1dfcd68b2536747776de7ae38b13873f18605a5bad42cd54ec2f4adfde5f
    │   │   └── 46b9f166584cef2e8d6e71274831a2206c89ed852b60f7149799a8f2d4100071
    │   └── df44f4c5e58ba2bdb721eadf6bce0203d880282a020bb0ee623193ea7246df38
    └── 1059678796083d29953aa7771d70a4e36dadd5d698de080fcea8ef8d84fcb15a
6e71274831a2206c89ed852b60f7149799a8f2d4100071
    │   └── df44f4c5e58ba2bdb721eadf6bce0203d880282a020bb0ee623193ea7246df38
    └── 1059678796083d29953aa7771d70a4e36dadd5d698de080fcea8ef8d84fcb15a
        └── effdab52c683c703af3848386f47c4e185dc1de67031a91dab4abe753f623c68
            └── 742718dc9625c6aac6eca4b57d93051208540cd3d65773a41131ce77eee16291
```

다음과 같이 트리구조로 저장되어, 이후 거래내역을 추적할 수 있는 것을 알 수 있다. 

<br>

## 글을 마치며
***
이번 포스팅에서는 비트코인이나 퀀텀에서 사용하는 UTXO정보가 트랜잭션에 남는다는 것을 활용하여, 자산추적 솔루션 개발까지 진행하였다. 본 포스팅이 생소할 수 있는 블록체인 보안에 관심이 있는 분들께, 도움이 되었으면 좋겠다.
<br>
<span style="color:skyblue">중부대학교 보안동아리 SCP 유재겸
<br>
## 참고 문헌
<br>
'박순태, 신용희, 강홍구.(2020).사이버 범죄에 악용되는 암호화폐 불법거래 추적.정보과학회지,38(9),40-47.'

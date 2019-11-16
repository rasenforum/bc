# edu

## 사전준비
- ubuntu image 다운로드 <br>
https://drive.google.com/open?id=1sAN1Feo2PWu2Jv_elu7KcYQ0HlgMUdCU

- ubuntu network 파일 수정
```
$) su //비밀번호 testpw
$) cd /etc/network/
$) vi interfaces
$) 수정 내용
auto lo
iface lo inet loopback

auto enp0s8
iface enp0s8 inet static
address 192.168.57.10
netmask 255.255.255.0
```

- ubuntu 네트워크 재시작
```
$) /etc/init.d/networking restart
```

- bitcoin download url
```
- Ubuntu 접속
- FireFox 브라우저 열기
- https://bitcoin.org/en/ 접속
- menu > resource > Bitcoin Core 클릭
- Bitcoin Core release 클릭
- 0.15.2 ReadMore 클릭
- https://bitcoincore.org/bin/bitcoin-core-0.15.2/ 클릭
- ~ linux-gnu.tar.gz 파일 다운로드
```

- bitcoin download using curl
```
$) curl -O https://bitcoincore.org/bin/bitcoin-core-0.15.2/bitcoin-0.15.2-x86_64-linux-gnu.tar.gz
```


## 비트코인 다운받기
- 타겟 디렉토리 만들기
```
$) cd ~
$) mkdir blockchain_edu1
$) cd blockchain_edu1
$) curl -O https://bitcoincore.org/bin/bitcoin-core-0.15.2/bitcoin-0.15.2-x86_64-linux-gnu.tar.gz
```
<br><br>

## 비트코인 시작하기(from no.4)
### **4.4 비트코인 Core 압축풀기**
```
$) tar -xvzf bitcoin-0.15.2-x86_64-linux-gnu.tar.gz
```
<br><br>

### **4.5 비트코인 실행을 위한 권한 설정 및 절대경로 지정**
```
$) sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-0.15.2/bin/*
$) bitcoind -version
//v0.15.2 확인
```
<br><br>

### **4.6 비트코인 실행하기**
 - bitcoin-qt 또는 bitcoind를 이용해 비트코인 네트워크를 구동

<br><br>

#### 4.6.1 bitcoin-qt :  크로스 플랫폼 애플리케이션 / GUI Interface 무료 툴킷

```
$)./bitcoin-qt
```
위 스크립트 실행시 아래와 같은 화면 출력

![bitcoin_qt](./bit-img/bitcoin_qt.png) 


<br><br>

#### 4.6.2 bitcoind : 개발자 및 운영자를 위한 블록체인 네트워크
- bitcoin 서비스 켜기(MainNet)
```
$) bitcoind -daemon
$) watch -n 2 tree -n 2 /home/test/.bitcoin
```

<br><br>

- bitcoin 서비스 확인
```
$) ps -ef | grep bitcoin
$) netstat -anp | grep 8332
$) watch -n 1 du ~/.bitcoin
```
<br>

- genesis block 확인

![bitcoin_qt](./bit-img/bitcoin_genesis.png) 
$) sudo apt-get install bless (hex editor tool)
$) bless
$) block 파일 열기



> bitcoin core 포트

- MainNet : 8333(P2P), 8332(RPC)
- TestNet : 18333(P2P), 18332(RPC)
- RegNet : 18444(P2P), 18332(RPC)

<br>

- bitcoin 서비스 중지
```
$) bitcoin-cli stop
```
<br>

> 잠깐! 비트코인은 어디에 연결되어 블록을 받아오는 것일까?

```
https://github.com/bitcoin/bitcoin/blob/4b51ed89cfce9870a20d75001fae3b68ac1dfd86/src/chainparams.cpp
```

![bitcoin_dns](./bit-img/bitcoin_dns.png | width=100) 



1. Seed 서버가 랜덤으로 Node IP(Public) 제공(IP가 변경되는 주기는 몇초 이상) 
$) dig seed.bitcoinstats.com +short

2. 오프라인일 경우 캐쉬에 저장된 IP에 접속 시도




<br><br>

> 비트코인 노드의 위치를 확인하기

- https://bitnodes.earn.com/#join-the-network

<br><br>

> 비트코인 MainNet/TestNet을 실습하기 어려운 이유
- Full BlockChain을 저장할 수 있는 디스크 용량 필요<br>
1. MainNet - https://www.blockchain.com/charts/blocks-size<br>
2. TestNet - https://blockchair.com/bitcoin/testnet <br>

- Transaction을 발생시키기 위한 MainNet 비트코인 보유 필수
  (현재 비트코인은 얼마?)
 1. 거래소 - https://www.bithumb.com/



<br><br>

> 다른 대안은?
 - reggression test network을 사용하기



<br><br>
## 5. 시작하기(regTest)

### 5.1. 비트코인 코어 데몬 실행 및 확인
```
$) rm -rf ~/.bitcoin
$)bitcoind -regtest -daemon -deprecatedrpc=generate
$) cd ~/.bitcoin
```
※ 응답값 : Bitcoin server starting <br>
※ STOP : ./bitcoin-cli -regtest stop <br>
<br>

### (optional)5.2 RPC 프로토콜을 위한 bitcoin.conf 파일 설정
```
$) vi ~/.bitcoin/bitcoin.conf
$) bitcoin.conf 내용

txindex=1
regtest=1
server=1

rpcallowip=0.0.0.0/0
rpcport=8332
rpcconnect=127.0.0.1

disablewallet=0

rpcuser=test
rpcpassword=testpw
[regtest]

$) chmod 0600 bitcoin.conf
```


<br>

### 5.2 블록생성(regtestMode에서만 가능)
 
```
$)bitcoin-cli -regtest generate 101
//Response로 Block Hash 값을 Return
//Reward BitCoin은 100컨펌을 기다려야 하기 때문에 101개를 만듦.
```
> 참고 : https://en.bitcoin.it/wiki/Confirmation<br>
...Freshly-mined coins cannot be spent for 100 blocks...

<br><br>

### 5.3 블록 정보 확인하기
```
$)bitcoin-cli -regtest getblock 위 블록해쉬값
//Final Index, Final Index-1 넣어보기 및 Confirmation 의미 확인
```
<br><br>

### 5.4 Balance 체크
```
$)bitcoin-cli -regtest getbalance
※ regtest모드에서는 최초 150블록에만 50BTC의 보상을 줌  
```
<br><br>


> 블록데이타 위치

```
$)cd ~/.bitcoin/regtest/blocks
```

<br><br>

### 5.5 new Address 생성
 
```
$)bitcoin-cli -regtest getnewaddress
$)NEW_ADDRESS=응답값
$)echo $NEW_ADDRESS
```
※ 응답값 : 비트코인 주소


<br><br>

### 5.6 transaction 발생 후 밸런스 체크
```
$)bitcoin-cli -regtest sendtoaddress $NEW_ADDRESS 10.00
//NEW_ADDRESS로 10BTC 보내기, return값(txid)
$)bitcoin-cli -regtest getbalance
//return값(49.99996680) : 블록은 아직 생성되기 전(transaction fee만 소모된 상태)
```

<br><br>
 
### 5.7 transaction 후 조회
```
$)bitcoin-cli -regtest listunspent 0
```
※ 응답값 : 동일한 transactionID로 vout이 2개있음(array처럼 순서가 있으므로 vector라는 이름씀 v)
                confirmation 확인

<br><br>

### 5.8 블록 만들어 보기
```
$)bitcoin-cli -regtest generate 1
$)bitcoin-cli -regtest listunspent 0
$)UTXO_TXID = Coinbase Transaction ID(새로운 50BTC짜리)
$)UTXO_VOUT = 0 
```
※ 응답값 : confirmation 확인

<br><br>

### 5.9 새로운 어드레스 만들기

```
$)bitcoin-cli -regtest getnewaddress
$)NEW_ADDRESS=응답값
$)bitcoin-cli -regtest getreceivedbyaddress $NEW_ADDRESS
//새로 만들었으므로 0BTC가 출력됨
$)bitcoin-cli -regtest getreceivedbyaddress 이전 ADDRESS도 확인해보기
```


<br><br>


### 5.10 RAW TRANSACTION 만들기

```
$)bitcoin-cli -regtest createrawtransaction '''
    [
      {
        "txid": "'$UTXO_TXID'",
        "vout": '$UTXO_VOUT'
      }
    ]
    ''' '''
    {
      "'$NEW_ADDRESS'": 49.9999
    }'''
$)RAW_TX=위 명령어 응답값(ex : 02000000014d586994365b7641e19f8e16edc5b6a2b41650b38f582f90d9e97cafe72845700000000000ffffffff01f0ca052a0100000017a914bcea2054333005b097526b0f56c35385ef8052948700000000)
```
※ 어떤 UTXO를 사용할 건지(TrxID, index)와 보낼 주소 필요<br>
※ CoinBase Transaction(Vout)이 Vin으로 들어간 것 확인
 
<br><br>

### 5.11 RAW TRANSACTION DECODE 해보기(UTXO)

```
$) bitcoin-cli -regtest decoderawtransaction $RAW_TX
```

<br><br>

### 5.12 RAW TRANSACTION에 싸인하기

```
$) bitcoin-cli -regtest signrawtransaction $RAW_TX
$) SIGNED_RAW_TX=응답값의 hex값
```

<br><br>


### 5.13 SIGNED_RAW TRANSACTION 보내기

```
$) bitcoin-cli -regtest sendrawtransaction $SIGNED_RAW_TX
```

<br><br>

### 5.14 MemPool에 있는 TRANSACTION 확인하기
```
$)bitcoin-cli -regtest getrawmempool
```
<br><br>

### 5.15 Transaction(MemPool) Block안으로 넣기
```
$)bitcoin-cli -regtest generate 1
$)bitcoin-cli -regtest getrawmempool
//해소된 memPool 확인
```
<br><br>

### 5.16 Block 확인하기
```
$)bitcoin-cli -regtest getblock 위에서 만든 blockHash
//블록안에 담긴 Hash값 확인
```
<br><br>

### 5.17 Balance 확인하기
```
$) bitcoin-cli -regtest listunspent 0
//새로운 Account에 BTC가 Transfer된 것을 확인
```

<br><br>



## 6. 시작하기(TestNet)
### 6.1 bitcoin generator(address)
- https://bitcoinpaperwallet.com/bitcoinpaperwallet/generate-wallet.html?design=alt-testnet


<br><br>

### 6.2 bitcoin testnet 충전하기
- https://coinfaucet.eu/en/btc-testnet/
//0.03 BTC
- https://bitcoinfaucet.uo1.net/send.php
//0.000005 BTC

<br><br>

### 6.3 bitcoin testnet 확인
- https://testnet.blockchain.info/address/

<br><br>

## 7. BitCoin Client SDK(JSON RPC)
### 7.1 IntellJ CE or STS or 설치하기
https://www.jetbrains.com/idea/ 

<br><br>

### 7.2 maven 설정
- pom.xml 설정
```
<dependency>
            <groupId>wf.bitcoin</groupId>
            <artifactId>bitcoin-rpc-client</artifactId>
            <version>1.1.0</version>
</dependency>
```
<br><br>

- BitcoinClient Util 클래스 생성
```
public class BitcoinClient extends BitcoinJSONRPCClient {
    static String rpcURL = "192.168.57.10";
    static String rpcPort = "8332";
    static String userName = "test";
    static String userPassword = "testpw";

    BitcoinClient() throws MalformedURLException {
        super("http://" + userName + ":" + userPassword + "@" + rpcURL + ":" + rpcPort + "/");
    }
}
```
<br><br>

#### 7.2.1 블록체인 정보 관련 API
```
public class Case1_BlockChainInfo {
    public static void main(String[] args) throws MalformedURLException {
        System.setProperty("java.util.logging.SimpleFormatter.format", "[%1$tF %1$tT] [%4$-7s] %5$s %n");
        String newLine = System.getProperty("line.separator");

        BitcoinClient bitcoinClient = new BitcoinClient();

        System.out.println("Balance");
        System.out.println(bitcoinClient.getBalance().toPlainString() + newLine);

        System.out.println("BestBlockHash");
        System.out.println(bitcoinClient.getBestBlockHash() + newLine);

        System.out.println("WalletInfo");
        System.out.println(bitcoinClient.getWalletInfo() + newLine);

        BitcoindRpcClient.BlockChainInfo bci = bitcoinClient.getBlockChainInfo();

        System.out.println("BlockInfo");
        System.out.println(bci.blocks() + newLine);

        System.out.println("BlockHash");
        System.out.println(bci.bestBlockHash());
    }
}

```

#### 7.2. 블록체인 정보 관련 API

<br><br>

## 8. bitcoin explorer 사용하기
### 8.1 github repository 접속(not official)
https://github.com/janoside/btc-rpc-explorer 접속

참고 : MIT license<br>
- 이 소프트웨어를 누구라도 무상으로 제한없이 취급해도 좋다. 단, 저작권 표시 및 이 허가 표시를 소프트웨어의 모든 복제물 또는 중요한 부분에 기재해야 한다.
- 저자 또는 저작권자는 소프트웨어에 관해서 아무런 책임을 지지 않는다.

<br><br>
### 8.1 npm을 이용한 explorer 설치
```
$)npm install -g btc-rpc-explorer
```

### 8.2 작업 디렉토리에 .env 파일 설정하기
```
$) vi .env
BTCEXP_HOST=0.0.0.0
BTCEXP_PORT=8080
```

<br><br>

### 8.3 bitcoin explorer 띄우기
```
btc-rpc-explorer --port 8080 --bitcoind-port 18332 --bitcoind-cookie ~/.bitcoin/regtest/.cookie
``` 

### 8.4 explorer port 확인하기
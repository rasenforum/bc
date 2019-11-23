# DAY_3_Ethereum

## **목차**
* [HandsOn 소개](#1.-handson-소개)
* [1. 이더리움 설치](#1.-handson-소개)
* [2. geth를 이용한 private network 구축](#2-geth를-이용한-private-network-구축)
* [3. puppet을 이용한 private network 구축](#3-puppet을-이용한-private-network-구축)
* [4. Truffle Framework 사용해보기](4-truffle-framework-사용해보기)

<br><br>

### Handson 소개
Linux(Ubuntu)상에서 geth, puppet, remix, truffle 등을 설치하고 각 단계별 실습을 통해 Ethereum의 동작원리를 알아본다.

<br><br>

### 1. 이더리움 설치

#### (Optional)사전 준비사항(HyperVisor를 이용한 Linux 환경이 없을 경우)
- Ubuntu Install Using Docker

https://github.com/fofoeet/knowledge/tree/master/DOCKER

<br>



#### geth Binary 설치 
> $) Binary 직접 다운로드 (1.1 ~ 1.3 생략)
```
$) curl -O https://gethstore.blob.core.windows.net/builds/geth-alltools-linux-amd64-1.9.7-a718daa6.tar.gz
$) tar -xvzf geth-alltools-linux-amd64-1.9.7-a718daa6.tar.gz
$) cd geth-alltools-linux-amd64-1.9.7-a718daa6
$) sudo cp ./* /usr/local/bin
$) geth version
```
<br>

#### (Optional) 1.1 go 설치 

```
$) mkdir -p ~/day3/files
$) cd ~/day3/files
$) curl -O https://dl.google.com/go/go1.10.8.linux-amd64.tar.gz
$) tar -C /usr/local -xzf go1.10.8.linux-amd64.tar.gz
$) export PATH=$PATH:/usr/local/go/bin
$) go version //go 버전확인 1.9.2이상 권장
```

<br><br>

#### (Optional) 1.2 geth 설치-SourceCode
```
$) git clone https://github.com/ethereum/go-ethereum.git
$) cd go-ethereum/
$) make all
```
<br><br>


#### (Optional) 1.3 자주 사용하는 exec파일 이동
```
$) cd build/bin
$) cp puppeth /usr/local/bin/
$) cp geth /usr/local/bin/
$) cp bootnode /usr/local/bin/
```
<br><br>

#### 1.4 : 작업환경을 위한 iTerm 콘솔창 분할(아래와 같이 콘솔창을 5칸으로 분할한다)

> 화면 참조

<br><br>

### [쉬어가기] Ethereum MainNet 접속해보기
```
$) geth console --nousb
```
> Linux는 기본적으로 ~/.ethereum 에 Blockchain Data 가 저장됨

<br><br>

```
$) cd /home/test/.ethereum
$) watch -n 1 du -h /home/test/.ethereum
```


<br><br>

### 2. geth를 이용한 private network 구축

#### 2.1 작업폴더 생성 및 이동(제일 상단 터미널)
```
$) mkdir ~/day3/ethereum/geth
$) cd ~/day3/ethereum/geth//command+option+i
```

<br><br>

#### 2.2 BootNode 설정하기(최초에 접속하기 위한 노드 설정)

```
$) bootnode --genkey=boot.key
$) cat boot.key //키주소 복사해놓고 아래서 사용한다
$) bootnode -nodekeyhex 키주소 -writeaddress //hex값으로 나온 데이타를 아래 명령어의 response와 비교한다
$) bootnode -nodekey boot.key -verbosity 9
//위 명령어 실행시 self=enode:주소 이런식으로 나오게 되는데 이 때 주소 값을 어딘가에 잘 복사해둔다. 이 주소가 ethereum network를 가장 먼저 연결하기 위한 bootnode가 된다(일종의 seed)
$) netstat -anp | grep 30301 //다른 창에서 실행해본다
```
<br><br>


#### 2.3 genesis block 설정파일 생성하기(왼쪽 상단 첫번째 터미널)

```
$) vi genesisBlock.json //아래 파일 복사 + 붙혀넣기
{
    "config": {
    "chainId": 13579,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip150Hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "ethash": {}
    },
    "nonce": "0x0",
    "timestamp": "0x00",
    "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "gasLimit": "0x8000000",
    "difficulty": "0x80000",
    "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "alloc": {
    "0000000000000000000000000000000000000000": {
      "balance": "0x1"
    }
    },
    "number": "0x0",
    "gasUsed": "0x0",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```
<br><br>

#### 2.4 노드별 설정 초기화 및 비밀번호 파일 만들기
```
$)geth --nousb --datadir peerData1 init genesisBlock.json  //왼쪽 상단 터미널에서 실행
$)geth --nousb --datadir peerData2 init genesisBlock.json  //오른쪽 상단 터미널에서 실행
//위 명령어를 정상적으로 수행하면 Successfully wrote genesis state라는 결과 값이 출력되며 chainData 폴더 안에 genesisBlock 초기화 파일이 생성된다

$) echo "testpw" > peerData1/password.txt
$) echo "testpw" > peerData2/password.txt

```
<br><br>

#### 2.5 node별 account 만들기
```
$)geth --datadir ./peerData1 account new //왼쪽 상단 터미널에서 실행
$)geth --datadir ./peerData2 account new //오른쪽 상단 터미널에서 실행
//위 명령어를 입력하게 되면 response값으로 주소값이 출력되며 chainData/keystore/이곳에 keystore 값이 저장된다. 어쨌든 이 주소를 어딘가에 저장을 해둔다.
```
<br><br>


#### 2.6 디렉토리 구조를 확인해보자
```
$) tree -L 4 peerData1 //왼쪽 창에서 실행
$) tree -L 4 peerData2 //오른쪽 창에서 실행
```
<br><br>


#### 2.7 ethereum 시작하기
```
$) vi runPeer1.sh
> geth --allow-insecure-unlock --unlock "0" --password peerData1/password.txt --nousb --datadir peerData1 --syncmode 'full' --port 9091 --rpc --rpcport 7071 --rpcaddr 127.0.0.1 --rpccorsdomain "*" --rpcapi "eth,net,web3,personal,miner" --bootnodes 'enode://fc733bf9ff476a100b19aa558f6cc8e0b5adf36a0147c94eb5989edb80295eb38d16eeadd2a0b3734da08c5007ed2396d0c9adbb5404f1e3d25ef2bf1925f2a1@127.0.0.1:0?discport=30301' --networkid 13579 --gasprice '1' console 2 >> /home/test/ethereum/geth/peer1.log

$) vi runPeer2.sh     
> geth --allow-insecure-unlock --unlock "0" --password peerData2/password.txt --nousb --datadir peerData2 --syncmode 'full' --port 9092 --rpc --rpcport 7072 --rpcaddr 127.0.0.1 --rpccorsdomain "*" --rpcapi "eth,net,web3,personal,miner" --bootnodes 'enode://fc733bf9ff476a100b19aa558f6cc8e0b5adf36a0147c94eb5989edb80295eb38d16eeadd2a0b3734da08c5007ed2396d0c9adbb5404f1e3d25ef2bf1925f2a1@127.0.0.1:0?discport=30301' --networkid 13579 --gasprice '1' console 2 >> /home/test/ethereum/geth/peer2.log

```

> 새로운 창을 띄운다

아래 명령어 수행하게 되면 CLI 출력창이 나오게 되는데 이 때 현재 account의 잔고를 확인하도록 한다.
Peer1과 Peer2의 잔고를 모두 확인하도록 한다(왼쪽, 오른쪽). BlockHeight도 확인해본다.
```
$) geth attach http://localhost:7071
$) > eth.accounts
$) > eth.getBalance(eth.accounts[0])
$) > eth.blockNumber
```

<br><br>

#### 2.8 mining 시작하기

```
$) geth attach http://localhost:7071
$) > miner.start(1) //왼쪽 노드에서만 실행한다. 시간이 좀 걸린다...
```
<br><br>

> 새로운 창 하나를 열고 peer2에 접속해서 마이닝을 하고 있는지 확인해보자

<br><br>

#### 2.9 mining 중지 & Account별 balance 확인하기

```
$) eth.mining // true
$) miner.stop() //null이 나오는게 정상이다..
$) eth.mining // false
$) eth.blockNumber //Peer1,Peer2의 블록이 동기화 되었는지 확인한다.
$) eth.getBalance(eth.accounts[0]) // wei 출력
$) web3.fromWei(eth.getBalance(eth.accounts[0]), "ether") // Ether로 출력
```
>위 명령어를 수행했을 때 나오는 기본 단위는 wei다.<br>
>https://etherconverter.online/에 접속하여 블록 당 얼마의 mining reward를 주는지 확인한다

<br><br>

#### 2.10 돈을 보내보기(sendTransaction)
> 이제 Peer1->Peer2로 돈을 보내보자!
```
$)(optional)eth.sendTransaction({from:eth.accounts[0], to:"보낼주소", value: web3.toWei(1, 'ether')}) //위 명령어를 수행하면 에러가 날 것이다. 왜냐하면 지갑을 임의로 사용할 수 없기 때문이다. 그래서 초반에 account를 만들 때 입력했던 비밀번호를 통해 지갑을 활성화 시켜주어야 한다.
$)(optional)web3.personal.unlockAccount(eth.accounts[0],"비밀번호", 15000)
//15000은 활성화 지속 시간이다. 이 이후에는 다시 락킹을 풀어줘야 한다.
$)eth.sendTransaction({from:eth.accounts[0], to:"보낼주소", value: web3.toWei(1, 'ether')}) //다시 한번 보내본다. 이 명령어를 정상적으로 수행하면 transaction ID가 나온다. 어디에 적어두자.
```
<br><br>

#### 2.11 Transaction memPool 확인하기

```
$)eth.getBlock("pending", true).transactions //아직 블록에 묶이지 않았으므로 pending 상태(mempool 영역에 있는 상태)이다. Peer1과 Peer2 두 곳 모두 전파된다. 
//Hash를 확인해보자
```

<br><br>

#### 2.12 Block 확인

```
$) eth.getBlock(블록넘버) // 위 블록은 예상되는 블록 번호이므로 null값이 출력된다.
```
<br><br>

#### 2.13 mining 재시작

```
$) > miner.start(1) //mining을 다시 시작한다. 
$) > miner.stop() //시작 후 블록이 2~3개 묶이면 멈춘다
```
<br><br>

#### 2.14 Transaction 및 Block 확인

```
$) web3.eth.getBlock(블록넘버);
$) web3.eth.getBlock(블록넘버).hash;
$) web3.eth.getBlock("블록해쉬");
```
<br><br>

#### 2.15 balance 및 히스토리 확인

```
$)eth.getBalance(eth.accounts[0]) //오른쪽 address의 밸런스를 확인한다
$)eth.getBalance(eth.accounts[0],블록넘버) //블록 전/후 값을 확인한다
$)eth.getBlock(블록넘버).hash;
$)eth.getTransaction(transactionHash)
```

<br><br>

### 3. puppet을 이용한 private network 구축

#### 3.1 puppeth폴더 생성 후 이동

```
$) cd ~/handsOn1
$) mkdir puppeth
$) cd puppeth 
```

<br><br>

#### 3.2 주소 만들기

```
$) geth --datadir ./peerData1 account new //왼쪽 상단 터미널에서 실행
$) geth --datadir ./peerData2 account new //오른쪽 상단 터미널에서 실행
$) echo 'testpw' > peerData1/password.txt //network 구동할 때 필요
$) echo 'testpw' > peerData2/password.txt //network 구동할 때 필요
```

<br><br>

#### 3.3 puppeth 실행
> puppeth은 geth 1.8이상이면 모두 내장되어 있다. 별도의 설치가 필요 없다(아까전에 /usr/local/bin에서 이동했으므로 바로 실행 가능하다)

```
$) puppeth

Please specify a network name to administer (no spaces or hyphens, please)
> kbnetwork //networkID - 식별 아이디로 사용! 아무거나 상관 없음. 만약 이전에 동일한 네트워크 아이디를 만들었었다면 /Users/사용자이름/.puppeth 밑에 있는 데이타를 삭제한다

What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
> 2

What would you like to do? (default = create)
> 1. Create new genesis from scratch
 2. Import already existing genesis

Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 2 // 합의 알고리즘 방식 선택, 

How many seconds should blocks take? (default = 15)
> 5 // 블록생성 주기

Which accounts are allowed to seal? (mandatory at least one) //합의(검증)을 위한 노드들을 선정한다
> 0x주소1
> 0x주소2
> 0x그냥앤터

Should the precompile-addresses (0x1 .. 0xff) be pre-funded with 1 wei? (advisable yes)
> yes

Which accounts should be pre-funded? (advisable at least one) //만든 어드레스에 balance를 충전해준다.
> 0x주소1
> 0x주소2
> 0x그냥앤터

Specify your chain/network ID if you want an explicit one (default = random)
> 2468 //network 식별 ID. 1,2,3은 디폴트로 사용중이므로 그 외의 것을 적용할 것
INFO [11-05|21:43:08.680] Configured new genesis block

$)그 다음에 What would you like to do? (default = stats) 이런 문구가 다시 나오면 Ctrl+C를 눌러 빠져나온다.
```

<br><br>


#### 3.4 genesis file 확인하기
```
$) cat ~/.puppeth/kbnetwork
```

<br><br>

> ethstat 을 실행하기 위한 docker 설치하기

#### 3.5 ethstat 실행하기
```
$) puppeth
$) Please specify a network name to administer (no spaces, hyphens or capital letters please)
> kbnetwork

$) What would you like to do? (default = stats)
> 4. Deploy network components

$) What would you like to deploy? (recommended order)
> 1. Ethstats


$) Which server do you want to interact with?
> 1. Connect another server

$) What is the remote server's address ([username[:identity]@]hostname[:port])?
> 192.168.56.10

$) The authenticity of host 'localhost:22 (127.0.0.1:22)' can't be established.
SSH key fingerprint is 77:d4:de:ac:b4:ff:79:cb:94:41:6c:8c:1f:bb:11:1d [MD5]
Are you sure you want to continue connecting (yes/no)? 
> yes

$) What's the login password for test at localhost? (won't be echoed)
> testpw

$) Which port should ethstats listen on? (default = 80)
> 8080


$) Allow sharing the port with other services (y/n)? (default = yes)
> no

$) What should be the secret password for the API? (must not be empty)
> testpw

```

<br><br>

#### 3.6 ethstat 접속해보기
```
http://192.168.56.10:8080 으로 접속 해보기
```

<br><br>
#### 3.7 bootNode 만들기
```
$) puppeth
$) network name ? 
> kbnetwork

$) what is the login password?
> testpw

$) what would you like to do?
> 4. Manage network components

$) next question?
> 2. Deploy net network component

$) What would you like to deploy?
> Bootnode

$) Which server do you want?
> 1. 192.168.56.10

$) Where should data be stored on the remote machine?
> /home/test/day3/ethereum/puppeth/bootNodeData

$) Which TCP/UDP port to listen on? (default = 30303)
> 30305

생략

$) What should the node be called on the stats page?
> bootnode
```

<br><br>

> 잠깐! public ethstat 사이트를 확인해보자 

https://ethstats.net/

<br><br>

#### 3.8 SealNode 만들기
```
위와 동일하게 진행(구두 설명)
port : 30303, 
disply name : sealnode
dataStor 위치 : /home/test/day3/ethereum/puppeth/sealNode
Keyjson : sealNode(peerData1)의 keystore/UTC 파일 전체를 넣어준다
```

<br><br>

#### 3.9 Docker 안으로 접속하기
```
$) docker exec -it kbnetwork_sealnode_1 geth attach
$) eth.mining
```

<br><br>

#### 3.10 Bootnode의 주소 알기
```
$) (optional) docker exec -it kbnetwork_bootnode_1 geth attach ipc:/home/test/.ethereum/geth.ipc(안되면 다음 > 버전마다 상이)
$) docker exec -it kbnetwork_bootnode_1 geth attach
$) admin.nodeInfo.enode
> response 예시 : enode://87fc8b4caee52741677c887a36ff9b7a534432d2f55ce15a27db4039cfc664c36505a80374190b60fbb72db52fcaa27c8f7baa82ffa18921b6fca8f235a68572@192.168.56.10:30305
```

<br><br>

#### 3.11 Wallet 생성하기
```
Which port should the wallet listen on? (default = 80)
> 8081

Allow sharing the port with other services (y/n)? (default = yes)
> no

Where should data be stored on the remote machine?
> /home/test/day3/ethereum/puppeth/wallet

Which TCP/UDP port should the backing node listen on? (default = 30303)
> 30306

Which port should the backing RPC API listen on? (default = 8545)
>

What should the wallet be called on the stats page?
> wallet
```

> 참고 : https://www.myetherwallet.com/

<br><br>

#### 3.12 Faucet 으로 충전하기
```
Which port should the faucet listen on? (default = 80)
> 8082

Allow sharing the port with other services (y/n)? (default = yes)
> no

How many Ethers to release per request? (default = 1)
>

How many minutes to enforce between requests? (default = 1440)
> 1

How many funding tiers to feature (x2.5 amounts, x3 timeout)? (default = 3)
>

Enable reCaptcha protection against robots (y/n)? (default = no)
>
WARN [05-20|12:51:13] Users will be able to requests funds via automated scripts

Where should data be stored on the remote machine?
> /home/test/day3/ethereum/puppeth/faucet

Which TCP/UDP port should the light client listen on? (default = 30303)
> 30307

What should the node be called on the stats page?
> faucet

Please paste the faucet's funding account key JSON:
> 위에서 다운받은 JSOP 파일을 넣는다

What's the unlock password for the account? (won't be echoed)
>

Permit non-authenticated funding requests (y/n)? (default = false)
> y
```


> 참고 : docker exec --privileged -it ubuntu bash


<br><br>

### 4. Truffle Framework 사용해보기

> 사전 필요 S/W
 1. node.js
```
$) sudo apt-get install npm
$) sudo npm install -g truffle
$) sudo chown -R test /usr/local/lib/node_modules
```

<br>

2. ganache 다운로드 및 실행
```
$) ubuntu desktop 접속
$) firefox 실행
$) google 에서 ganache 검색
$) ganache 최신버전 다운로드(파일명.AppImage)
$) chmod a+x ganache-1.3.0-x86_64.AppImage
$) ./ganache-1.3.0-x86_64.AppImage
//기본으로 7545 포트가 Listen 된다
```

<br><br>

> ganache 접속해보기
ganache는 기본적으로 event가 발생하면 자동으로 블록을 생성한다
```
$) geth attach localhost:7545
$) eth.sendTransaction({from:eth.accounts[0], to:"보낼주소", value: web3.toWei(1, 'ether')})
$) ganache 콘솔을 확인한다
```


<br><br>

#### 4.1 Truffle Framework다운로드(Node.js 5.0 이상 필요)
```

$) cd ~/handsOn1
$) mkdir truffle
$) cd truffle
$) sudo npm install -g truffle
```
<br><br>

#### 4.2 Truffle 샘플앱 다운로드 받아보기(그냥 예제....)
```
$) mkdir sample
$) cd sample
$) sudo truffle unbox metacoin //Unbox List는 여러가지가 있다. https://truffleframework.com/boxes 참조할 것
$) tree 로 구조 확인
```
<br><br>

#### 4.3 Truffle Project 생성

```
$) mkdir sample2
$) cd sample2
$) truffle init // SmartContract 기본 구성
```

<br><br>   

#### 4.4 SmartConract 만들기(HelloWorld)

```
$) truffle create contract HelloWorld
$) cd contracts
$) vi HelloWorld.sol
>
pragma solidity ^0.4.23;


contract HelloWorld {
    string public printWord;

     constructor() public {
        printWord = "Hello World Ethereum";
     }

    function print() public constant returns(string _printWord) {
        return printWord;
    }
}
```

<br><br>

#### 4.5 배포 파일 만들기
```
$) truffle create migration deploy
$) 위 명령어가 완료되면 migrations 폴더에 배포 파일이 생성됨
```

<br><br>

#### 4.6 배포 파일 수정
```
$) vi 배포파일.js
>
const HelloWorld = artifacts.require("./HelloWorld");

module.exports = function(_deployer) {
   _deployer.deploy(HelloWorld);
};
```


<br><br>


#### 4.7 network설정(truffle.js 또는 truffle-config.js)

```
$) vi truffle.js
$) > 
module.exports = {
networks: {
     development: {
      host: "127.0.0.1",     // Localhost (default: none)
      port: 7071,            // Standard Ethereum port (default: none)
      network_id: "*",       // Any network (default: none)
     }
  },
  mocha: {
    // timeout: 100000
  },

  // Configure your compilers
  compilers: {
    solc: {
       version: "0.4.23",    // Fetch exact version from solc-bin (default: truffle's version)
    }
  }
};
```
<br><br>
 
#### 4.8 SmartContract 컴파일
```
$) truffle compile
//ABI Byte 코드로 떨어짐
```

<br><br>

#### 4.9 SmartContract Build(truffle)
아래 명령어를 수행하면 build되며 네트워크에 Deploy된다. Deploy된 Contract의 주소를 적어둔다.
```
$) cd ..
$) truffle compile // solidity -> ByteCode/ABI
$) truffle migrate // default로 development로 가짐
response 중 : Hello: 0x630cc872af35d1037008a77fb1159c8f24f415f1 이 값이 contract address이다
```
<br><br>

#### 4.10 SmartContract execute
geth console로 deploy된 contract의 코드를 호출하기 위해서는 다음 절차를 따른다.
>**1. 호출한 function명을 keccak256(sha-3)으로 Hashing한다**
예) web3.sha3("HelloWorld()") //실행하면 hexCode가 추출된다. 추출된 값의 첫 4byte(8자리까지) + function parameter hex lpad(32byte)를 합쳐 Data부를 만든다.
예) 0x7fffb7bd0000000000000000000000000000000000000000000000000000000000000000

>**2. curl로 호출해본다**

```
curl http://localhost:7501 -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_call","params":[{"from": "0x보내는사람주소", "to": "0xContract주소", "data":"0x7fffb7bd0000000000000000000000000000000000000000000000000000000000000000"},"latest"],"id":1}'
//보내는 사람은 그냥 peerData1의 초기 주소를 넣는다

예시) curl http://localhost:7071 -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_call","params":[{"from": "0x10635e8c194757c31807b3e85e5dc33ac2db2bee", "to": "0x8127a89A4Afb7629c62b84676B761A50C5b1708C", "data":"0x13bdfacd0000000000000000000000000000000000000000000000000000000000000000"},"latest"],"id":1}'
```
>**3. response 값은 hex값으로.. 나올텐데 이를 위해 아래 사이트에 접속하여 변환해본다**
```
{"jsonrpc":"2.0","id":1,"result":"0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000001448656c6c6f20576f726c6420457468657265756d000000000000000000000000"}
```
https://codebeautify.org/hex-string-converter


# Day3_Fabric_Ethereum

#### (Optional)Fabric Docker Refresh
```
$) docker stop $(docker ps -a -q)
$) docker rm $(docker ps -a q)
$) docker rmi $(docker images -q dev*)
```

<br><br>

#### 1. download_fabric.sh
```
#!/bin/bash

export VERSION=1.4.1
export THIRDPARTY_VERSION=0.4.15

#peer orderer ccenv javaenv tools ca : v1.4.1
for IMAGES in peer orderer ccenv javaenv tools ca; do
    echo "==> FABRIC IMAGE: $IMAGES"
    echo
    docker pull hyperledger/fabric-$IMAGES:$VERSION
    docker tag hyperledger/fabric-$IMAGES:$VERSION hyperledger/fabric-$IMAGES
done

#zookeeper, kafka, couchdb : v0.4.15
for IMAGES in couchdb kafka zookeeper; do
    echo "==> THIRDPARTY DOCKER IMAGE: $IMAGES"
    echo
    docker pull hyperledger/fabric-$IMAGES:$THIRDPARTY_VERSION
    docker tag hyperledger/fabric-$IMAGES:$THIRDPARTY_VERSION hyperledger/fabric-$IMAGES
done

docker pull hyperledger/fabric-baseos:amd64-0.4.15
```

<br>

>  쉘파일 실행 권한 주기
```
$) chmod +x download.sh
```

<br>

>  쉘파일 실행하기
```
$) ./download.sh
```

<br>


#### (필수) crypto-config.yaml

```
OrdererOrgs:
  - Name: Orderer
    Domain: kb.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org
    Domain: org.kb.com
    CA:
        Country: KOREA
        Locality: SEOUL
        Organization: KB
    Template:
      Count: 3
    Users:
      Count: 1
    Admins:
      Count: 1
```
<br>

#### (필수) configtx.yaml


```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:

    # SampleOrg defines an MSP using the sampleconfig.  It should never be used
    # in production but may be used as a template for other definitions
    - &OrdererOrg
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrdererOrg

        # ID to load the MSP definition as
        ID: OrdererMSP

        # MSPDir is the filesystem path which contains the MSP configuration
        MSPDir: crypto-config/ordererOrganizations/kb.com/msp

    - &Org
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrgMSP

        # ID to load the MSP definition as
        ID: OrgMSP

        MSPDir: crypto-config/peerOrganizations/org.kb.com/msp

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org.kb.com
              Port: 7051

################################################################################
#
#   SECTION: Application
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for application related parameters
#
################################################################################
Application: &ApplicationDefaults

    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:

################################################################################
#
#   SECTION: Orderer
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for orderer related parameters
#
################################################################################
Orderer: &OrdererDefaults

    # Orderer Type: The orderer implementation to start
    # Available types are "solo" and "kafka"
    OrdererType: solo

    Addresses:
        - orderer.kb.com:7050

    # Batch Timeout: The amount of time to wait before creating a batch
    BatchTimeout: 1s

    # Batch Size: Controls the number of messages batched into a block
    BatchSize:

        # Max Message Count: The maximum number of messages to permit in a batch
        MaxMessageCount: 500

        # Absolute Max Bytes: The absolute maximum number of bytes allowed for
        # the serialized messages in a batch.
        AbsoluteMaxBytes: 99 MB

        # Preferred Max Bytes: The preferred maximum number of bytes allowed for
        # the serialized messages in a batch. A message larger than the preferred
        # max bytes will result in a batch larger than preferred max bytes.
        PreferredMaxBytes: 512 KB

    Kafka:
        # Brokers: A list of Kafka brokers to which the orderer connects
        # NOTE: Use IP:port notation
        Brokers:
            - 127.0.0.1:9092

    # Organizations is the list of orgs which are defined as participants on
    # the orderer side of the network
    Organizations:

################################################################################
#
#   Profile
#
#   - Different configuration profiles may be encoded here to be specified
#   as parameters to the configtxgen tool
#
################################################################################
Profiles:

    OneOrgOrdererGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
        Consortiums:
            KBConsortium:
                Organizations:
                    - *Org
    OneOrgChannel:
        Consortium: KBConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org
```

<br><br>



#### 2. generate_artifact.sh
```
#!/bin/sh
#
# Copyright IBM Corp All Rights Reserved
#
# SPDX-License-Identifier: Apache-2.0
#
export PATH=$GOPATH/src/github.com/hyperledger/fabric/build/bin:${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}
CHANNEL_NAME=kbchannel

# create folder if not exist
mkdir -p config
mkdir -p crypto-config


# remove previous crypto material and config transactions
rm -fr config/*
rm -fr crypto-config/*

# generate crypto material
./bin/cryptogen generate --config=./crypto-config.yaml
if [ "$?" -ne 0 ]; then
  echo "Failed to generate crypto material..."
  exit 1
fi

# generate genesis block for orderer
./bin/configtxgen -profile OneOrgOrdererGenesis -outputBlock ./config/genesis.block
if [ "$?" -ne 0 ]; then
  echo "Failed to generate orderer genesis block..."
  exit 1
fi

# generate channel configuration transaction
./bin/configtxgen -profile OneOrgChannel -outputCreateChannelTx ./config/channel.tx -channelID $CHANNEL_NAME
if [ "$?" -ne 0 ]; then
  echo "Failed to generate channel configuration transaction..."
  exit 1
fi

# generate anchor peer transaction
./bin/configtxgen -profile OneOrgChannel -outputAnchorPeersUpdate ./config/OrgMSPanchors.tx -channelID $CHANNEL_NAME -asOrg OrgMSP
if [ "$?" -ne 0 ]; then
  echo "Failed to generate anchor peer update for OrgMSP..."
  exit 1
fi
```

<br><br>

> binary file 다운로드
```
$) curl https://www.dropbox.com/s/hcbqvy61yy0tccf/bin.tar.gz?dl=1 -O -J -L
```

<br>

> binary 파일 압축풀기
```
$) tar -xvzf bin.tar.gz
```

<br><br>

> compose project env 파일 설정하기
```
$) vi .env
> COMPOSE_PROJECT_NAME=kb
```

<br><br>


> docker-compose.yml 파일 생성
```
#
# Copyright IBM Corp All Rights Reserved
#
# SPDX-License-Identifier: Apache-2.0
#
version: '2'

networks:
  basic:

services:
  ca.kb.com:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.kb.com
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org.kb.com-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/f12d4eeeb67fd1c7a730058d575e7dff3783fd6d6b7e9a5527aac17d3943456a_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start -b admin:adminpw --cfg.identities.allowremove'
    volumes:
      - ./crypto-config/peerOrganizations/org.kb.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.kb.com
    networks:
      - basic

  orderer.kb.com:
    container_name: orderer.kb.com
    image: hyperledger/fabric-orderer
    environment:
      - ORDERER_LOGGING_GRPC=debug
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 7050:7050
    volumes:
        - ./config/:/etc/hyperledger/configtx
        - ./crypto-config/ordererOrganizations/kb.com/orderers/orderer.kb.com/:/etc/hyperledger/msp/orderer
    networks:
      - basic

  peer0.org.kb.com:
    container_name: peer0.org.kb.com
    image: hyperledger/fabric-peer
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_ATTACHSTDOUT=true
      - CORE_PEER_ID=peer0.org.kb.com
      - CORE_LOGGING_PEER=info
      - CORE_CHAINCODE_LOGGING_LEVEL=info
      - CORE_PEER_LOCALMSPID=OrgMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer0.org.kb.com:7051
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    ports:
      - 7051:7051
      - 7053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org.kb.com/peers/peer0.org.kb.com/msp:/etc/hyperledger/msp/peer
        - ./crypto-config/peerOrganizations/org.kb.com/users:/etc/hyperledger/msp/users
        - ./config:/etc/hyperledger/configtx
    depends_on:
      - orderer.kb.com
      - couchdb0
    networks:
      - basic

  couchdb0:
    container_name: couchdb0
    image: hyperledger/fabric-couchdb
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 5984:5984
    networks:
      - basic

  peer1.org.kb.com:
    container_name: peer1.org.kb.com
    image: hyperledger/fabric-peer
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_ATTACHSTDOUT=true
      - CORE_PEER_ID=peer1.org.kb.com
      - CORE_LOGGING_PEER=info
      - CORE_CHAINCODE_LOGGING_LEVEL=info
      - CORE_PEER_LOCALMSPID=OrgMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer1.org.kb.com:7051
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    ports:
      - 8051:7051
      - 8053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org.kb.com/peers/peer1.org.kb.com/msp:/etc/hyperledger/msp/peer
        - ./crypto-config/peerOrganizations/org.kb.com/users:/etc/hyperledger/msp/users
        - ./config:/etc/hyperledger/configtx
    depends_on:
      - orderer.kb.com
      - couchdb1
    networks:
      - basic

  couchdb1:
    container_name: couchdb1
    image: hyperledger/fabric-couchdb
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 6984:5984
    networks:
      - basic


  peer2.org.kb.com:
    container_name: peer2.org.kb.com
    image: hyperledger/fabric-peer
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_ATTACHSTDOUT=true
      - CORE_PEER_ID=peer2.org.kb.com
      - CORE_LOGGING_PEER=info
      - CORE_CHAINCODE_LOGGING_LEVEL=info
      - CORE_PEER_LOCALMSPID=OrgMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer2.org.kb.com:7051
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb2:5984
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    ports:
      - 9051:7051
      - 9053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org.kb.com/peers/peer2.org.kb.com/msp:/etc/hyperledger/msp/peer
        - ./crypto-config/peerOrganizations/org.kb.com/users:/etc/hyperledger/msp/users
        - ./config:/etc/hyperledger/configtx
    depends_on:
      - orderer.kb.com
      - couchdb2
    networks:
      - basic

  couchdb2:
    container_name: couchdb2
    image: hyperledger/fabric-couchdb
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 7984:5984
    networks:
      - basic

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=info
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org.kb.com:7051
      - CORE_PEER_LOCALMSPID=OrgMSP
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org.kb.com/users/Admin@org.kb.com/msp
      - CORE_CHAINCODE_KEEPALIVE=10
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./files/:/opt/gopath/src/
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
    networks:
        - basic

```

#### 3.runFabric.sh

```
set -ev

# delete previous creds
rm -rf ~/.hfc-key-store/*

# copy peer admin credentials into the keyValStore
mkdir -p ~/.hfc-key-store

# don't rewrite paths for Windows Git Bash users
export MSYS_NO_PATHCONV=1

docker-compose -f docker-compose.yml down

docker-compose -f docker-compose.yml up -d ca.kb.com orderer.kb.com peer0.org.kb.com peer1.org.kb.com peer2.org.kb.com couchdb0 couchdb1 couchdb2 cli

```

<br><br>

#### 4.createChannel.sh

```
docker exec \
-e "CORE_PEER_LOCALMSPID=OrgMSP" \
-e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org.kb.com/msp" peer0.org.kb.com peer channel create \
-o orderer.kb.com:7050 \
-c kbchannel \
-f /etc/hyperledger/configtx/channel.tx
```


<br><br>

#### 5.joinChannel.sh
```
# Join peer0.kb.com to the channel.
printf "Join peer0.kb.com to the channel \n"
docker exec \
-e "CORE_PEER_LOCALMSPID=OrgMSP" \
-e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org.kb.com/msp" peer0.org.kb.com peer channel join \
-b kbchannel.block

# Join peer1.kb.com to the channel.
printf "Join peer1.kb.com to the channel \n"
docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org.kb.com/msp" peer1.org.kb.com peer channel fetch config -o orderer.kb.com:7050 -c kbchannel
docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org.kb.com/msp" peer1.org.kb.com peer channel join -b kbchannel_config.block


# Join peer2.kb.com to the channel.
printf "Join peer2.kb.com to the channel \n"
docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org.kb.com/msp" peer2.org.kb.com peer channel fetch config -o orderer.kb.com:7050 -c kbchannel
docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org.kb.com/msp" peer2.org.kb.com peer channel join -b kbchannel_config.block

```

<br><br>
 
 > 체인코드를 배포하기 위한 폴더 만들기
 ```
 $) cd ~/day3/hyperledger/files/github.com
 $) sudo mkdir kb 
 ```

<br><br>

 > kb폴더 밑에 체인코드를 위치시킨다
 
 - chaincode.go
 ```
 /*
 * SPDX-License-Identifier: Apache-2.0
 */

package main

import (
	"encoding/json"
	"fmt"
	"strings"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	sc "github.com/hyperledger/fabric/protos/peer"
)

// Chaincode is the definition of the chaincode structure.
type Chaincode struct {
}

// TestModel is the definition of asset(model) structure.
type TestModel struct {
	Key   string
	Value string
}

// Init is called when the chaincode is instantiated by the blockchain network.
func (cc *Chaincode) Init(stub shim.ChaincodeStubInterface) sc.Response {
	fcn, params := stub.GetFunctionAndParameters()
	fmt.Println("Init()", fcn, params)
	return shim.Success(nil)
}

// Invoke is called as a result of an application request to run the chaincode.
func (cc *Chaincode) Invoke(stub shim.ChaincodeStubInterface) sc.Response {
	fcn, params := stub.GetFunctionAndParameters()

	if fcn == "putData" {
		return cc.putData(stub, params)
	} else if fcn == "getData" {
		return cc.getData(stub, params)
	}
	return shim.Error("Invalid Chaincode function name")
}

// Define example to call submit transaction
func (cc *Chaincode) putData(stub shim.ChaincodeStubInterface, params []string) sc.Response {
	key := strings.TrimSpace(params[0])

	testModel := TestModel{
		Key:   key,
		Value: params[1],
	}

	valueJSONAsBytes, err := json.Marshal(testModel)

	if err != nil {
		return shim.Error(err.Error())
	}

	err = stub.PutState(key, valueJSONAsBytes)

	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

// Define example to call evaluate transaction
func (cc *Chaincode) getData(stub shim.ChaincodeStubInterface, params []string) sc.Response {
	key := strings.TrimSpace(params[0])

	valueJSONAsBytes, err := stub.GetState(key)

	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(valueJSONAsBytes)
}

 ```

 - main.go
```
/*
 * SPDX-License-Identifier: Apache-2.0
 */

package main

import "github.com/hyperledger/fabric/core/chaincode/shim"

func main() {
	err := shim.Start(new(Chaincode))
	if err != nil {
		panic(err)
	}
}

```
<br><br>

#### 6.installChaincode.sh
```
CHAINCODE_NAME="kb"
CHAINCODE_VERSION="1"

#install chaincode peer0
printf "install chaincode peer0 \n"
DOCKER_COMMAND="CORE_PEER_LOCALMSPID='OrgMSP' CORE_PEER_ADDRESS=peer0.org.kb.com:7051 peer chaincode install -n $CHAINCODE_NAME -v $CHAINCODE_VERSION -p github.com/$CHAINCODE_NAME"
docker exec cli /bin/bash -c "$DOCKER_COMMAND"
printf "\n"

#install chaincode peer1
printf "install chaincode peer1 \n"
DOCKER_COMMAND="CORE_PEER_LOCALMSPID='OrgMSP' CORE_PEER_ADDRESS=peer1.org.kb.com:7051 peer chaincode install -n $CHAINCODE_NAME -v $CHAINCODE_VERSION -p github.com/$CHAINCODE_NAME"
docker exec cli /bin/bash -c "$DOCKER_COMMAND"
printf "\n"

#install chaincode peer2
printf "install chaincode peer2 \n"
DOCKER_COMMAND="CORE_PEER_LOCALMSPID='OrgMSP' CORE_PEER_ADDRESS=peer2.org.kb.com:7051 peer chaincode install -n $CHAINCODE_NAME -v $CHAINCODE_VERSION -p github.com/$CHAINCODE_NAME"
docker exec cli /bin/bash -c "$DOCKER_COMMAND"
printf "\n"
```

<br><br>

#### 7.instantiateChaincode.sh
```
CHAINCODE_NAME="kb"

docker exec -e "CORE_PEER_LOCALMSPID=OrgMSP" \
-e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org.kb.com/users/Admin@org.kb.com/msp" \
cli peer chaincode instantiate \
-o orderer.kb.com:7050 \
-C kbchannel \
-n $CHAINCODE_NAME \
-v 1 -c '{"Args":["init","a","100","b","200"]}' 

```

<br><br>

#### 8.invoke.sh

```
CHAINCODE_NAME="kb"

DOCKER_COMMAND="CORE_PEER_LOCALMSPID=\"OrgMSP\" \
CORE_PEER_ADDRESS=peer0.org.kb.com:7051 \
peer chaincode invoke \
-C kbchannel \
-n $CHAINCODE_NAME \
-c '{\"Args\":[\"putData\", \"user1\",\"홍길동\"]}'"
docker exec cli /bin/bash -c "$DOCKER_COMMAND"


```

<br><br>

#### 9.query.sh
```
CHAINCODE_NAME="kb"

#query chaincode peer0
DOCKER_COMMAND="CORE_PEER_LOCALMSPID=\"OrgMSP\" \
CORE_PEER_ADDRESS=peer0.org.kb.com:7051 \
peer chaincode query \
-C kbchannel \
-n $CHAINCODE_NAME \
-c '{\"Args\":[\"getData\", \"user1\"]}'"
docker exec cli /bin/bash -c "$DOCKER_COMMAND"


#query chaincode peer1
DOCKER_COMMAND="CORE_PEER_LOCALMSPID=\"OrgMSP\" \
CORE_PEER_ADDRESS=peer1.org.kb.com:7051 \
peer chaincode query \
-C kbchannel \
-n $CHAINCODE_NAME \
-c '{\"Args\":[\"getData\", \"user1\"]}'"
docker exec cli /bin/bash -c "$DOCKER_COMMAND"


#query chaincode peer2
DOCKER_COMMAND="CORE_PEER_LOCALMSPID=\"OrgMSP\" \
CORE_PEER_ADDRESS=peer2.org.kb.com:7051 \
peer chaincode query \
-C kbchannel \
-n $CHAINCODE_NAME \
-c '{\"Args\":[\"getData\", \"user1\"]}'"
docker exec cli /bin/bash -c "$DOCKER_COMMAND"

```

<br><br>

#### 10.couchDB 확인하기
```
http://192.168.56.10:5984/_utils
http://192.168.56.10:6984/_utils
http://192.168.56.10:7984/_utils
```
# Fabric

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

#### (필수) crypto-config.yaml

```
OrdererOrgs:
  - Name: Orderer
    Domain: kb.com
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
Organizations:
    - &OrdererOrg
        Name: OrdererOrg

        ID: OrdererMSP

        MSPDir: crypto-config/ordererOrganizations/kb.com/msp

    - &Org
        Name: OrgMSP

        ID: OrgMSP

        MSPDir: crypto-config/peerOrganizations/org.kb.com/msp

        AnchorPeers:
            - Host: peer0.org.kb.com
              Port: 7051

Application: &ApplicationDefaults
    Organizations:

Orderer: &OrdererDefaults
    OrdererType: solo

    Addresses:
        - orderer.kb.com:7050

    BatchTimeout: 1s

    BatchSize:
        MaxMessageCount: 500
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB

    Kafka:
        - 127.0.0.1:9092

    Organizations:
        OneOrgOrdererGenesis:
            Orderer:
                <<: *OrdererDefaults
                Organizations:
                    - *OrdererOrg
            Consortiums:
                kbConsortium:
                    Organizations:
                        - *Org
        OneOrgChannel:
            Consortium: kbConsortium
            Application:
                <<: *ApplicationDefaults
                Organizations:
                    - *Org
```

#### 2. generate_artifact.sh
```
#!/bin/sh

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
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

> bin 파일 다운로드
```
$) https://drive.google.com/open?id=17MTYwIGzbyvapxqdUhAi6mHFTZ_tYLL8
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
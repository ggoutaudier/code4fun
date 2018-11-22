# Welcome!
Welcome to the Code4Fun Hyperledger Meetup in Geneva.
This page contains the script that we will follow during the workshop.

# Prerequisites
In this workshop we will play with the latest version of Hyperledger Fabric, currently version 1.3.

Please verify that you have all the prerequisites listed on this page:
https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html

You can safely skip the node.js configuration as we will provide a docker image containing the client application.


# Hyperleder network setup
## Download all you need
First, choose your working directory and clone the code4fun Git repository:
```
git clone https://github.com/ggoutaudier/code4fun
```

For the rest of the workshop we will work from the code4fun directory:
```
cd code4fun
```


We need to download the Hyperledger 1.3 docker images and binaries. Hyperledger provides a simple script to do that:
```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0 1.3.0 0.4.13
```


## Generate the certificates
We will first generate the certificates used by the different entities composing the Hyperledger network.
To do that, we will use the `cryptogen` tool. 

First review the `crypto-config.yaml` file:
```
# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes
# ---------------------------------------------------------------------------
OrdererOrgs:
  - Name: Orderer
    Domain: code4fun.com
    Specs:
      - Hostname: orderer

# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ---------------------------------------------------------------------------
PeerOrgs:
  - Name: Geneva
    Domain: geneva.code4fun.com
    EnableNodeOUs: true
    Specs:
    - Hostname: peer1
    - Hostname: peer2
    Users:
      Count: 1

  - Name: Zurich
    Domain: zurich.code4fun.com
    EnableNodeOUs: true
    Specs:
    - Hostname: peer1
    - Hostname: peer2
    Users:
      Count: 1
```

Now generate the certificates:

`./fabric-samples/bin/cryptogen generate --config=./crypto-config.yaml`

A new `crypto-config` directory has been created. Have a look at its structure and the various files that have been generated.

## Generate the first transactions
Now it is time to configure the Blockchain network. This will be done by submitting transactions to the network.
We need to generate 3 transactions:
- the orderer genesis block,
- the channel configuration transaction,
- and two anchor peer transactions - one for each Peer Org.

We will generate all these transactions using the `configtxgen` command and the `configtx.yaml` configuration file:
```
copy file content here
```

First things first, let's create the Orderer genesis block (which can be understood as the initial setup of the system channel):

```
export FABRIC_CFG_PATH=$PWD
mkdir channel-artifacts
./fabric-samples/bin/configtxgen -profile TwoOrgsOrdererGenesis -channelID code4fun-sys-channel -outputBlock ./channel-artifacts/genesis.block
```

Now we can create the channel configuration:
```
./fabric-samples/bin/configtxgen -profile TwoOrgsChannel -channelID code4fun-sys-channel -outputCreateChannelTx ./channel-artifacts/channel.tx 
```

Finally we can create the transactions defining the anchor peers:
```
./fabric-samples/bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/GenevaMSPanchors.tx -channelID swiss-channel -asOrg GenevaMSP
./fabric-samples/bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/ZurichMSPanchors.tx -channelID swiss-channel -asOrg ZurichMSP
```

## Start the network
All the containers composing the blockchain network (orderers, peers, and cli) are described in a docker-compose yaml file.
Take a couple of minutes to have a look at the `docker-compose-cli.yaml` and `base/*` files that are provided.

In particular, double check that the mounted volumes correspond to the crypto-config folder that we previously created (remember, this is the folder that contains all the certificates that will be used by the different components to identify themselves on the Blockchain network).

Now open a new terminal and start the network:
```
docker-compose -f docker-compose-cli.yaml up
```

## Create the channel
peer channel create -o orderer.code4fun.com:7050 -c swiss-channel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/code4fun.com/orderers/orderer.code4fun.com/msp/tlscacerts/tlsca.code4fun.com-cert.pem



# Chaincode installation
blahblah

# Optional - To go further
blahblah




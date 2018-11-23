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

Review the `crypto-config.yaml` file and generate the certificates:
`./fabric-samples/bin/cryptogen generate --config=./crypto-config.yaml`

A new `crypto-config` directory has been created. Have a look at its structure and the various files that have been generated.

## Generate the first transactions
Now it is time to configure the Blockchain network. This will be done by submitting transactions to the network.
We need to generate 3 transactions:
- the orderer genesis block,
- the channel configuration transaction,
- and two anchor peer transactions - one for each Peer Org.

We will generate all these transactions using the `configtxgen` command and the `configtx.yaml` configuration file.

First things first, let's create the Orderer genesis block (which can be understood as the initial setup of the system channel):

```
export FABRIC_CFG_PATH=$PWD
mkdir channel-artifacts
./fabric-samples/bin/configtxgen -profile TwoOrgsOrdererGenesis -channelID code4fun-sys-channel -outputBlock ./channel-artifacts/genesis.block
```

Now we can create the channel configuration:
```
./fabric-samples/bin/configtxgen -profile TwoOrgsChannel -channelID swiss-channel -outputCreateChannelTx ./channel-artifacts/channel.tx 
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

Now start the network:
```
docker-compose -f docker-compose-cli.yaml up
```
Leave your terminal open to watch the logs and open a new one to continue. Make sure you work from the `code4fun` directory.

## Create the channel
We will now connect to the CLI container and create a channel using the `channel.tx` file that we created previously:
```
docker exec -it cli bash
peer channel create -o orderer.code4fun.com:7050 -c swiss-channel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/code4fun.com/orderers/orderer.code4fun.com/msp/tlscacerts/tlsca.code4fun.com-cert.pem
```
You may have noticed that the docker-compose files contained some environment variables pointing to the `peer1.geneva.code4fun.com` peer. The command that we just executed was therefore processed through this peer.


## Add peer to the channel
From the CLI container run the following command to join a peer to the channel:
```
peer channel join -b swiss-channel.block
```
Remember that we are sending the commands through `peer1.geneva.code4fun.com`. It is therefore this peer that will be joined to the channel. If you look at the logs in the other terminal, you should see several lines from the peer1.geneva.code4fun.com container illustrating this joining process.

Now let's update the relevant environment variables and ask peer1.zurich.code4fun.com to join the network as well:
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/zurich.code4fun.com/users/Admin@zurich.code4fun.com/msp 
CORE_PEER_ADDRESS=peer1.zurich.code4fun.com:7051 
CORE_PEER_LOCALMSPID="ZurichMSP" 
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/zurich.code4fun.com/peers/peer1.zurich.code4fun.com/tls/ca.crt 
peer channel join -b swiss-channel.block
```

# Chaincode installation
blahblah

# Optional - To go further
blahblah


### Troubleshooting
If you notice issues while starting the network, use this command to properly shut down all the containers:
```

docker-compose -f docker-compose-cli.yaml down --volumes --remove-orphans
```


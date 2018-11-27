# Welcome!
Welcome to the Code4Fun Hyperledger Meetup in Geneva.
This page contains the script that we will follow during the workshop.

# Prerequisites
## Install the Hyperledger prerequisites
In this workshop we will play with the latest version of Hyperledger Fabric, currently version 1.3.

Please verify that you have all the prerequisites listed on this page:
https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html

You can safely skip the node.js configuration as we will provide a docker image containing the client application.

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

# Hyperleder network setup

## Generate the certificates
We will first generate the certificates used by the different entities composing the Hyperledger network.
To do that, we will use the `cryptogen` tool. 

Review the `crypto-config.yaml` file and generate the certificates:
```
./fabric-samples/bin/cryptogen generate --config=./crypto-config.yaml
```

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


## Add peers to the channel
In this section we will send commands to the different peers from the CLI. To target a specifc peer, the following environment variables need to be set (example for peer1.zurich.code4fun.com):
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/zurich.code4fun.com/users/Admin@zurich.code4fun.com/msp 
CORE_PEER_ADDRESS=peer1.zurich.code4fun.com:7051 
CORE_PEER_LOCALMSPID="ZurichMSP" 
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/zurich.code4fun.com/peers/peer1.zurich.code4fun.com/tls/ca.crt 
```
We created a few helper scripts to change these environment variables for each peer in the network: see `setenv_gva_peer1.sh`, `setenv_gva_peer2.sh`, `setenv_zh_peer1.sh`, and `setenv_zh_peer1.sh` in the `./scripts` directory. We will use these scripts instead of manually setting the environment variables.

Run the following command to join all the peers to the network:
```
. scripts/setenv_gva_peer1.sh
peer channel join -b swiss-channel.block
. scripts/setenv_gva_peer2.sh
peer channel join -b swiss-channel.block
. scripts/setenv_zh_peer1.sh
peer channel join -b swiss-channel.block
. scripts/setenv_zh_peer2.sh
peer channel join -b swiss-channel.block
```

If you look at the logs in the other terminal, you should see several message from each peer1.geneva.code4fun.com while joining the network.


## Definition of the anchor peers
Enter the following commands to set `peer1.geneva.code4fun.com` and `peer1.zurich.code4fun.com` as anchor peers for the Geneva and Zurich organisations respectively:
```
. scripts/setenv_gva_peer1.sh
peer channel update -o orderer.code4fun.com:7050 -c swiss-channel -f ./channel-artifacts/GenevaMSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/code4fun.com/orderers/orderer.code4fun.com/msp/tlscacerts/tlsca.code4fun.com-cert.pem
. scripts/setenv_zh_peer1.sh
peer channel update -o orderer.code4fun.com:7050 -c swiss-channel -f ./channel-artifacts/ZurichMSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/code4fun.com/orderers/orderer.code4fun.com/msp/tlscacerts/tlsca.code4fun.com-cert.pem
```


# Chaincode installation
Let's install the chaincode on peer1.geneva.code4fun.com:
```
. scripts/setenv_gva_peer1.sh
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```
Note that the -p parameter corresponds to the `$GOPATH/src`, i.e. `/opt/gopath/src`, and that `./chaincode` is mounted on `/opt/gopath/src/github.com/chaincode` in the CLI container, therefore the given path.

Now we can instantiate the chaincode on the channel:
```
peer chaincode instantiate -o orderer.code4fun.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/code4fun.com/orderers/orderer.code4fun.com/msp/tlscacerts/tlsca.code4fun.com-cert.pem -C swiss-channel -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('GenevaMSP.peer','ZurichMSP.peer')"
```
Note that `-P "AND ('GenevaMSP.peer','ZurichMSP.peer')"` argument. It means that transactions must be endorsed by a peer from the Geneva organisation AND a peer from the Zurich organisation. 

# Chaincode queries
## Read
```
peer chaincode query -C swiss-channel -n mycc -c '{"Args":["query","a"]}'
```
## Write
Before we can write to the chaincode, we need to deploy it on at least one peer from the Zurich organization (remember that per the policy that we defined, transactions have to be endorsed by both organisations to be valid):

```
. scripts/setenv_zh_peer1.sh
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```
Note that the chaincode has already been instantiated at the channel level, so there is no need to instantiate it again on the zurich peer (the code will be built automatically).

Now we can invoke the chaincode:
```
peer chaincode invoke -o orderer.code4fun.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/code4fun.com/orderers/orderer.code4fun.com/msp/tlscacerts/tlsca.code4fun.com-cert.pem -C swiss-channel -n mycc --peerAddresses peer1.geneva.code4fun.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/geneva.code4fun.com/peers/peer1.geneva.code4fun.com/tls/ca.crt --peerAddresses peer1.zurich.code4fun.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/zurich.code4fun.com/peers/peer1.zurich.code4fun.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'

```
Have a look at the logs: you should see that a new block has been committed by the 4 peers (not only the peers that are running the chaincode). You can also check that a value of "10" has been transferred from "a" to "b":
```
peer chaincode query -C swiss-channel -n mycc -c '{"Args":["query","a"]}'
peer chaincode query -C swiss-channel -n mycc -c '{"Args":["query","b"]}'
```

# Troubleshooting
If you notice issues while starting the network, use this command to properly shut down all the containers:
```

docker-compose -f docker-compose-cli.yaml down --volumes --remove-orphans
```
If you also need to regenerate the certificates and the channel artifacts:
```
rm -rf crypto-config
rm channel-artifacts/*
```



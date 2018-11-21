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

`git clone https://github.com/ggoutaudier/code4fun`

We also need to download the Hyperledger 1.3 docker images and binaries. Hyperledger provides a simple script to do that:

`curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0 1.3.0 0.4.13`

## Generate the certificates
We will first generate the certificates used by the different entities composing the Hyperledger network.
To do that, we will use the `cryptogen` tool. 

First review the ./code4fun/crypto-config.yaml file:
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

`./fabric-samples/bin/cryptogen generate --config=./code4fun/crypto-config.yaml`

A new `crypto-config` directory has been created. Have a look at its structure and the various files that have been generated.

## Generate the first transactions
Now it is time to configure the Blockchain network. This will be done by submitting transactions to the network.
We need to generate 3 transactions:
- the orderer genesis block,
- the channel configuration transaction,
- and two anchor peer transactions - one for each Peer Org.

We will generate all these transactions using the `configtxgen` command and the `./code4fun/configtx.yaml` configuration file:
```
copy file content here
```

First things first, let's create the Orderer genesis block (which can be understood as the initial setup of the system channel):

```
./fabric-samples/bin/configtxgen -configPath ./code4fun/configtx.yaml -profile TwoOrgsOrdererGenesis -channelID code4fun-sys-channel -outputBlock ./channel-artifacts/genesis.block
```





# Chaincode installation
blahblah

# Optional - To go further
blahblah




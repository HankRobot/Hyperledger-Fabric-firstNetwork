## Hyperledger Fabric Network Design
Navigate to 'first-network' directory and run the following commands:

#### Generate Crypto Material for Network Entities
```sh
../bin/cryptogen generate --config=./crypto-config.yaml
```
#### Generate Channel Configuration
```sh
export FABRIC_CFG_PATH=$PWD
../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
export CHANNEL_NAME=mychannel  && ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg NTUMSP
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg NUSMSP
```
#### Start the Network
```sh
docker-compose -f docker-compose-cli.yaml up -d
```
#### Setting Environment Variables
```sh
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/ntu.singihl.com/users/Admin@ntu.singihl.com/msp
CORE_PEER_ADDRESS=peer0.ntu.singihl.com:7051
CORE_PEER_LOCALMSPID="NTUMSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/ntu.singihl.com/peers/peer0.ntu.singihl.com/tls/ca.crt
```
#### Create and Join Channel
```sh
docker exec -it cli bash
export CHANNEL_NAME=mychannel
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/ntu.singihl.com/users/Admin@ntu.singihl.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/ntu.singihl.com/peers/peer0.ntu.singihl.com/tls/ca.crt
peer channel create -o orderer.singihl.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/singihl.com/orderers/orderer.singihl.com/msp/tlscacerts/tlsca.singihl.com-cert.pem
```
#### Chaincode Installation, Instantiation and Query
```sh
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
peer channel join -b mychannel.block
peer chaincode instantiate -o orderer.singihl.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/singihl.com/orderers/orderer.singihl.com/msp/tlscacerts/tlsca.singihl.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('NTUMSP.peer','NUSMSP.peer')"
sleep 10
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

```
# Hyperledger Fabric network in K8s with External Chaincodes as pods

Refer the tutorial and instructions in this article: https://medium.com/@pau.aragones/how-to-implement-hyperledger-fabric-external-chaincodes-within-a-kubernetes-cluster-fd01d7544523

## Install the binaries

```sh
wget https://github.com/hyperledger/fabric/releases/download/v2.2.0/hyperledger-fabric-linux-amd64-2.2.0.tar.gz

tar -xzf hyperledger-fabric-linux-amd64-2.2.0.tar.gz

# Move to the bin path
mv bin/* /bin

# Check that you have successfully installed the tools by executing
configtxgen --version

# Should print the following output:
# configtxgen:
#  Version: 2.2.0
#  Commit SHA: 5ea85bc54
#  Go version: go1.14.4
#  OS/Arch: linux/amd64
```

## Launching the network

```sh
./fabricOps.sh start
```

Create the namespace and launch the workload:

```
kubectl create ns hyperledger
kubectl create -f orderer-service/
kubectl create -f org1/
kubectl create -f org2/
```

## Configure the network

```
peer channel create -o orderer0:7050 -c mychannel -f ./scripts/channel-artifacts/channel.tx --tls true --cafile $ORDERER_CA
peer channel join -b mychannel.block
peer channel fetch 0 mychannel.block -c mychannel -o orderer0:7050 --tls --cafile $ORDERER_CA
```

## Package the chaincode information for the peer and install it

```
cd chaincode/packaging
tar cfz code.tar.gz connection.json
tar cfz marbles-org1.tgz code.tar.gz metadata.json
peer lifecycle chaincode install marbles-org1.tgz
```

## Build and deploy the chaincode

```
docker build -t chaincode/marbles:1.0 .
kubectl create -f chaincode/k8s
```

## Approve the chaincode and commit it to the channel

```
peer lifecycle chaincode approveformyorg --channelID mychannel --name marbles --version 1.0 --init-required --package-id marbles:d8140fbc1a0903bd88611a96c5b0077a2fdeef00a95c05bfe52e207f5f9ab79d --sequence 1 -o orderer0:7050 --tls --cafile $ORDERER_CA

peer lifecycle chaincode approveformyorg --channelID mychannel --name marbles --version 1.0 --init-required --package-id marbles:af45e0faa9676dd884a56e34b6a9bc1f7f1df04d6356aa1b2b9f123bd1d9e9e6 --sequence 1 -o orderer0:7050 --tls --cafile $ORDERER_CA

peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name marbles --version 1.0 --init-required --sequence 1 -o -orderer0:7050 --tls --cafile $ORDERER_CA

peer lifecycle chaincode commit -o orderer0:7050 --channelID mychannel --name marbles --version 1.0 --sequence 1 --init-required --tls true --cafile $ORDERER_CA --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt
```

## Invoke and query the chaincode

```
peer chaincode invoke -o orderer0:7050 --tls true --cafile $ORDERER_CA -C mychannel -n marbles --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt -c '{"Args":["initMarble","marble1","blue","35","tom"]}' --waitForEvent

peer chaincode invoke -o orderer0:7050 --tls true --cafile $ORDERER_CA -C mychannel -n marbles --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt -c '{"Args":["initMarble","marble2","red","50","tom"]}' --waitForEvent

peer chaincode query -C mychannel -n marbles -c '{"Args":["readMarble","marble1"]}'
```

## Commands for ContractApi based External Chaincode

```
peer lifecycle chaincode approveformyorg --channelID mychannel --name fabcar --version 1.0 --package-id fabcar:005c35f4f172c056723eca09d41e8048e0beaa2712d920c19af837640df7e2aa --sequence 1 -o orderer0:7050 --tls --cafile $ORDERER_CA


peer lifecycle chaincode approveformyorg --channelID mychannel --name fabcar --version 1.0 --package-id fabcar:61ab817a6ad76098d340952e5d8e928d9c5ddff1a53341dc8d0c64b4345564b0 --sequence 1 -o orderer0:7050 --tls --cafile $ORDERER_CA


peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name fabcar --version 1.0 --sequence 1 -o -orderer0:7050 --tls --cafile $ORDERER_CA

peer lifecycle chaincode commit -o orderer0:7050 --channelID mychannel --name fabcar --version 1.0 --sequence 1 --tls true --cafile $ORDERER_CA --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt


peer chaincode invoke -o orderer0:7050 --tls true --cafile $ORDERER_CA -C mychannel -n fabcar --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt -c '{"Args":["InitLedger"]}' --waitForEvent

peer chaincode query -C mychannel -n fabcar -c '{"Args":["QueryAllCars"]}' 
```
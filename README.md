# Hyperledger Fabric network in K8s with External Chaincodes as pods

## Install the binaries

```sh
wget https://github.com/hyperledger/fabric/releases/download/v2.0.1/hyperledger-fabric-linux-amd64-2.0.1.tar.gz

tar -xzf hyperledger-fabric-linux-amd64-2.0.1.tar.gz

# Move to the bin path
mv bin/* /bin

# Check that you have successfully installed the tools by executing
configtxgen --version

# Should print the following output:
# configtxgen:
#  Version: 2.0.1
#  Commit SHA: 1cfa5da98
#  Go version: go1.13.4
#  OS/Arch: linux/amd64
```

## Launching the network

```sh
./fabricOps.sh start
```

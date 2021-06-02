# study-fabric
学习fabric的坎坷之路

# 搭建操作的全部命令

```shell
# 1、生成证书文件
cryptogen generate --config=crypto-config.yaml

# 2、创世区块和通道文件的生成
configtxgen -channelID sc -profile ItcastOrgsOrdererGenesis -outputBlock ./genesis.block
configtxgen -channelID mc -profile ItcastOrgsChannel -outputCreateChannelTx channel.tx

# 3、可能需要用到的一步：删除所有之前无用的镜像挂载，不删除则会导致镜像挂载卷失败
docker volume rm $(docker volume ls -q)

# 4、最好检查docker是否之前关闭
docker-compose ps 若没有则docker-compose down

# 5、启动镜像（-d可以省略，则直接在终端可以监测各个镜像的信息）
docker-compose up -d

# 6、创建通道（需要先进入docker容器中的cli终端）
docker exec -it cli /bin/bash 或者 通过vscode中docker插件直接attach shell

 peer channel create -o orderer.itcast.com:7050 -c mc -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/itcast.com/msp/tlscacerts/tlsca.itcast.com-cert.pem
	# 6.1、检查在容器中是否成功创建相应的文件（需在容器中检查）
	ls
```

```shell
# 7、将各个节点加入通道（以下全是）
# OrgGo组织的peer0.orggo.itcast.com
export CORE_PEER_ADDRESS=peer0.orggo.itcast.com:7051
export CORE_PEER_LOCALMSPID=OrgGoMSP  # 组织ID
export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/users/Admin@orggo.itcast.com/msp
 peer channel join -b mc.block
# OrgGo组织的peer1.orggo.itcast.com
export CORE_PEER_ADDRESS=peer1.orggo.itcast.com:8051
export CORE_PEER_LOCALMSPID=OrgGoMSP  # 组织ID
export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/users/Admin@orggo.itcast.com/msp
peer channel join -b mc.block
# OrgCpp组织的peer0.orgcpp.itcast.com
export CORE_PEER_ADDRESS=peer0.orgcpp.itcast.com:9051
export CORE_PEER_LOCALMSPID=OrgCppMSP  # 组织ID
export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer0.orgcpp.itcast.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer0.orgcpp.itcast.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer0.orgcpp.itcast.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/users/Admin@orgcpp.itcast.com/msp
peer channel join -b mc.block
# OrgCpp组织的peer1.orgcpp.itcast.com
export CORE_PEER_ADDRESS=peer1.orgcpp.itcast.com:10051
export CORE_PEER_LOCALMSPID=OrgCppMSP  # 组织ID
export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/users/Admin@orgcpp.itcast.com/msp
peer channel join -b mc.block
```

```shell
# 8、如上操作完成后可以检验当前cli连接到哪个节点
echo $CORE_PEER_ADDRESS
```



** <font color="red"> 如下切记所有路径的正确性，相应的及时更改 ! ! !  </font> **

```shell
# 9、链码安装（需要进入到cli容器中），每个组织安装一次（最好每个节点安装一次）
peer chaincode install -n testcc -v 1.0 -p github.com/chaincode/
	# 9.1、用上面的环境变量切换cli连接的节点（即换个组织或者依次换所有节点）。若之前是go组织，如下则换成cpp
export CORE_PEER_ADDRESS=peer1.orgcpp.itcast.com:10051
export CORE_PEER_LOCALMSPID=OrgCppMSP  # 组织ID
export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/users/Admin@orgcpp.itcast.com/msp
peer chaincode install -n medical -v 1.0 -p github.com/chaincode/mygo/


# 10、链码实例化（即调用链码中初始化函数init）
peer chaincode instantiate -o orderer.itcast.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/itcast.com/msp/tlscacerts/tlsca.itcast.com-cert.pem -C mc -n testcc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('OrgGoMSP.member', 'OrgCppMSP.member')"
	# 10.1、查询通道中安装的链码和实例化的链码（可做可不做，只为检验是否确定成功安装和实例化）
peer chaincode list -C mc --installed
peer chaincode list -C mc --instantiated


# 11、链码调用，包括上传信息、获取记录等等
#（切记tls通信加密参数，一般只需要更改后三个参数，若通道名和合约名不变，则只需要更改最后一个参数）
peer chaincode invoke -o orderer.itcast.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/itcast.com/orderers/orderer.itcast.com/msp/tlscacerts/tlsca.itcast.com-cert.pem --peerAddresses peer0.orggo.itcast.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/ca.crt --peerAddresses peer1.orgcpp.itcast.com:10051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/ca.crt -C mc -n medical -c '{"Args":["testInput","aaa","asd"]}'
```






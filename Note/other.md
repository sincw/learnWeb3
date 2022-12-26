# 杂项

学习过程中记录查询到的资料

##### 安装geth客户端

https://www.jianshu.com/p/c66b74d9dc35

```json
{
"config": {
"chainId": 66,  //该链的ID。在用geth 启动区块链时，还需要指定一个network 参数。只有当network、chainID、创世区块配置都相同时，才是同一条链。
"homesteadBlock": 0, //相关协议机制的升级区块所在的高度，签名算法是homestead ->eip155 -> eip158，所以从homesteadBlock 之前区块都通过homestead 相关算法机制来验证，homesteadBlock 到eip155Block 之间的用eip155 算法来验证，依次类推。
"eip150Block": 0,
"eip155Block": 0,
"eip158Block": 0
},
"coinbase" : "0x0000000000000000000000000000000000000000",  //每挖出一个区块，都会获得奖励。该值指定默认情况下把奖励给到哪个账户。实际上，我们每次挖矿开始之前，都会自己指定miner.setEtherbase(UserAddress)，一般都会把奖励给自己


"difficulty" : "0x2000",  //定义了每次挖矿时，最终确定nonce 的难度
"extraData" : "",
"gasLimit" : "0xffffffff",  //规定该区块链中，gas 的上限
"nonce" : "0x0000000000000042",  //预定一个随机数，这是一个与PoW 机制有关的值
"mixhash" : "0x0000000000000000000000000000000000000000000000000000000000000000",  //一个与PoW 机制有关的值
"parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000", //在区块链中，区块是相连的，parentHash 指定了本区块的上一个区块Hash。对于创世区块来说，parentHash 为0
"timestamp" : "0x00",  // 时间戳，规定创世区块开始的时间
"alloc": { }   //代表初始资产配置，在该区块链产生时，就预先赋予这些账户一定数额的WEI（不是ETH）
}
```

```bash
geth --datadir . init genesis.json
geth --datadir . --networkid 66 --http console 2>>geth.log --nodiscover --dev --dev.period 1 
```

```bash
eth.accounts//查看账户

eth.getBalance(eth.accounts[0])//获取余额
miner.start(1);admin.sleepBlocks(1);miner.stop();//开始挖矿，挖到后停止
```

```bash
geth --datadir . --networkid 66 --http --http.addr 192.168.88.131 --http.vhosts "*" --http.port 8545 --http.api 'db,net,eth,web3,personal' --http.corsdomain "*" console 2>>geth.log --nodiscover --dev --dev.period 1 
personal.newAccount("your password")

web3.fromWei(eth.getBalance(eth.accounts[0]))
eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:web3.toWei(1,"ether")});
curl -X POST -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":74}' 192.168.88.131:8545
```

##### geth常用命令

http://t.zoukankan.com/kaifayuan-p-14970409.html

##### linux安装s5全局代理工具proxychains4

[proxychains 安装和proxychains 代理nmap_【码猿】的博客-CSDN博客_proxychains](https://blog.csdn.net/qq_53086690/article/details/121779832)

##### POAP

Proof of Attendance Protocol = 出勤证明协议

使用ERC721实现的链上足迹

# Web3JS基础

### 概述

开发以太坊区块链应用程序，涉及到以下部分:

- **智能合约开发** – 使用solidity语言编写代码，部署到区块链。
- **网站或客户端开发** – 与区块链中的智能合约进行交互，读写数据。

在进行网站或客户端开发时，就需要用到[web3.js](https://github.com/ethereum/web3.js/)。web3.js库是一个javascript库

基于eth的RPC调用。

### 基础使用

##### 创建web3对象

```js
const Web3 = require('web3')
const rpcURL = 'http://localhost:8545' // RPC URL
const web3 = new Web3(rpcURL)
const address = '' // 账户地址
// 读取address中的余额，余额单位是wei
web3.eth.getBalance(address, (err, wei) => {
    if(!error){
       // 余额单位从wei转换为ether
    balance = web3.utils.fromWei(wei, 'ether')
    console.log("balance: " + balance)
    } else {
    console.error(err)
    }

})
```

##### 创建合约对象

```js
const abi = [{"constant":true,"inputs":[],"name":"mintingFinished","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transferFrom","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"unpause","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"nam e":"_amount","type":"uint256"}],"name":"mint","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"paused","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"finishMinting","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"pause","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_amount","type":"uint256"},{"name":"_releaseTime","type":"uint256"}],"name":"mintTimelocked","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"remaining","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"newOwner","type":"address"}],"name":"transferOwnership","outputs":[],"payable":false,"type":"function"},{"anonymous":false,"inputs":[{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Mint","type":"event"},{"anonymous":false,"inputs":[],"name":"MintFinished","type":"event"},{"anonymous":false,"inputs":[],"name":"Pause","type":"event"},{"anonymous":false,"inputs":[],"name":"Unpause","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"owner","type":"address"},{"indexed":true,"name":"spender","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"}]
const address = "0xd26114cd6EE289AccF82350c8d8487fedB8A0C07"
const contract = new web3.eth.Contract(abi, address)

contract.methods.totalSupply().call((err, result) => { console.log(result) })
// > 1402453982451327807892396
contract.methods.name().call((err, result) => { console.log(result) })
// > OMGToken 
contract.methods.symbol().call((err, result) => { console.log(result) })
// > OMG 
contract.methods.balanceOf('0xd26114cd6EE289AccF82350c8d8487fedB8A0C07').call((err, result) => { console.log(result) })
// > 22620364140612904305860
```

##### PromiEvent

跟promise类似

```js
web3.eth.sendTransaction({from: '0x123...', data: '0x432...'}) 
.once('transactionHash', function(hash){ ... }) 
.once('receipt', function(receipt){ ... }) 
.on('confirmation', function(confNumber, receipt){ ... }) 
.on('error', function(error){ ... }) 
.then(function(receipt){ 
// will be fired once the receipt is mined
 });
```

##### 应用二进制接口ABI

web3.js 通过以太坊[智能合约](https://so.csdn.net/so/search?q=%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6&spm=1001.2101.3001.7020)的 json 接口（Application Binary Interface，ABI）创建一个 JavaScript 对象。abi对象在编译sol后会生成。

```js
//batch用于同步执行
const abi = [{"constant":true,"inputs":[],"name":"mintingFinished","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transferFrom","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"unpause","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"nam e":"_amount","type":"uint256"}],"name":"mint","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"paused","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"finishMinting","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"pause","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_amount","type":"uint256"},{"name":"_releaseTime","type":"uint256"}],"name":"mintTimelocked","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"remaining","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"newOwner","type":"address"}],"name":"transferOwnership","outputs":[],"payable":false,"type":"function"},{"anonymous":false,"inputs":[{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Mint","type":"event"},{"anonymous":false,"inputs":[],"name":"MintFinished","type":"event"},{"anonymous":false,"inputs":[],"name":"Pause","type":"event"},{"anonymous":false,"inputs":[],"name":"Unpause","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"owner","type":"address"},{"indexed":true,"name":"spender","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"}]
const address = "0xd26114cd6EE289AccF82350c8d8487fedB8A0C07"
web3.eth.contract(abi).at(address).balance.request(a
ddress, callback2); 
```

##### 批量执行

保证代码同步执行

```js
const address = "0xd26114cd6EE289AccF82350c8d8487fedB8A0C07"var batch = web3.createBatch(); 
batch.add(web3.eth.getBalance.request('0x0000000000000000
000000000000000000000000', 'latest', callback)); 
batch.add(web3.eth.contract(abi).at(address).balance.request(a
ddress, callback2)); 
batch.execute();
```

##### 大数处理

```js
var balance = new BigNumber('1111111111111111111111111111111111111111');
balance.plus(21).toString(10);//对小数保留20位浮点精度
```

##### 常用API-基本信息查询

**获取 network id**
• 同步：web3.version.network
• 异步：web3.version.getNetwork((err, res)=>{console.log(res)})
• v1.0.0：web3.eth.net.getId().then(console.log)
**获取节点的以太坊协议版本**
• 同步：web3.version.ethereum
• 异步：web3.version.getEthereum((err, res)=>{console.log(res)})
• v1.0.0：web3.eth.getProtocolVersion().then(console.log)
**是否有节点连接/监听，返回true/false**
• 同步：web3.isConnect() 或者 web3.net.listening
• 异步：web3.net.getListening((err,res)=>console.log(res))
• v1.0.0：web3.eth.net.isListening().then(console.log)

**查看当前连接的 peer 节点**
• 同步：web3.net.peerCount
• 异步：web3.net.getPeerCount((err,res)=>console.log(res))
• v1.0.0：web3.eth.net.getPeerCount().then(console.log)
**设置 provider**

• web3.setProvider(provider)  
• web3.setProvider(new web3.providers.HttpProvider(‘http://localhost:8545’))

##### 常用API-通用工具方法

**以太单位转换**
• web3.fromWei web3.toWei

**数据类型转换**
• web3.toString web3.toDecimal web3.toBigNumber

**字符编码转换**
• web3.toHex web3.toAscii web3.toUtf8 web3.fromUtf8

**地址相关**
• web3.isAddress web3.toChecksumAddress
**coinbase 查询**
• 同步：web3.eth.coinbase
• 异步：web3.eth.getCoinbase( (err, res)=>console.log(res) )
• v1.0.0：web3.eth.getCoinbase().then(console.log)

**账户查询**
• 同步：web3.eth.accounts
• 异步：web3.eth.getAccounts( (err, res)=>console.log(res) )
• v1.0.0：web3.eth.getAccounts().then(console.log)
**区块高度查询**
• 同步：web3.eth. blockNumber
• 异步：web3.eth.getBlockNumber( callback )

**gasPrice 查询**
• 同步：web3.eth.gasPrice
• 异步：web3.eth.getGasPrice( callback )
**区块查询**

```js
web3.eth.getBlockNumber( hashStringOrBlockumber)
web3.eth.getBlockNumber( hashStringOrBlockNumber, callback)
web3.eth.getBlockTransactionCount( hashStringOrBlockNumber)
web3.eth.getBlockTransactionCount( hashStringOrBlockNumber)
, callback )
```

##### 常用api-交易相关

**余额查询**

- 同步：web3.eth.getBalance(addressHexString [, defaultBlock])
- 异步：web3.eth.getBalance(addressHexString [, defaultBlock ] , callback])

**交易查询**

- 同步：web3.eth.getTransaction(transactionHash)
- 异步：web3.eth.getTransaction(transactionHash [, callback])

**交易收据查询（已进块）**

- 同步：web3.eth.getTransactionReceipt(hashString)
- 异步：web3.eth.getTransactionReceipt(hashString [,

**callback])**

- 估计 gas 消耗量
- 同步：web3.eth.estimateGas(callObject)
- 异步：web3.eth.estimateGas(callObject [, callback])

**发送交易**

- web3.eth.sendTransaction(transactionObject [, callback])

**交易对象：**

- from：发送地址

- to：接收地址，如果是创建合约交易，可不填

- value：交易金额，以wei为单位，可选

- gas：交易消耗 gas 上限，可选

- gasPrice：交易 gas 单价，可选

- data：交易携带的字串数据，可选

- nonce：整数 nonce 值，可选
  
  **消息调用**

- web3.eth.call(callObject [, defaultBlock] [, callback])

参数：

- 调用对象：与交易对象相同，只是from也是可选的
- 默认区块：默认“latest”，可以传入指定的区块高度
- 回调函数，如果没有则为同步调用

```js
var result = web3.eth.call({
            to: "0xc4abd0339eb8d57087278718986382264244252f",
            data: "0xc6888fa100000000000000000000000000000000000000000000000000
            0 0000000000003 " }); 
            console.log(result);
```

##### 日志过滤（事件监听）

```js
//web3.eth.filter(filterOptions[, callback])
// filterString 可以是 'latest' or 'pending' 
var filter = web3.eth.filter(filterString);
// 或者可以填入一个日志过滤 options 
var filter = web3.eth.filter(options);
// 监听日志变化
filter.watch(function(error, result) {
    if (!error) console.log(result);
});
// 还可以用传入回调函数的方法，立刻开始监听日志
web3.eth.filter(options, function(error, result) {
    if (!error) console.log(result);
});
```

##### 创建合约

```js
//web3.eth.contract
var MyContract = web3.eth.contract(abiArray);
// 通过地址初始化合约实例
var contractInstance = MyContract.at(address);
// 或者部署一个新合约
var contractInstance = MyContract.new([constructorParam1]
    [, constructorParam2], {
        data: '0x12345...',
        from: myAccount,
        gas: 1000000
    });
```

##### 调用合约函数

```js
// 直接调用，自动按函数类型决定用 sendTransaction 还是 call
myContractInstance.myMethod(param1[, param2, ...][,
    transactionObject
][, defaultBlock][, callback]);
// 显式以消息调用形式 call 该函数
myContractInstance.myMethod.call(param1[, param2, ...][,
    transactionObject
][, defaultBlock][, callback]);
// 显式以发送交易形式调用该函数
myContractInstance.myMethod.sendTransaction(param1[,
    param2, ...][, transactionObject][, callback]);
```

##### 监听合约事件

```js
var event = myContractInstance.MyEvent({
        valueA: 23
    }
    [, additionalFilterObject])
// 监听事件
event.watch(function(error, result) {
    if (!error) console.log(result);
});
//还可以用传入回调函数的方法，立刻开始监听事件
var event = myContractInstance.MyEvent([{
        valueA: 23
    }]
    [, additionalFilterObject],
    function(error, result) {
        if (!error) console.log(result);
    }
);
```

##### 使用web3编写自动转账脚本

**transfer_script.js**

```js
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'));

var arguments = process.argv.splice(2);

if(!arguments || arguments.length != 2){
    console.log("Parameter error!");
    return;
}

var _from = arguments[0];
var _to = arguments[1];
var _value = arguments[2];

web3.eth.sendTransaction({from: _from, to: _to, value: _value}, (err, res)=>{
    if(err)
        console.log("Error: ", err);
    else
        console.log("Tx: ", res);
});
```

```bash
node transfer_script.js 0x6903269a98f6ef7ea7190fc0122d5e15ddd25526 0x456982ded2a3eb889b3c3774e48de7fd867c0e5a 10000000
```

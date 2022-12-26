# solidity基础语法

### 概述

 Solidity接近javaScript，一种面对对象的语言。

### 基础语法

##### 数据类型

值类型 bool、unit、int8 to int 256、unit8 to unit 256、fixed/unfixed、fixedMxn

地址类型 address

引用类型 数组，struct,map  这些类型涉及到的数据量比较大，赋值他们要消耗大量gas

字符串是特殊的数组，是引用类型

##### 变量

**状态变量** 永久保存在合约存储空间中的变量。

**局部变量** 仅存储在函数执行过程中有效的变量

**全局变量** 保全在全局命名空间，用于获取区块链相关信息的变量

```javascript
pragma solidity ^0.5.0;
contract SolidityTest {
   uint storedData; // 状态变量
   constructor() public {
      storedData = 10;   
   }
   function getResult() public view returns(uint){
      uint a = 1; // 局部变量
      uint b = 2;
      uint result = a + b;
      return storedData; // 访问状态变量
   }
}
```

| 名称                                            | 返回                                   |
| --------------------------------------------- | ------------------------------------ |
| blockhash(uint blockNumber) returns (bytes32) | 给定区块的哈希值 – 只适用于256最近区块, 不包含当前区块。     |
| block.coinbase (address payable)              | 当前区块矿工的地址                            |
| block.difficulty (uint)                       | 当前区块的难度                              |
| block.gaslimit (uint)                         | 当前区块的gaslimit                        |
| block.number (uint)                           | 当前区块的number                          |
| block.timestamp (uint)                        | 当前区块的时间戳，为unix纪元以来的秒                 |
| gasleft() returns (uint256)                   | 剩余 gas                               |
| msg.data (bytes calldata)                     | 完成 calldata                          |
| msg.sender (address payable)                  | 消息发送者 (当前 caller)                    |
| msg.sig (bytes4)                              | calldata的前四个字节 (function identifier) |
| msg.value (uint)                              | 当前消息的wei值                            |
| now (uint)                                    | 当前块的时间戳                              |
| tx.gasprice (uint)                            | 交易的gas价格                             |
| tx.origin (address payable)                   | 交易的发送方                               |

##### 变量作用域

**Public** 公共状态变量可以在内部访问，也可以通过消息访问。公共状态变量，会生成一个自动getter函数。

**Internal** 内部状态变量只能从当前合约或者派生合约内访问。

**Private** 私有状态变量只能从当前合约内部访问，派生合约不能访问。

**external**外部访问，合约内不能访问

```js
pragma solidity ^0.5.0;
contract C {
   uint public data = 30;
   uint internal iData= 10;

   function x() public returns (uint) {
      data = 3; // 内部访问
      return data;
   }
}
contract Caller {
   C c = new C();
   function f() public view returns (uint) {
      return c.data(); // 外部访问
   }
}
contract D is C {
   uint storedData; // 状态变量

   function y() public returns (uint) {
      iData = 3; // 派生合约内部访问
      return iData;
   }
   function getResult() public view returns(uint){
      uint a = 1; // 局部变量
      uint b = 2;
      uint result = a + b;
      return storedData; // 访问状态变量
   }
}
```

##### 运算符

跟js一样，支持算术，比较，逻辑，赋值，条件等运算符。

##### 数据存储位置

提供4种类型的数据位置

- Storage 永久存储，成本高。

- Memory 临时数据，比存储位置便宜。类似RAM 

- CallData 类似入参的存储位置。

- Stack EVM的堆栈

```js
pragma solidity ^0.5.0;  

contract Locations {  

  /* 此处都是状态变量 */  

  // 存储在storage  
  bool flag;  
  uint number;  
  address account;  

  function doSomething() public  {  

    /* 此处都是局部变量  */  

    // 值类型
    // 所以它们被存储在内存中
    bool flag2;  
    uint number2;  
    address account2;  

    // 引用类型，需要显示指定数据位置，此处指定为内存
    uint[] memory localArray;
    //引用类型   此处为storage     
    uint[] x localArray2;   
  }  
}   
```

```js
//经典问题    调用f函数，a的值会变化
//非定长数据都会存储数据的len   a的位置被非定长数组存储了数据长度   
pragma solidity ^0.5.0;  

contract Locations {  
    unit public a;
    unit[] public data;
    function f() public{
        unit[] x;//此处如果不定义内容只是指针   那会指向storage的第一个空间，就是a
        x.push(2);
        data = x;
    }
}


//解决方案
function f() public{
        unit[] x = data;
        x.push(2);
}
```

```js
//经典问题    调用f函数，a的值会变化
//mem类型的变量  一旦长度确定   就不会在变
//  
pragma solidity ^0.5.0;  

contract c{  

    unit[] public x;
    function f(unit[] menoryArray) public{
        //将mem类型的值赋值给storage，会拷贝整个副本过去
        x = memoryArray;
        unit[] y = x;//y是个引用变量
        y[7];
        y.length =2 ;//这里就等于只保留两个长度  
        delete x;//软删除   就是把长度变成0
        y = memoryArray;//这里就有编译错误   无法将mem的小内存 指向  一开始定义的storage大内存中
        unit[] memory z = memoryArray;//这样就可以
        delete y;

    }

}
```

##### 字符串

```js
pragma solidity ^0.5.0;

contract SolidityTest {
   string data = "test";
   bytes32 data = "test";
}
```

##### 数组

```js
pragma solidity ^0.5.0;

contract C {
    bytes32 data = "test";
    function f(uint len) {

        uint[] memory a = new uint[](7);
        bytes memory b = new bytes(len);
        // a.length == 7, b.length == len
        a[6] = 8;
        b.push(byte(8));
    }
}
```

bytes比byte[]更省油

(bytes4) calldata 的前四个字节 (function identifier)

##### Enum

```js
pragma solidity ^0.5.0;

contract test {
   enum FreshJuiceSize{ SMALL, MEDIUM, LARGE }//0,1,2
   FreshJuiceSize choice;
   FreshJuiceSize constant defaultChoice = FreshJuiceSize.MEDIUM;

   function setLarge() public {
      choice = FreshJuiceSize.LARGE;
   }
   function getChoice() public view returns (FreshJuiceSize) {
      return choice;
   }
   function getDefaultChoice() public pure returns (uint) {
      return uint(defaultChoice);//这里返回的其实是1
   }
}
```

##### 结构体

```js
pragma solidity ^0.5.0;

contract test {
   struct Book { 
      string title;
      string author;
      uint book_id;
   }
   Book book;

   function setBook() public {
      book = Book('Learn Sol', 'sincw', 1);
   }
   function getBookId() public view returns (uint) {
      return book.book_id;
   }
}
```

##### Map

```js
pragma solidity ^0.5.0;

contract LedgerBalance {
   mapping(address => uint) public balances;

   function updateBalance(uint newBalance) public {
      balances[msg.sender] = newBalance;
   }
}
contract Updater {
   function updateBalance() public returns (uint) {
      LedgerBalance ledgerBalance = new LedgerBalance();
      ledgerBalance.updateBalance(10);
      return ledgerBalance.balances(address(this));
   }
}
```

##### 

##### 类型转换

跟java一样，支持显示转换

转换为更小的类型时会丢失高位

转为为更大类型时会向左侧添加填充位

不能超出范围，会发生截断

```js
int8 y = -3;
uint x = uint(y);
uint16 a = 0x1234;
uint32 b = uint32(a); // b = 0x00001234
bytes2 a = 0x1234;
bytes1 b = bytes1(a); // b = 0x12
```

##### 以太单位

最小单位为伟

```js
assert(1 wei == 1);
assert(1 szabo == 1e12);
assert(1 finney == 1e15);
assert(1 ether == 1e18);
assert(2 ether == 2000 fenny);
```

##### 枚举

```js
library Math {
    enum Rounding {
        Down, // Toward negative infinity
        Up, // Toward infinity
        Zero // Toward zero
    }
}
```

##### 函数

- **public** - （任意访问，作为合约接口）可以通过内部调用或通过消息调用。对公共状态变量而言，会有的自动访问限制符的函数生成。  

- **private** - （仅当前合约内）私有函数和状态变量仅仅在定义该合约中可见， 在派生的合约中不可见。  

- **internal** - （仅当前合约及所继承的合约）这些函数和状态变量只能内部访问(即在当前合约或由它派生的合约)，而不使用（关键字）this ,直接调用。  

- **external** - （仅外部访问，也是合约接口）它们可以从其他合约调用, 也可以通过事务调用。外部函数f不能被内部调用（在内部也只能用外部访问方式访问，即 f()不执行,但this.f()执行）。

```js
pragma solidity >0.4.22;
contract SimpleSrorage{
    unit myData;
    constructor(unit newData) public {
        myData = newData;
    }
    function setData(unit newData) public view {
         require(msg.sender == owner);    
         myData = newData;
    }
    function getData() public view resutns(unit){
         return myData;
    }
    function pureAdd(unit a,unit b) public pure returns(unit sum,unit _a){

        reutrn (a + b,a);
    }
}
```

- **require**  类似if,如果条件不满足则抛出异常

- **view**  函数声明为view时，则函数不会修改状态，如果修改状态会发出警告，什么是修改状态
  
  - 修改状态变量
  
  - 产生事件
  
  - 使用selfdestruct
  
  - 调用发送以太
  
  - 调用任何没有标记为view pure的
  
  - 使用低级调用   call
  
  - 包含内联汇编

- **pure**  函数声明为pure时，不读取或修改状态，什么是读取
  
  - 读取状态变量
  
  - 访问this.balance
  
  - 访问block,tx,msg
  
  - 调用任何未标记为pure的函数
  
  - 包含内联汇编

- **payable**，说明该函数外部调用的时候必须发送ether和gas，没有加payable的函数，则没有方法接受 ETH， 附加ETH调用会出错。address  添加payable才能发送eth

- fallback   回退函数
  
  当合约中不存在的函数被调用时，将调用fallback函数。
  被标记为外部函数。
  它没有名字。
  它没有参数。
  它不能返回任何东西。
  每个合约定义一个fallback函数。
  如果没有被标记为payable，则当合约收到无数据的以太币转账时，将抛出异常。

- 函数可以重载，跟java类似，方法名相同参数不同

- 数学函数   内置了一些数学工具函数 addmod(4, 5, 3); mulmod(4, 5, 3);

- 加密函数   

```javascript
pragma solidity ^0.5.0;

contract Test {   
   function callKeccak256() public pure returns(bytes32 result){
      return keccak256("ABC");
   }  
}
```

`keccak256(bytes memory) returns (bytes32)` 计算输入的Keccak-256散列。

`sha256(bytes memory) returns (bytes32)` 计算输入的SHA-256散列。

`ripemd160(bytes memory) returns (bytes20)` 计算输入的RIPEMD-160散列。

`ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)` 从椭圆曲线签名中恢复与公钥相关的地址，或在出错时返回零。函数参数对应于签名的ECDSA值: r – 签名的前32字节; s: 签名的第二个32字节; v: 签名的最后一个字节。这个方法返回一个地址。

- **transfer、send、call** 

都是合约转账，建议用call   a.transfer(msg.value) ,合约转账给a账户本次交易发送的金额。

如果使用addr.transfer(1 ether)、addr.send(1 ether)，addr合约中必须增加fallback回退函数！如果使用addr.call.value(1 ether)，那么被调用的方法必须添加payable修饰符，否则转账失败！

```js
address(a).call.value(1)(abi.encodeWithSignature("register(string)", "MyName"));

//合约转账给a地址 1wei并且调用a地址的register,参数为MyName
address(a).call.value(1)()
//这里表示调用的是fallback函数
//call()的返回结果是一个bool，表示是否成功的调用，或者是失败引起了EVM异常。该方法无法直接访问函数返回结果(因为需要事前知道编码和返回结果大小)。
```

- delegatecall

call与delegatecall的功能类似，区别仅在于后者仅使用给定地址的代码，其它信息则使用当前合约(如存储，余额等等)。

函数的设计目的是为了使用存储在另一个合约的库代码。

二者执行代码的上下文环境的不同，当使用call调用其它合约的函数时，代码是在被调用的合约的环境里执行，对应的，使用delegatecall进行函数调用时代码则是在调用函数的合约的环境里执行

- staticcall

针对staticcall与call，delegatecall的区别，简单来说就是staticcall不允许存在状态改变

##### 限制(restricted)访问

- **onlyBy** 限制可以调用该函数的调用者（根据地址）。

- **onlyAfter** 限制该函数只能在特定的时间段之后调用。

- **costs** 调用方只能在提供特定值的情况下调用此函数。。
  
  ```js
  pragma solidity ^0.5.0;
  
  contract Test {
     address public owner = msg.sender;
     uint public creationTime = now;
  
     modifier onlyBy(address _account) {
        require(
           msg.sender == _account,
           "Sender not authorized."
        );
        _;
     }
     function changeOwner(address _newOwner) public onlyBy(owner) {
        owner = _newOwner;
     }
     modifier onlyAfter(uint _time) {
        require(
           now >= _time,
           "Function called too early."
        );
        _;
     }
     function disown() public onlyBy(owner) onlyAfter(creationTime + 6 weeks) {
        delete owner;
     }
     modifier costs(uint _amount) {
        require(
           msg.value >= _amount,
           "Not enough Ether provided."
        );
        _;
        if (msg.value > _amount)
           msg.sender.transfer(msg.value - _amount);
     }
     function forceOwnerChange(address _newOwner) public payable costs(200 ether) {
        owner = _newOwner;
        if (uint(owner) & 0 == 1) return;        
     }
  }
  ```

##### 合约继承

继承通过关键字 is 来实现

- 访问权限 
  
   1 子类不能访问父类的private的，其他的都可以访问，比如Public interna
  
   2 继承不允许函数或变量重名

- 重写函数
  
  从 0.6 开始，solidity 引入了 abstract, virtual, override 几个关键字，用于重写函数。

- override不允许改变可见性

```js
pragma solidity ^0.8.0;

abstract contract Employee {
    function getSalary() public  pure virtual returns(int);
}

contract Manager is Employee {
    function getSalary() public  pure override returns(int){
        return 2;
    }
}
```

##### 构造函数

- 一个合约只能有一个构造函数。

- 构造函数在创建合约时执行一次，用于初始化合约状态。

- 在执行构造函数之后，合约最终代码被部署到区块链。合约最终代码包括公共函数和可通过公共函数访问的代码。构造函数代码或仅由构造函数使用的任何内部方法不包括在最终代码中。

- 构造函数可以是公共的，也可以是内部的。

- 内部构造函数将合约标记为抽象合约。

- 如果没有定义构造函数，则使用默认构造函数。

- 如果基合约具有带参数的构造函数，则每个派生/继承的合约也都必须包含参数。

```js
pragma solidity ^0.5.0;

contract Base {
   uint data;
   constructor(uint _data) public {
      data = _data;   
   }
}
contract Derived is Base (5) {
   constructor() public {}
}
contract Derived2 is Base {
   constructor(uint _info) Base(_info * _info) public {}
}
```

##### 抽象合约

类似java中的抽象类，抽象合约至少包含一个没有实现的函数（抽象函数）。

```js
pragma solidity ^0.5.0;

contract Calculator {
   function getResult() public view returns(uint);
}

contract Test is Calculator {
   function getResult() public view returns(uint) {
      uint a = 1;
      uint b = 2;
      uint result = a + b;
      return result;
   }
}
```

##### web3调用合约

```bash
var address="0xca514484194b5ae9467043fdcb96df5ed12099a7"
var metacoin = web3.eth.contract(abi).at(address);
```

##### 接口

接口类似于抽象合约，使用`interface`关键字创建，接口只能包含抽象函数，不能包含函数实现。以下是接口的关键特性：

- 接口的函数只能是外部类型。
- 接口不能有构造函数。
- 接口不能有状态变量。
- 接口可以包含enum、struct定义，可以使用`interface_name.`访问它们。

```js
pragma solidity ^0.5.0;

interface Calculator {
   function getResult() external view returns(uint);
}

contract Test is Calculator {
   constructor() public {}
   function getResult() external view returns(uint){
      uint a = 1; 
      uint b = 2;
      uint result = a + b;
      return result;
   }
}
```

##### 库

```js
pragma solidity ^0.5.0;

library Search {
   function indexOf(uint[] storage self, uint value) public view returns (uint) {
      for (uint i = 0; i < self.length; i++) if (self[i] == value) return i;
      return uint(-1);
   }
}
contract Test {
   uint[] data;
   constructor() public {
      data.push(1);
      data.push(2);
      data.push(3);
      data.push(4);
      data.push(5);
   }
   function isValuePresent() external view returns(uint){
      uint value = 4;

      // 使用库函数搜索数组中是否存在值
      uint index = Search.indexOf(data, value);

//using A for B指令，可用于将库A的函数附加到给定类型B。这些函数将把调用者类型作为第一个参数(使用self标识)。
contract Test2 {
   using Search for uint[];
   uint[] data;
   constructor() public {
      data.push(1);
      data.push(2);
      data.push(3);
      data.push(4);
      data.push(5);
   }
   function isValuePresent() external view returns(uint){
      uint value = 4;      

      // data 表示库
      uint index = data.indexOf(value);
      return index;
   }
}  return index;
   }
}
```

##### 汇编(Assembly)

可以在Solidity源程序中嵌入汇编代码,主要在编写库函数时使用

```js
pragma solidity ^0.5.0;

library Sum {   
   function sumUsingInlineAssembly(uint[] memory _data) public pure returns (uint o_sum) {
      for (uint i = 0; i < _data.length; ++i) {
         assembly {
            o_sum := add(o_sum, mload(add(add(_data, 0x20), mul(i, 0x20))))
         }
      }
   }
}
```

##### 事件的监听

事件是智能合约发出的信号，智能合约的前端可以接收到这个信息。

```js
pragma solidity ^0.4.21;

contract Multicounter {
    mapping (uint256 => uint256) public counts;

    event Increment(uint256 indexed which, address who);

    function increment(uint256 which) public {
        emit Increment(which, msg.sender);
        counts[which] += 1;
    }
}
```

web3.js前端监听

最多有三个参数

事件不能从Ethereum虚拟机(EVM)中访问。这意味着合约不能读取自己的或其他合约的日志及事件。

```js
... 

counter.Increment({ which: counterId }, function (err, result) {
  if (err) {
    return error(err);
  }

  log("Counter " + result.args.which + " was incremented by address: "
      + result.args.who);
  getCount();
});

...
getCount();
```

### Indexed属性

可以在事件参数上增加`indexed`属性，最多可以对三个参数增加这样的属性。加上这个属性，可以允许你在web3.js中通过对加了这个属性的参数进行值过滤

```js
var event = myContract.transfer({value: "100"});
```

增加了`indexed`的参数值会存到日志结构的`Topic`部分，便于快速查找。而未加`indexed`的参数值会存在`data`部分，成为原始日志。需要注意的是，如果增加`indexed`属性的是数组类型（包括`string`和`bytes`），那么只会在`Topic`存储对应的数据的`web3.sha3`哈希值，将不会再存原始数据。因为Topic是用于快速查找的，不能存任意长度的数据，所以通过Topic实际存的是数组这种非固定长度数据哈希结果。要查找时，是将要查找内容哈希后与Topic内容进行匹配，但我们不能反推哈希结果，从而得不到原始值。

##### 错误处理

错误处理中，使用的一些重要方法：

- `assert(bool condition)` − 如果不满足条件，此方法调用将导致一个无效的操作码，对状态所做的任何更改将被还原。这个方法是用来处理内部错误的。

- `require(bool condition)` − 如果不满足条件，此方法调用将恢复到原始状态。此方法用于检查输入或外部组件的错误。

- `require(bool condition, string memory message)` − 如果不满足条件，此方法调用将恢复到原始状态。此方法用于检查输入或外部组件的错误。它提供了一个提供自定义消息的选项。

- `revert()` − 此方法将中止执行并将所做的更改还原为执行前状态。

- `revert(string memory reason)` − 此方法将中止执行并将所做的更改还原为执行前状态。它提供了一个提供自定义消息的选项。
  
  任何错误处理直接回退，会消耗gas

##### TRY/CATCH

```js

```

##### ERC20

学习了基础语法后，阅读一遍ERC20的代码，solidity基本就掌握了

[openzeppelin-contracts/contracts/token/ERC20 at master · OpenZeppelin/openzeppelin-contracts · GitHub](https://github.com/OpenZeppelin/openzeppelin-contracts/tree/master/contracts/token/ERC20)

https://blog.csdn.net/linkcmath/article/details/126300452

https://www.jianshu.com/p/131c07c6f72f

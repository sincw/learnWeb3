# 什么是ERC20

**ERC20（Ethereum Request for Comments 20）一种代币标准。**[**EIP-20**](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) 中提出。

ERC20 代币合约跟踪同质化（可替代）代币：任何一个代币都完全等同于任何其他代币；没有任何代币具有与之相关的特殊权利或行为。这使得 ERC20 代币可用于交换货币、投票权、质押等媒介。

# 为什么要遵守ERC20

允许以太坊上的任何代币被其他应用程序（从钱包到去中心化交易所）重新使用的标准接口。以太坊上的所有应用都默认支持 ERC20 ，如果你想自己发币，那么你的代码必须遵循 ERC20 标准，这样钱包（如MetaMask）等应用才能将你的币显示出来。

# ERC20源码解读

![](image\ERC20.png)

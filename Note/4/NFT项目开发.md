# NFT项目开发

b站很多教程，可以挑选自己喜欢的nft来做

建议先阅读ERC20和ERC721源码，代码其实不复杂

### 1、环境安装

vsCode + node + gancche

hardhat、openzeppelin、react、chakra-ui

```bash
npm install -g install
// npminit
npx create-react-app wxnft
npm i -D hardhat
npx hardhat

npm i @openzeppelin/contracts

npm i @chakra-ui/react @emotion/react @emotion/styled framer-motion
npm i -D dotenv

npx hardhat clean
npx hardhat compile

npx hardhat run scripts/deploy.js --network rinkeby
miner.start(1);admin.sleepBlocks(1);miner.stop();
```

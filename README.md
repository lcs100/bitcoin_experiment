# bitcoin_experiment_based_Docker
## before start
本教程基于Ubuntu16.04，而非windows  
建议用户使用双系统或者虚拟机安装Ubuntu16.04(或18.04)  
本教程将介绍比特币测试网的搭建以及测试网的使用  

## Prerequisites
1、Docker  
参考网址:[click here](https://www.runoob.com/docker/ubuntu-docker-install.html)  

2、make  
```
$ sudo apt-get install make
```

## Install Docker image of bitcoin-testnet
```
$ docker pull freewil/bitcoin-testnet-box
```

## Run bitcoin testnet
输入命令  
```
$ docker run -t -i -p 19001:19001 -p 19011:19011 freewil/bitcoin-testnet-box
```
输出如下所示，可以看到系统已经进入Docker，默认目录是~/bitcoin-testnet-box中  
```
tester@0bfc5154a423 ~/bitcoin-testnet-box$
```
接着执行下面命令，默认是在本地启动两个比特币节点，组成比特币测试网络  
```
$ make start
```
输出如下所示，可以看出本地的两个节点已经在后台运行  
```
bitcoind -datadir=1  -daemon
Bitcoin server starting
bitcoind -datadir=2  -daemon
Bitcoin server starting
```
输入下面命令，可以查看测试网络节点状态信息：  
```
$ make getinfo
```
结果如下所示，可以看到输出的信息是以类似于json格式表示的，其中展现了：  
当前比特币版本号、协议版本号、钱包版本号  
余额、区块数量、连接数、代理、当前挖矿难度值等  
```
bitcoin-cli -datadir=1  -getinfo
{
  "version": 170100,
  "protocolversion": 70015,
  "walletversion": 169900,
  "balance": 0.00000000,
  "blocks": 0,
  "timeoffset": 0,
  "connections": 1,
  "proxy": "",
  "difficulty": 4.656542373906925e-10,
  "testnet": false,
  "keypoololdest": 1584693069,
  "keypoolsize": 1000,
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "warnings": ""
}
bitcoin-cli -datadir=2  -getinfo
{
  "version": 170100,
  "protocolversion": 70015,
  "walletversion": 169900,
  "balance": 0.00000000,
  "blocks": 0,
  "timeoffset": 0,
  "connections": 1,
  "proxy": "",
  "difficulty": 4.656542373906925e-10,
  "testnet": false,
  "keypoololdest": 1584693069,
  "keypoolsize": 1000,
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "warnings": ""
}
```

## creat address and block data
输入如下命令：  
```
$ bitcoin-cli -datadir=1 getnewaddress
```
可以得到一个字符串，即比特币地址  
```
2NBhfVG1QrMMEdKP77Hq9fj5YJBtb3cq7T3
```
同样输入第二条命令：  
```
$ bitcoin-cli -datadir=2 getnewaddress
```
得到第二个地址：  
```
2NB71PgMkmebtDCoV5ysBWNEWVdJGm1xeBs
```
之后是查看每个地址的私钥，操作和结果如下所示：  
```
tester@0bfc5154a423 ~/bitcoin-testnet-box$ bitcoin-cli -datadir=1 dumpprivkey 2NBhfVG1QrMMEdKP77Hq9fj5YJBtb3cq7T3
cT42ZMFsF4T2KjNSe4rmBiXjaPMugJB1TNd8XqCpopMF8JSfnyos
tester@0bfc5154a423 ~/bitcoin-testnet-box$ bitcoin-cli -datadir=2 dumpprivkey 2NB71PgMkmebtDCoV5ysBWNEWVdJGm1xeBs
cNxWZoWMYN2PRRLA4Db48XFnnW9yMQcskaRizdnWp2qg8tqQ2zg3
```
接下来是创建区块，命令如下，其中100是可以自改动的  
```
$ make generate BLOCKS=100
```
结果类似于下面：  
```
bitcoin-cli -datadir=1  generate 100
[
  "71d56b7f67d6b490e5f7cfa5f5be592eb3b238780aabafd1b7c26bca94c68cbc",
  ...
```
现在看一下钱包的状态，命令和操作如下所示：  
```
tester@0bfc5154a423 ~/bitcoin-testnet-box$ bitcoin-cli -datadir=1 getwalletinfo
{
  "walletname": "",
  "walletversion": 169900,
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 5000.00000000,
  "txcount": 100,
  "keypoololdest": 1584693069,
  "keypoolsize": 999,
  "keypoolsize_hd_internal": 1000,
  "paytxfee": 0.00000000,
  "hdseedid": "ec239a39720337ffccac5818d15d37576943ee96",
  "hdmasterkeyid": "ec239a39720337ffccac5818d15d37576943ee96",
  "private_keys_enabled": true
}
```
可以看出这个钱包中还没有金钱，因为测试网中挖块的奖励费需要等100个区块深度以后才可以进入钱包  
继续生成200个区块，再看结果：  
```
tester@0bfc5154a423 ~/bitcoin-testnet-box$ bitcoin-cli -datadir=1 getwalletinfo
{
  "walletname": "",
  "walletversion": 169900,
  "balance": 8725.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 2487.50000000,
  "txcount": 300,
  "keypoololdest": 1584693069,
  "keypoolsize": 999,
  "keypoolsize_hd_internal": 1000,
  "paytxfee": 0.00000000,
  "hdseedid": "ec239a39720337ffccac5818d15d37576943ee96",
  "hdmasterkeyid": "ec239a39720337ffccac5818d15d37576943ee96",
  "private_keys_enabled": true
}
```
之后将地址一中10个比特币转给地址二，命令和结果如下所示：  
```
tester@0bfc5154a423 ~/bitcoin-testnet-box$ bitcoin-cli -datadir=1 sendtoaddress 2NB71PgMkmebtDCoV5ysBWNEWVdJGm1xeBs 10
19fd1c218a3774de3eb4250e85b2f784ec6c63ecbfddf07a138d2af62ffdf6e7
```
再次查询地址二钱包中的比特币数量：  
```
tester@0bfc5154a423 ~/bitcoin-testnet-box$ bitcoin-cli -datadir=2 getwalletinfo
{
  "walletname": "",
  "walletversion": 169900,
  "balance": 0.00000000,
  "unconfirmed_balance": 10.00000000,
  "immature_balance": 0.00000000,
  "txcount": 1,
  "keypoololdest": 1584693069,
  "keypoolsize": 999,
  "keypoolsize_hd_internal": 1000,
  "paytxfee": 0.00000000,
  "hdseedid": "62199f8834dd279cc2e9b263a20f8dac3f78a37a",
  "hdmasterkeyid": "62199f8834dd279cc2e9b263a20f8dac3f78a37a",
  "private_keys_enabled": true
}
```
可以看出未认证的余额有10个比特币，我们需要在生成区块对这笔交易进行认证  
所以继续生成10个比特币，再次观察钱包，可以看出10个比特币已经进入余额中。  
```
tester@0bfc5154a423 ~/bitcoin-testnet-box$ bitcoin-cli -datadir=2 getwalletinfo
{
  "walletname": "",
  "walletversion": 169900,
  "balance": 10.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 1,
  "keypoololdest": 1584693069,
  "keypoolsize": 999,
  "keypoolsize_hd_internal": 1000,
  "paytxfee": 0.00000000,
  "hdseedid": "62199f8834dd279cc2e9b263a20f8dac3f78a37a",
  "hdmasterkeyid": "62199f8834dd279cc2e9b263a20f8dac3f78a37a",
  "private_keys_enabled": true
}
```

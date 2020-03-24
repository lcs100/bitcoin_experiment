# bitcoin_experiment_based_Ubuntu
## Before start
本教程基于Ubuntu16.04纯净版  
将介绍如何搭建比特币全量节点，并对一些常用命令进行介绍  
并分析比特币目录中每个目录的具体作用  

## Prerequisites
1、git  
```
sudo apt-get install git
```

2、make 
```
sudo apt-get install make
```

3、必须依赖： 
    
    libboost | 
    libevent 
4、可选依赖：  
    miniupnpc
    libdb4.8
    qt
    libqrencode
    univalue
    libzmq3


## Get started
1、首先是下载比特币源码：  
```
$ git clone https://github.com/bitcoin/bitcoin.git
```
下载完成后，当前目录下会出现bitcoin目录，切换到该目录中：  
```
cd bitcoin
```
之后选择比特币源码的版本，根据书上的做法，这里选择稳定版v0.11.2  
```
$ git tag
...
$ git checkout v0.11.2
```
之后可以进行确定是否已经成功切换版本，命令如下：  
```
$ git status
```
2、之后开始执行操作：  
```
$ ./autogen.sh
```
该文件根据具体的电脑系统和配置自动产生合适的配置选项并确保电脑拥有所有必须的依赖库  

接着是文件configure  
最重要的是文件configure，如果需要帮助，可以执行命令：  
```
$ ./configure --help
```
这里显示几种选项：  
```
--prefix=$HOME
```
This overrides the default installation location (which is /usr/local/) for the resulting executable. Use $HOME to put everything in your home directory, or a different path.
```
--disable-wallet
```
This is used to disable the reference wallet implementation.
```
--with-incompatible-bdb
```
If you are building a wallet, allow the use of an incompatible version of the Berkeley DB library.
```
--with-gui=no
```
Don’t build the graphical user interface, which requires the Qt library. This builds server and command-line bitcoin only.

3、运行命令：  
```
$ ./configure
```
这个命令会自动发现所有必须的依赖库并且并为系统创建一个构造脚本  

4、构建比特币核心执行程序：  
命令如下所示：  
```
$ make
```
如果一切顺利的话，该命令结束后就会产生核心程序了  
接着命令如下所示：  
```
$ sudo make install
```
这个命令将核心程序安装到系统中，默认位置是/usr/local/bin  
如果一切顺利的话，可以查看具体的位置：  
```
$ which bitcoind
```
或者
```
$ which bitcoin-cli
```

5、运行比特币核心程序
在执行之前，需要建立一个配置文件，该文件至少包括rpcuser和rpcpassword条目，另外建议设置警报机制  
比特币默认的主目录是.bitcoin，是一个隐藏目录，在用户的主目录中。  
在该目录中创建一个文件，命名为.bitcoin/bitcoin.conf，并在其中提供用户和密码：  
```
rpcuser=bitcoinrpc
rpcpassword=YOUR_PASSOWRD
```
除了上述两项之外，还有很多其他参数可以设置，这里仅介绍几个比较重要的参数：  
datadir:  
```
选择要存放所有区块链数据的目录和文件系统。系统默认选择账户主目录的.bitcoin子目录。要确保这个文件系统具有几GB的可用空间
```
txindex:  
```
维护所有交易的索引。这意味着区块链的完整副本，以编程方式通过ID检索任何交易

```
maxmempool
```
设定最大的保存交易的内存空间
```

好了。接下来是一个全量节点的配置文件：  
```
alertnotify=myemailscript.sh "Alert: %s"
datadir=/lotsofspace/bitcoin
txindex=1
rpcuser=bitcoinrpc 
rpcpassword=CHANGE_THIS
```
如果机器配置不是很好，下面是一个低配置的文件：  
```
alertnotify=myemailscript.sh "Alert: %s"
maxconnections=15
prune=5000
minrelaytxfee=0.0001
maxmempool=200 
maxreceivebuffer=2500 
maxsendbuffer=500 
rpcuser=bitcoinrpc 
rpcpassword=CHANGE_THIS
```

配置完文件后就可以开始运行了  
第一个命令是查看程序运行时的输出：  
```
$ bitcoind -printtoconsole
```
第二个命令是如果想把比特币程序放在后台程序运行：  
```
$ bitcoind -daemon
```
第三个命令是监视比特币的进度和运行状态：  
```
$ bitcoin-cli getinfo
```

## Bitcoin Core Application API
下面是针对比特币核心客户端进行说明。核心客户端实现了JSON-RPC接口，该接口可以使用命令行帮助程序
bitcoin-cli访问。
第一个命令是查看可用的比特币RPC命令列表：  
```
$ bitcoin-cli help
```
### 获得比特币核心客户端状态信息
```
$ bitcoin-cli getinfo
```
这个命令显示关于比特币网络节点、钱包、区块链数据状态的基础信息。结果如下所示：  
```
$ bitcoin-cli getinfo 
{
    "version" : 110200,
    "protocolversion" : 70002,
    "blocks" : 396367,
    "timeoffset" : 0,
    "connections" : 15,
    "proxy" : "",
    "difficulty" : 120033340651.23696899,
    "testnet" : false,
    "relayfee" : 0.00010000,
    "errors" : "" 
}
```
再返回的数据中可以看到:  
比特币软件客户端版本号(version)  
比特币协议版本号(protocolversoin)  
区块链高度(blocks)  
等  

### 找交易并解码
命令如下：  
```
$ bitcoin-cli getrawtransaction 0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2
```
将返回的结果进行解码，命令如下：  
```
$ bitcoin-cli bitcoin-cli decoderawtransaction 0100000001186f9f998a5aa6f048e51dd8419a14d8a0f1a8a2836dd734d2804fe65fa35779000000008b483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adfffffffff0260e31600000000001976a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788acd0ef8000000000001976a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac00000000
```
执行这个命令后就可以看到具体的交易的组成部分  

### 探究区块
命令如下：  
```
$ bitcoin-cli getblockhash 277316
```
和
```
$ bitcoin-cli getblock 0000000000000001b6b9a13b095e96db41c4a928b97ef2d944a9b31b2cc7bdc4
```

## .bitcoin目录分析







## Other
如果想继续深入了解比特币的相关知识，那么需要继续深入学习
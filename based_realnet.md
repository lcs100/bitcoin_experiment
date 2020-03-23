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
在执行之前，需要建立一个配置文件，至少有一个rpcuser和rpcpassword条目，另外还需要设置警报机制  
比特币默认的主目录是.bitcoin，是一个隐藏目录，在用户的主目录中。  
在该目录中创建一个文件，命名为.bitcoin/bitcoin.conf，并在其中提供用户和密码：  
```
rpcuser=bitcoinrpc
rpcpassword=YOUR_PASSOWRD
```
住了上述两项之外，还有很多其他条目：  
datadir:  
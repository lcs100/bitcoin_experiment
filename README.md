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
sudo apt-get install make
```


## Install Docker
安装Docker可以去官网下载最新镜像

```
docker pull freewil/bitcoin-testnet-box
```

## Run bitcoin testnet
运行命令，开启Docker镜像
```
docker run -t -i -p 19001:19001 -p 19011:19011 freewil/bitcoin-testnet-box
```
结果如下图所示：

之后切换到目录freewil/bitcoin-testnet-box  
执行命令
```
make start
```
输出结果如下图所示：






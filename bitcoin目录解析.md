# Bitcoin Core Data Storage
## Overview
本地目录主要包括四部分：  
1、blocks/blk*.dat  
这些文件是真正存数据的地方，以网络格式存在磁盘中，每个文件大小是128MB左右。  
作用是：在寻找扫描钱包中丢失的事务，重新组织到链的不同部分以及将块数据提供给其他节点  
2、blocks/index/*  
这是一个LevelDB数据库，其中包含有关所有已知块的元数据，以及在磁盘上找到它们的位置。没有这个，找到一个区块很慢  
3、chainstate/*  
这个目录也是一个LevelDB数据库，表示的是目前没有花出去的交易输出，即UTXO集合。以及一些关于这些交易的元数据  
这里的数据十分重要，尤其是对于新产生的区块以及交易集合。  
4、blocks/rev*.dat  
这个目录包含的文件是“没有处理的数据”。可以将其视为链的“补丁”，对链回滚的状态很有帮助，在重组情况下是必需的  

## Raw Block Data
区块文件保存随意的区块，并没有特定的形式  
区块文件大小约是128MB，会被16MB的磁盘块分割。
每一个区块文件都有一个对应的rev*.dat，其中包含在重组（叉）的情况下从区块链中删除区块所需的数据  
关于该文件的信息，都会存储在区块索引中，即LevelDB，在两个地方：  
1、关于文件的通用信息被存放在区块索引LevelDB中，即以“f”开头的四位数字键值，包括：  
    文件中存储的区块数量  
    文件大小  
    高度最低的区块以及高度最高的区块  
    时间戳-最早的时间戳以及最晚的时间戳  
2、找到具体某个区块的信息是以“b”开头的记录：  
    每一个区块包括了一个指针，指向磁盘中数据的位置（文件号以及偏移量）


### 从代码访问区块数据
区块数据文件可以通过以下访问：  
1、DiskBlockPos：一个结构体，包括一个指针指向磁盘中数据的位置，即一个文件号码和位移  
2、vInfoBlockFiles：一个区块文件信息对象的向量，这个变量用来：  
    决定新的区块是否可以填进当前的文件或者需要创建一个新文件进行创建  
    通过区块和未做的文件计算磁盘总使用量  
    遍历区块文件找到可以修剪的文件  

区块在接收到后立即通过函数AcceptBlock写入磁盘。（实际的磁盘写操作在WriteBlockToDisk中）。访问块文件的代码与访问并写入硬币数据库（/chainstate）的代码有些重叠。何时将状态刷新到磁盘存在比较复杂系统。这些代码都不会影响块文件，这些块文件在接收到时会简单地写入磁盘，接收并存储它们后，仅需要块文件才能将块提供给其他节点。  

## Block Index（leveldb）
区块索引拥有关于已知区块的元数据，包括这些区块在磁盘的具体位置  
“已知区块”集合是最长链的扩展集，因为它包括了已接收和处理但不属于活动链的区块，例如，孤块在一些情况下就与活动链分离  

### Terminology
区块树  
对于存储在磁盘上的一组一直区块，更好的叫法是“块树”，因为该术语涵盖了具有来自主链的众多分支（尽管很小）的树结构。  

键值对  
在实际LevelDB内部，键值对被用做：  
```
   'b' + 32-byte block hash -> block index record. Each record stores:
       * The block header
       * The height.
       * The number of transactions.
       * To what extent this block is validated.
       * In which file, and where in that file, the block data is stored.
       * In which file, and where in that file, the undo data is stored.
```
可以看到这里，以b开头的键的value是block index record  
存储的是区块头、区块的高度、交易的数量、区块被验证的程度、区块数据被保存那个文件以及文件的具体位置、未操作数据所在文件位置以及文件的具体位置  
```
  'f' + 4-byte file number -> file information record. Each record stores:
       * The number of blocks stored in the block file with that number.
       * The size of the block file with that number ($DATADIR/blocks/blkNNNNN.dat).
       * The size of the undo file with that number ($DATADIR/blocks/revNNNNN.dat).
       * The lowest and highest height of blocks stored in the block file with that number.
       * The lowest and highest timestamp of blocks stored in the block file with that number.
```
以f开头的键的value是file information record  
存储的是文件中存储区块的数量、区块文件的大小、未操作数据文件的大小、最高高度的区块和最低区块的区块、最早时间戳和最晚时间戳  
```
   'l' -> 4-byte file number: the last block file number used.
```
以l开头的键的value是file number  
最新的保存区块链的文件号码  
```
   'R' -> 1-byte boolean ('1' if true): whether we're in the process of reindexing.
```
以‘R’开头的键的value是一字节的布尔值，表示的是是否处于重新排序的进程中  
```
   'F' + 1-byte flag name length + flag name string -> 1 byte boolean ('1' if true, '0' if false): various flags that can be on or off. Currently defined flags include:
        * 'txindex': Whether the transaction index is enabled.
```
以‘F’开头的键的value是一字节布尔值   
表示的是关于交易是否可以通过索引进行查询？（这里不太明白）  
```
   't' + 32-byte transaction hash -> transaction index record. These are optional and only exist if 'txindex' is enabled (see above). Each record stores:
       * Which block file number the transaction is stored in.
       * Which offset into that file the block the transaction is part of is stored at.
       * The offset from the start of that block to the position where that transaction itself is stored.
```
以‘t’开头的键的value是交易索引记录  
表示的是这笔交易存在哪个区块链文件号、保存交易的区块的偏移值、交易在区块中偏移值  

### 数据访问层
数据集是通过包装类CBlockTreeDB访问的  
包装类在一个被称作pblcoktree的全局变量中定义的  
CBlockIndex  
存储在数据库中的块在内存中表示为CBlockIndex对象。收到区块头后，首先创建此类型的对象。  
该代码不会等待接收完整的块，通过网络接收到区块头时，会将它们流式传输到CBlockHeaders的向量中，然后对其进行检查。  
每个检出的区块头都将导致创建一个新的CBlockIndex，并将其存储到数据库中。  
CBlock/CBlockHeader  
CBlock将整个交易保存在区块中，交易数据部分放在两个位置。一个是blk*.dat文件，一个是UTXO数据库。  
块索引数据库只保存元数据  
Loading the block database into memory  
整个数据库在启动时就会加载到内存中。详见LoadBlockIndexGuts（txdb.cpp）。  
块（“b”键）被加载到全局“mapBlockIndex”变量中。“mapBlockIndex”是一个unordered_map容器，它为整个块树中的每个块保存CBlockIndex，不只是主动链。  
块文件元数据（“f”键）已加载到vInfoBlockFiles中。  

## UTXO set
UTXO集合也就是链状态leveldb  
具体定义这里不再阐述  
看一下levelDB中关于链状态的键值对：  
```
'c' + 32-byte transaction hash -> unspent transaction output record for that transaction. These records are only present for transactions that have at least one unspent output left. Each record stores:
    * The version of the transaction.
    * Whether the transaction was a coinbase or not.
    * Which height block contains the transaction.
    * Which outputs of that transaction are unspent.
    * The scriptPubKey and amount for those unspent outputs.
'B' -> 32-byte block hash: the block hash up to which the database represents the unspent transaction outputs.
```
'c'+32字节的交易哈希，意思是该交易的未用交易输出记录，这些记录仅适用于至少剩一个未使用输出的交易，包括：  
1、交易的版本  
2、交易是否是一个币基交易  
3、哪一个高度的区块包含了这笔交易  
4、交易的哪一笔输出还没有被花掉  
5、交易的哪些输出被花掉  
6、未使用的输出的scriptPubKey和数量  
'B'+32字节区块哈希，意思是指数据库代表未完成交易输出的块哈希  

### Data Access Layer and Caching
有一点需要注意，在本地检索相关交易时，在UTXO数据库中检索的开销会占大多数，而在检索区块索引的开销会比较小  
如下所示：（但代码实现上看执行不清晰）  
基本数据结构是CCoins（代表单个交易的代币）和CCoinsView（代表代币数据库的状态）。 CCoinsView有几种实现。 一个虚拟对象，一个由硬币数据库（coins.dat）支持，一个由内存池支持，另一个在其顶部添加缓存。  

视图的类型如下：  
```
      CCoinsView (abstract class)
             /            \
         ViewDB          ViewBacked 
      (database)          /      \
                   ViewMempool   ViewCache
```
这里可以看作是将CCoinsViews基类与其子类之间的关系，包括ViewDB，以及ViewBacked，以及子类ViewMempool和ViewCache  
每一个类有一个主属性：  
CCoinsView是基类，其作用是验证硬币是否存在以及对硬币进行检索  
ViewDB具有与LevelDB交互的代码  
ViewBacked有一个指向另外一个View的指针，因此具有UTXO集合的另一个视图  
ViewCache具有一个缓存(CCoins的映射)  
ViewMempool将内存池和视图进行关联  

试图对应的类如下所示：  
```
            Database       
           /       \
       MemPool     Blockchain cache (pcoinsTip) 
     View/Cache            \
                         Validation cache
```

## Raw undo data(rev*.dat)
这些文件包括了如果把某个区块断开或者回滚时所需要的必要信息，实际上是类CTxOut。  

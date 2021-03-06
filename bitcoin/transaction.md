#交易
##交易形式
1. 最常见的交易形式就是一个地址支付到另一个地址，因为比特币中一笔未花费的输出只能被使用一次，所以当输出金额大于交易所需的输入金额时，那么这笔交易的输出还包括向自己的“找零”。即一般的这种交易通常是一个输入两个输出。
2. 集合型交易：这种交易是多个输出向一个地址转账的形式。
3. 分散性交易：跟集合型交易正好相反，它由一个地址向多个地址转账，类似于公司的账户给员工发工资。
##交易的构建
###创建交易输入
交易的输入是之前未花费的输出，之前一直迷惑如何找到合法的未花费的输出？原来一般钱包应用都维护了使用本钱包秘钥锁定的未花费列表的小型数据库。完整的客户端包含所以未消费的输出副本。
####交易输入的结构

| 字段         | 描述                                 | 字节大小 |
| ------------ | ------------------------------------ | -------- |
| 交易         | 指向交易包含的被花费的UTXO的哈希指针 | 32个字节 |
| 输出索引     | 被花费的UTXO的索引号， 第一个是0     | 4字节    |
| 解锁脚本尺寸 | 用字节表示的后面的解锁脚本长度       | 1-9字节  |
| 序列号       | 暂不清楚啥功能                       | 4字节    |

###创建交易输出
交易的输出被创建成为一个包含这笔数额的脚本形式，只有解答了这个脚本，才能获取里面的比特币。交易输出除了支付的数量和找零以外，还包含隐藏的手续费
####交易输出结构
| 字段                      | 描述                         | 字节大小 |
| ------------------------- | ---------------------------- | -------- |
| 交易总量（nValue)         | 用聪表示的比特币值           | 8个字节  |
| 锁定脚本（ scriptPubKey） | 定义其中的比特币该如何取出来 | 边长     |

##交易的结构
一笔交易是包含输入和输出的数据结构：

| 字段     | 描述                       | 大小      |
| -------- | -------------------------- | --------- |
| 版本     | 交易的版本号               | 4字节     |
| 输入数量 | 包含的输入数量             | 1-9个字节 |
| 输入     | 一个或多个输入             | 不定      |
| 输出数量 | 包含的输出数量             | 1-9个字节 |
| 输出     | 一个或多个输出             | 不定      |
| 锁定时间 | 一个unix时间戳或者区块高度 | 4字节     |

> 锁定时间：在比特币代码中的变量为`nLockTime`,它定义了该交易最早可以被加入区块的时间，一般的交易默认值为0，表示立即执行。如果锁定时间大于0小于5亿，就被称为区块高度，意思是在这个区块高度之前，该交易不能加入区块链；如果这个值大于5亿，则它被当做一个unix时间戳（从1970年1月1日以来的秒数），表示在这个时间点之前不能被加入到区块链中。

##交易费
交易费并不并不包含在交易的结构当中，而是通过总的输入减去总的输出计算而来。即：
```
交易费 = 求和(输入) - 求和（输出）
```

如果不是自己通过api构建交易的话，大多数钱包应用都会帮你完成这个动作。还有交易的手续费不是根据你具体教的比特数额来收取的，而是根据交易的大小，假如你通过几百个小额交易来支付一个输出，总输入是0.1btc,但是它的手续费要比你一个是1btc的输入要高得多，因为在前者交易大而复杂。

##脚本
比特币交易的验证通过叫脚本来进行：一个锁定脚本，一个解锁脚本。
标准交易分为5中类型的脚本：P2PKH、P2PK、MS、P2SH、OP_Return
###P2PKH(Pay-to-public-key-hash)
这是最常见的脚本，大多数交易都是此类型。通过名字可以看出，这种脚本是公钥哈希作为锁定的输出，公钥的哈希即是熟知的比特币地址。锁定脚本形式如下：
```
OP_DUP OP_HASH160 <PUBLIC Hash> OP_EQUAL OP_CHECKDSIG
```
对应的解锁脚本形式如下：
```
<sig> <PUBLIC>
```
脚本的解锁：
将解锁脚本和锁定脚本合并组成一个完整的脚本对
```
<sig> <PUBLIC> OP_DUP OP_HASH160 <PUBLIC Hash> OP_EQUAL OP_CHECKDSIG
```
只有解锁脚本和锁定脚本的条件相匹配时，以上脚本的组合测试返回一个true。脚本的验证利用了一个栈结构，整个过程就是执行相应命令数据入栈，出栈验证过程。
###P2PK(PAY-to-public-key)
与P2PKH相比，P2PK更为简单，它直接将对方公钥包含在锁定脚本之中，而且代码长度也更短
P2PK锁定版脚本形式如下：
```
<Public Key A> OP_CHECKSIG
```
用于解锁的脚本是一个简单签名：
```
<Signature from Private Key A>
```
经由交易验证软件确认的组合脚本为：
```
<Signature from Private Key A> <Public Key A> OP_CHECKSIG
```
该脚本只是CHECKSIG操作符的简单调用， 该操作主要是为了验证签名是否正确， 如果正确， 则返回为真（Ture） 
###多重签名
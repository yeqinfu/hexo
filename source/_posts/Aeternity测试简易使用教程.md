---
title: Aeternity测试网简易使用教程
date: 2018-09-05 09:00:58
tags: block chain
---

Aeternity测试网络已经上线许久，这里简单介绍如何在测试网络创建账号，充值，转账，编译智能合约，发布智能合约，调用智能合约的步骤。

### 使用[Python SDK](https://github.com/aeternity/aepp-sdk-python)创建账号

Aeternity 创建账号可以使用Python SDK，JS SDK或者Go SDK。这里举例Python SDK，下载SDK后执行：

![image](http://ws3.sinaimg.cn/large/c1b251b3gy1fwotdkp96qj20su0qm77u.jpg)

可以看到sdk包含的内容有公链数据查询，合约编译，账户区块查询，预言机，钱包创建等相关内容。我们选择钱包创建，执行：

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fwotgn0oidj217m076myj.jpg)

在account后面加上保存密钥文件的路径，加上create命令，输入密码后可以得到如上结果。

### 对生成的账户进行充值

生成的账户其实是不会在链上有记录的，只有执行过交易才会有记录，所以如果发出转账会提示找不到此账户，因为此账户的余额为0.[进入测试网充值](https://faucet.aepps.com/)后，如图所示

![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fwotlmm4zcj222k0qgq74.jpg)

默认充值是5000AE，充值后就会产生交易记录，可以看到这个交易ID，点击跳转到区块链交易浏览器详情，不过似乎还没开发好，看不到详情。我们使用Python SDK查看交易详情,第一次查询是未充值显示找不到账户，第二次是充值后可以查询到相关余额。

![image](http://ws3.sinaimg.cn/large/c1b251b3gy1fwotnpgoozj21bo0eywhl.jpg)

### 测试网转账

我们的账户已经可以看到有余额是5000AE了，下面对此账户进行转账。

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fwots7wfc7j21qk08w0vj.jpg)

指定了我们的密钥路径后，接上spend命令，转账到哪个账户的地址，转账余额。会提示输入密码，此密码就是解密我们密钥的密码输出上图，我们再看一下两个账户的账户余额

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fwottyrwghj21cy0degoc.jpg)

经过我多次转账测试，发现每次转账测试网的手续费收取1AE，这个应该会调整。

### 智能合约编译

Python SDK也提供了编译部署智能合约的功能，不过这里介绍[在线IDE](https://contracts.aepps.com/)进行编译，合约使用的是一个叫Sophia 的语言。

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fuyh8zlv97j21470pogni.jpg)

直接使用测试样例，点击编译后成字节码，再点击下面的部署按钮就可以部署到测试网络了。

![image](http://wx1.sinaimg.cn/large/c1b251b3gy1fuyha19hogj213c0j4405.jpg)

点击部署后会生成一个合约地址，我这里的合约地址为

```
ct$PzpYsHcA858UNDJLnfmQznnNKPwxmatXwFrWG6GjNhr1rScgm
```

### 使用区块链浏览器进行查看合约和交易

![image](http://wx3.sinaimg.cn/large/c1b251b3gy1fuyhfg87drj215g0hlgqf.jpg)

此浏览器可以查看我们以上执行动作的过程，包括充值交易，转账交易，合约创建。SDK并没有提供查询合约的方法，浏览器也不是很完善，所以刚才创建的合约我就不去找了。

### 智能合约调用

![image](http://ws3.sinaimg.cn/large/c1b251b3gy1fuyhj1caa0j21290ct3yx.jpg)

这里使用IDE调用，传入方法名字和参数就可以了，更加复杂的合约例子可以看github的例子。

### 在线IDE更改账户

![image](http://ws2.sinaimg.cn/large/c1b251b3gy1fuyhkiz46lj213008g0t8.jpg)

如果需要更改账户的话，在线IDE的头部可以输入你的账户的密钥，此外，这个密钥是一个十六进制的字符串，用Python SDK生成的是一个已经加密的的密钥文件，所以我们可以在

![image](http://wx4.sinaimg.cn/large/c1b251b3gy1fuyhm96wd5j20yw0hswgm.jpg)

如上图的位置增加一个输出，得到真正的私钥，后保存下来使用。



### 还需要改进的地方

+ 区块链浏览器 不能根据交易ID查询，根据合约ID查询，交易详情不够完善，比如gas的使用量，交易费用等。
+ 和以太坊IDE相比的话，缺少Debugger功能，还不够完善
+ Python SDK缺少查询合约的功能



#### 相关地址

python sdk地址：https://github.com/aeternity/aepp-sdk-python

测试网充值地址：https://faucet.aepps.com/

在线合约IDE：https://contracts.aepps.com/

























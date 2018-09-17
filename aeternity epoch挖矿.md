---
title: epoch挖矿简易教程
date: 2018-09-12 09:00:58
tags: block chain
---



### 准备工作

ubuntu16.04安装erlang环境

```
sudo apt-get install erlang
```

下载epoch release包，并解压

#### 生成 beneficiay(挖矿受益者)账户

解压后重命名文件夹为node，在node子文件夹bin中有一个名为epoch的shell脚本执行命令。不过，在生成账户之前需要安装密钥支持库的相关依赖

```
sudo apt-get install build-essential
```

```
wget https://download.libsodium.org/libsodium/releases/libsodium-1.0.16.tar.gz
tar -xf libsodium-1.0.16.tar.gz && cd libsodium-1.0.16
./configure && make && sudo make install && sudo ldconfig
```

以上的依赖完成后，可以利用node的子目录bin/epoch生成账户

```
./bin/epoch keys_gen PASSWORD
```

后面的PASSWORD是你自己定义的密钥，此密钥目的是给生成的公钥私钥对的私钥进行对称加密，每次使用账户的时候需要用你输入的密钥进行对私钥解密。

![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fv6sak7yntj20pn01pq39.jpg)

生成后会在当前目录下生成新的文件夹generated_keys里面包含新生成的文件夹公钥私钥

#### 配置beneficiay账户

在node文件夹下创建一个epoch.yaml文件写入以下配置信息，并且更改配置账户字段为刚才生成的账户公钥

```
---
sync:
    port: 3115
    external_port: 3015

keys:
    dir: keys
    password: "secret"

http:
    external:
        port: 3013
    internal:
        port: 3113

websocket:
    internal:
        port: 3114
    channel:
        port: 3014

mining:
    beneficiary: "beneficiary_pubkey_to_be_replaced"
    autostart: true

chain:
    persist: true
    db_path: ./my_db
```



利用epoch脚本检查配置是否有效

```
./bin/epoch check_config epoch.yaml
```



![image](http://wx3.sinaimg.cn/large/c1b251b3gy1fv6snbmbljj20i2028jro.jpg)



启动epoch

![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fv6sowxxzsj210o01y3yz.jpg)



会提示需要修改最大文件打开数量根据提示修改即可

> ubuntu18.04 可能无法修改此项，16版本则可以。可能是硬件设备在不同版本受限不同。
>
> 我同样是虚拟机，18不行16则可以

执行

```
ulimit -n 50000
```

再次启动

```
./bin/epoch start
```

#### 查看挖矿日志

启动后，node目录下会生成log/*的若干文件，可以尝试搜索相关关键字来查看是否挖到新的块

```
grep "mined" log/epoch_mining.log
```

![image](http://ws3.sinaimg.cn/large/c1b251b3gy1fv7lx9nnhqj219q02smyb.jpg)

当结果有值时，代表已经挖到新的块了

#### 可能出现的错误

检查log/epoch_mining.log有类似出现这些内容

```
 Failed to mine block, runtime error; retrying with different nonce (was 13078180597498667023). Error: {execution_failed,{signal,sigabrt,true}}
```

更改配置文件epoch.yaml

```
mining:
    autostart: true
```

改为

```
mining:
    autostart: true
    cuckoo:
        miner:
            executable: lean30
            extra_args: ""
            node_bits: 30
```

然后执行`bin/epoch stop; bin/epoch start;`重启



![image](http://ws3.sinaimg.cn/large/c1b251b3gy1fv6ty3zoaij20s203n755.jpg)

如果log/epoch_mining.log最后显示starting mining说明正在挖矿



#### 验证你的节点是否和测试网同步

```
curl http://31.13.249.70:3013/v2/blocks/top
curl http://127.0.0.1:3013/v2/blocks/top
```

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fv6u5cmwjcj21h805swi5.jpg)

比较哈希值是否一致



#### 注意事项

epoch不同版本都不是向前兼容的，所以当我们看到已经挖到新的矿，想查询自己的beneficiay账户是否得到相应的奖励。可以在epoch启动的情况下执行

```
curl http://localhost:3013/v2/accounts/ak%242ibwhmwkiQpUjtAQAfEQ139CqtknJFhkpzgtqtF791dQAF3L4m
```

把账户公钥改成自己的公钥，如果你拿着自己的账户用python sdk查询可能会查不到账户余额，原因是本地的epoch版本和python sdk调用的epoch版本不一致，简单说就是不同版本在不同的链上。

epoch启动后可以根据[epoch api](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/) 中展示的接口去查询所需要的信息，例如查询自己挖矿链上的最高的一个块数据

![image](http://ws2.sinaimg.cn/large/c1b251b3gy1fv824pwf54j21ar0j8gn0.jpg)



把host部分替换成 ``` localhost:3013``` 在本地请求就可以了，更多请求接口信息可以参考[这个项目](https://github.com/aeternity/epoch-api-docs) 。官方回复我epoch尚未暴露swagger 接口部分，可能后续会改进，就不用那么麻烦了。

#### 备注

epoch 项目地址：https://github.com/aeternity/epoch

epoch-api-doc地址：https://github.com/aeternity/epoch-api-docs

####  



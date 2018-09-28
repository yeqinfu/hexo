---
title: docker方式启动epoch
date: 2018-09-26 09:00:58
tags: block chain
---

这里介绍两种环境ubuntu，win7以docker方式启动epoch

### WIN7使用docker 启动epoch

win7  安装[DockerToolbox](https://www.cnblogs.com/canger/p/9028723.html)的步骤这里略过，可以自己百度

> docker是不支持win7的，这里使用DockerToolbox 安装



![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fvn28vpl82j20jh0em75z.jpg)





docker安装完之后可以看到如上图

获取镜像

```
docker pull aeternity/epoch
```



![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fvn2nb53r2j20gr05p74d.jpg)



下载完成后，可以查看到镜像

#### 生成账户

```
docker run --entrypoint=/bin/bash \
    -v /c/Users/generated_keys:/home/epoch/node/generated_keys \
    aeternity/epoch \
    -c './bin/epoch keys_gen my_password'
```

![image](http://ws2.sinaimg.cn/large/c1b251b3gy1fvp0cy2etkj20hl04u3yj.jpg)

![image](http://wx4.sinaimg.cn/large/c1b251b3gy1fvp0d929auj20mb09cgmf.jpg)



> 注意我们这个步骤是挂载主机目录到运行的容器，但是因为当前系统是win7 这里挂载目录需要绝对路径，并且要配置了共享文件目录才能挂载成功，这里默认使用当前已经共享的Users目录来保存生成的公钥私钥，如果是其他环境可能就不需要这么麻烦了

公钥

```
  ak_2GCsQFhmwQ5UPnRFPfpJgTig1yLQU3qEAcyu6JMtscobAPqrpg
```

#### 写配置文件

和上个教程一样，现在写一个配置文件把公钥写进去，配置文件的格式如下

```
---
sync:
    port: 3115
    external_port: 3015

keys:
    dir: keys
    password: "my_password"

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
    beneficiary: "ak_2GCsQFhmwQ5UPnRFPfpJgTig1yLQU3qEAcyu6JMtscobAPqrpg"
    autostart: true

chain:
    persist: true
    db_path: ./my_db
```

把当前的配置文件也放在Users目录下，命名myepoch.yaml

#### 启动docker

这里新建一个容器，把配置文件挂载到容器

```
docker run -d -p 3013:3013 \
    -v /c/Users/myepoch.yaml:/home/epoch/myepoch.yaml \
    -e EPOCH_CONFIG=/home/epoch/myepoch.yaml \
    aeternity/epoch -aehttp enable_debug_endpoints true
```

![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fvp0ldx0ggj20ii08gjro.jpg)

可以看到容器已经在运行了。我们进入交互模式，查看挖矿日志

```
 docker exec -it da55 /bin/bash
```

> 把da55改为你生成的容器id

```
grep "mining" log/epoch_mining.log
```

发现报错

![image](http://wx3.sinaimg.cn/large/c1b251b3gy1fvp0ps96ykj20iq04e0sw.jpg)

> 如果没有报错就无需下一步了

这里更改配置文件

```
mining:
    beneficiary: "ak_2GCsQFhmwQ5UPnRFPfpJgTig1yLQU3qEAcyu6JMtscobAPqrpg"
    autostart: true
    cuckoo:
        miner:
            executable: lean30
            extra_args: ""
            node_bits: 30
```

然后重启一下容器就可以了，后续省略。



### Ubuntu docker 启动epoch

```
sudo docker pull aeternity/epoch
```

![image](http://wx3.sinaimg.cn/large/c1b251b3gy1fvnw5rjc83j20kj086abo.jpg)

> 如果需要添加当前用户到docker用户组
>
> sudo groupadd docker     #添加docker用户组
> sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
> newgrp docker     #更新用户组ps 
> docker ps    #测试docker命令是否可以使用sudo正常使用



```
 docker run --entrypoint=/bin/bash \
 -v /home/yeqinfu/aeternity/test:/home/epoch/node/generated_keys  \
 aeternity/epoch   \
 -c './bin/epoch keys_gen my_password'
```

![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fvp1enozkaj21a702bgm0.jpg)



![image](http://wx3.sinaimg.cn/large/c1b251b3gy1fvp1f3h8pxj20gd085aa9.jpg)

> 如果挂载不成功，先在主机目录创建文件夹，比如/home/yeqinfu/aeternity/test这个文件夹先创建起来再执行

#### 写配置文件

同上面一样，我们在test目录写一个配置文件，把公钥填写进去

#### 启动epoch



![image](http://wx3.sinaimg.cn/large/c1b251b3gy1fvp1jyo3ipj21ga02yq3h.jpg)



#### 进入容器查看挖矿日志

![image](http://wx1.sinaimg.cn/large/c1b251b3gy1fvp1lp7lohj20q706adh8.jpg)



后续的步骤雷同。

[官方文档移步这里](https://github.com/aeternity/epoch/blob/master/docs/docker.md)






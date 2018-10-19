---
title: CUDA Miner
date: 2018-10-12 14:00:58
tags: code
---

### 下载CUDA工具

```
cd ~
wget https://developer.nvidia.com/compute/cuda/9.2/Prod/local_installers/cuda-repo-ubuntu1604-9-2-local_9.2.88-1_amd64
sudo dpkg -i cuda-repo-ubuntu1604-9-2-local_9.2.88-1_amd64
sudo apt-key add /var/cuda-repo-9-2-local/7fa2af80.pub
```

第一个文件比较大，大概1G多，命令窗下载会比较大，可能会断了，可以选择拷贝后面的链接部分用浏览器下载，再执行第二个命令解压。

![image](http://ws2.sinaimg.cn/large/c1b251b3gy1fwdlto2jz3j20py0790ue.jpg)

安装

```
sudo apt-get update && sudo apt-get install cuda
```

过程可能有点久，安装完成之后可以在

``` 
/usr/local/
```

目录下看到安装的文件，把此文件下的可执行文件夹加入到环境变量中。此步骤主要是为了安装目录下`` nvcc`` 这个可执行脚本在编译cuda30需要用到。

```
export PATH=/usr/local/cuda-9.2/bin${PATH:+:${PATH}}
```

中间的cuda-9.2以你下载的驱动文件型号为转移。可以把此命令加入到开机启动的bashrc文件中。

### 编译epoch

```
cd ~
git clone https://github.com/aeternity/epoch.git epoch && cd epoch
git checkout tags/v0.19.0
```

下载epoch源码，这里提示说必须下载官方源码，而不能只下载release包，因为release包不包含cuda30，所以之前的动作都是为编译做准备。

```
cd apps/aecuckoo && make c_src/.git
cd c_src/src && make libblake2b.so cuda30
cp cuda30 ~/node/lib/aecuckoo-0.1.0/priv/bin
```

![image](http://wx1.sinaimg.cn/large/c1b251b3gy1fwdnsdpi5aj21960933za.jpg)

进入这个目录后编译产生的文件cuda30，把它拷贝到我们之前解压的release包node目录下```node/lib/aecuckoo-0.1.0/priv/bin``

### 更改配置文件

```
mining:
    cuckoo:
        miner:
            executable: cuda30
            extra_args: ""
            node_bits: 30
            hex_encoded_header: true
```

拷贝成功后，在配置文件中就可以把原来的lean30改成cuda30.然后重启epoch就可以了

查看GPU使用情况

```
nvidia-smi
```



### 注意

+ 所有流程都是在ubuntu电脑，而非虚拟机环境下进行的，我不清楚如何让虚拟机使用宿主机显卡

+ 编译cuda30的代码默认是7GB，你的显卡至少要4GB，如果是4GB需要更改代码重新编译，更改的代码如图

  ![image](http://wx3.sinaimg.cn/large/c1b251b3gy1fwdnvz8a56j20uc0fdq4q.jpg)

  这个mean_nimer.cu中menGB更改为4

+ 当使用显卡挖矿时，CPU挖矿就停止了，不能并行挖矿
































































































---
title: Grin 挖矿
date: 2019-01-09 10:00:58
tags: block-chain
---

### Grin 挖矿

#### 环境

+ ubuntu16
+ N卡驱动410
+ cudatoolkit10

#### 安装节点

+ 安装1.31+版本RUST

  ```
  curl https://sh.rustup.rs -sSf | sh; source $HOME/.cargo/env
  ```

+ 安装Grin Node所需依赖软件包

  ```
  sudo apt install build-essential cmake git libgit2-dev clang libncurses5-dev libncursesw5-dev zlib1g-dev pkg-config libssl-dev llvm
  ```

+ 安装Grin node.

  ```
  git clone https://github.com/mimblewimble/grin.git
  cd grin
  cargo build --release
  ```

+ 添加grin命令环境变量

  ```
  export PATH=/<yourPath>/grin/target/release:$PATH      
  ```

  > build 之后的产物，在grin/target/release里面，把里面的可执行文件添加到全局变量

+ 添加grin-server.toml文件到grin文件夹

  ```
  grin —floonet server config
  ```

  > grin文件夹下执行上诉命令，会生成配置文件，名字就是grin-server.toml

+ 修改生成的配置文件

  ```
  enable_stratum_server = true
  ```

  > 找打这行改成true

+ 启动节点

  ```
  grin —floonet
  ```

  > 后面的参数表示测试网，主网上线后，就不加这个参数启动节点

![image](https://ws1.sinaimg.cn/large/c1b251b3gy1fz2peq3eq1j20k10boq38.jpg)

> 上图为节点显示信息，这个命令窗不要关闭，保持节点运行

#### 钱包创建

+ 另起一个窗口，钱包初始化

  ```
  grin --floonet wallet init 
  ```

  > 注意另起窗口后，要确认grin是否像刚才上一步那样加到全局变量
  >
  > 生成的助记词要保存下来，用来钱包恢复执行此命令后在~/.grin目录下有相应的产物，包括钱包种子子类的东西

+ 钱包监听

  ```
  grin --floonet wallet listen
  ```

  > 钱包监听是负责挖矿到账的监听，所以这个窗口也不能关闭

#### 旷工创建

+ 再启动一个窗口，下载旷工软件依赖

  ```
  sudo apt install make libncurses5-dev libncursesw5  zlib1g-dev linux-headers-$(uname -r)
  ```

+ 下载旷工程序

  ```
  git clone https://github.com/mimblewimble/grin-miner.git
  cd grin-miner
  ```

+ 挖矿方式配置，默认是cpu挖矿，这里要改成一下配置文件，构建的时候才有cuda的解码器,更改构建文件Cargo.toml

  ```
  change:
    cuckoo_miner = { path = "./cuckoo-miner" }
    to:
    cuckoo_miner = { path = "./cuckoo-miner", features = ["build-cuda-plugins"]}
  ```

  找不到没关系，直接执行下面这个命令去更改

  ```
  sed -i 's/^\(cuckoo_miner.*\)}/\1, features = ["build-cuda-plugins"] }/' Cargo.toml
  ```

+ Grin miner 开始构建

  ```
  git submodule update --init
  cargo build 
  ```

  > 构建完成的产物同样在grin-miner文件夹下的target文件夹下
  >
  > 最后一个如果是cargo build --release那就是打release包，在target/release下

+ 把产物的命令文件加入全局变量

  ```
  export PATH=/<yourPath>/grin-miner/target/debug:$PATH
  ```

+ 解码器插件在target/debug/plugin下面，然后更改旷工配置文件，让它使用cuda plugin

  ```
  sed -i 's/^plugin_name =.*/plugin_name = "cuckaroo_cuda_29"/' grin-miner.toml
  ```

  > 直接执行上面的sed命令更改grin-miner.toml 文件，或者自己打开grin-miner.toml去更改

+ 启动旷工

  ```
  grin-miner//这个可执行文件在产物文件夹下，加入全局变量后，父目录执行
  ```

+ 然后看到旷工运行界面

  ![image](https://ws2.sinaimg.cn/large/c1b251b3gy1fz2q2ivb9fj210d0aodgf.jpg)

可以看到运行的插件情况

![image](https://ws2.sinaimg.cn/large/c1b251b3gy1fz2q3m78ioj20kb06waae.jpg)

然后让这个保持运行

#### 查看钱包运行情况

+ 另起一个窗口

```
grin —floonet wellet info
```

![image](https://wx4.sinaimg.cn/large/c1b251b3gy1fz2q78f14zj20hf04z74k.jpg)




































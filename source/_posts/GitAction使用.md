---
title: GithubAction 使用
date: 2019-11-19 09:00:00
tags: code
---

之前静态博客源码一个仓库，生成的github page在另一个仓库，并且[使用Travis自动部署]([http://www.yeqinfu.com/2017/02/06/%E5%8D%9A%E5%AE%A2TravisCI%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2Hexo/](http://www.yeqinfu.com/2017/02/06/博客TravisCI自动部署Hexo/))。

现在使用github action替代使用。

代码大概也类似

### 新建源码仓库

我把源码仓库（名字暂时叫hexo）设置为私有仓库，不对外开放，因为Action里面会填写token，然后静态网页代码仓库（xxxx.github.io）也就是网站主目录放在另外一个仓库

所以hexo仓库大概长这样

![image](https://tvax1.sinaimg.cn/large/c1b251b3gy1g93dtj1vg6j210p0cx0u2.jpg)

xxx.github.io仓库长这样

![image](https://tva2.sinaimg.cn/large/c1b251b3gy1g93dux4ygbj211y0c375g.jpg)

### 本地搭建Hexo

这个就略过了，官网教程执行就可以了。

### 新增Action

我们在hexo的仓库新增一个action

代码如下

```
name: Build and Deploy

on: [push]

jobs:

  build-and-deploy:

    runs-on: ubuntu-latest

    strategy:

      matrix:

        node-version: [10.x]

    steps:

      - uses: actions/checkout@v1



      - name: Use Node.js ${{ matrix.node-version }}

        uses: actions/setup-node@v1

        with:

          node-version: ${{ matrix.node-version }}



      - name: Configuration environment

        run: |

          git config --global user.name "xxx"

          git config --global user.email "xxxx@email.com"



      - name: Install dependencies

        run: |

          npm i -g hexo-cli

          npm i



      - name: Deploy hexo

        run: |

          hexo g

      - name: push to github.io

        env:

          GIT_TOKEN: replace_yours

          GH_REF: github.com/xxx/xxx.github.io.git

        run: |

          cd ./public

          git init

          git add .

          git commit -m "update docs"

          git push --force --quiet "https://${GIT_TOKEN}@${GH_REF}" master:master
```



可以看个大概，第一行脚本名字

```
on: [push]
```

触发条件是每次这个hexo仓库被push的时候，就会触发

然后可以看到很多任务被执行

+ 检出这个仓库代码
+ 配置nodejs环境

+ 配置git环境
+ 配置hexo环境
+ hexo g的生成命令
+ 生成后我们知道多出一个public文件夹
+ 然后进入public文件夹对这个文件夹内容进行git初始化
+ 然后强制推到我们指定的github pager的地址

注意，这里需要自己去获取git_token，生成后填写进脚本，不能暴露，因为这个token有提交代码权限，所以这里把源码项目设置为私库。


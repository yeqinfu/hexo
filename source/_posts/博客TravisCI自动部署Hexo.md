---

title: TravisCI自动部署Hexo

date: 2017-02-06 01:39:34

tags:
---

用hexo搭建静态博客需要配置hexo，nodejs环境，如果换了电脑，岂不是很麻烦。
自己博客源码放在一个名为hexo的仓库，生成的静态博客代码在yeqinfu.github.io仓库。
Travis CI 可以监测源码仓库的代码提交，在线生成静态博客代码，并自动提交到yeqinfu.github.io
这个仓库。以下叙述配置过程
登陆[Travis CI 官网]( https://travis-ci.org/)
添加源码仓库

![图1](/imgs/a1.png)

 这个仓库将会被监听提交，在这个项目中添加一个配置文件，主要是在线构建的命令
```
language: node_js
node_js: stable

# S: Build Lifecycle
install:
  - npm install
#before_script:
 # - npm install -g gulp
script:
  - hexo g 
after_script:
  - cd ./public
  - git init
  - git config user.name "yeqinfu"
  - git config user.email "990761790@qq.com"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GIT_TOKEN}@${GH_REF}" master:master
# E: Build LifeCycle
branches:
  only:
    - master
env:
 global:
   - GH_REF: github.com/yeqinfu/yeqinfu.github.io.git

```

可以看到当仓库有变动的时候，各个执行命令。最后被提交到GH_REF这个仓库中。
这个仓库配置为yeqinfu.github.io
提交权限需要一个GIT_TOKEN，在github中
![图2](/imgs/a2.png)

 添加即可。拿到token添加到Travis CI的配置面板
![图3](/imgs/a3.png)

 
就完成了配置。所有的提交都会被构建后提交到另一个库。
[参考地址](http://i.woblog.cn/2016/05/04/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E4%BD%BF%E7%94%A8Travis%20CI%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E4%BD%A0%E7%9A%84Hexo%E5%8D%9A%E5%AE%A2%E5%88%B0Github%E4%B8%8A/)































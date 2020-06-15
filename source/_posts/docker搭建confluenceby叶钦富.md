---
title: docker搭建confluence
date: 2020-03-07 09:00:00
tags: 运维
---

## 镜像拉去

>  docker pull atlassian/confluence-server

## 运行实例

> ```
> docker run -v /data/your-confluence-home:/var/atlassian/application-data/confluence --name="confluence" -d -p 30166:8090 -p 30167:8091 atlassian/confluence-server
> ```

Docker的命令看出，这里指定了一个数据卷，还有两个端口映射。因为测试环境好像只开放了三万以上的端口

## 驱动配置

confluence的数据库打算使用mysql，这里也是打算使用一个实例跑mysql。官方镜像没有mysql的驱动的。这里选择了mysql8.0

> 刚开始使用5.7,但是5.7要改utf-8mb4格式，这个格式好像是长度会长一点不会乱码可以存emoji。
>
> 后面又改成8.0了，但是8.0要改身份验证机制

[驱动下载地址](https://mvnrepository.com/artifact/mysql/mysql-connector-java?__cf_chl_jschl_tk__=0f6ac7c4166a3dddc068ffcc493c1336ce2888cf-1584155550-0-ASWUhoeavLLSgIdIstoEejsx5qtw-caG_9luQWcNgSKm9x3rfrg0SEbA-FjVnQV7CDaAWry49crproTwCIZk186xFYaXLAWWtD62SDHE7olrsLaeeyDrVL-2bj0OYZUlpWaj8etwoiTNFFzdIlatVP2ASwLWPtw2_nofucWf2Rv5dxTjeFueO7AT2pKtO1yaXN2TpdDWkA3rY4j-oH_1z03PJZjddP0LNmOcB5t5Az5noXD0vFiyF9hExtO6Phk7H5mfB_bmFSszGpfsxTB2zokcGmIKRmk4FxHV4fya3_hzxRyVZfCRKm5JfjHubObLNfsDWtQ0k1jc81uaVUYgCN_lR9cRxPzGKtDyG3m2CvxIR-tT7RpxUMdOuJHD_ELmgw)

把从maven仓库下载的驱动上传到宿主机的某个目录后，进行一个宿主机到容器的文件拷贝

> 因为有挂载数据卷，放在数据卷然后进入容器bash拷贝也行，我这里没有放在卷目录下

执行拷贝命令

> docker cp /home/dev/xmstd/confluencejar/mysql-connector-java-8.0.19.jar confluence:/opt/atlassian/confluence/lib

使用的驱动版本是8.0.19,拷贝目录如上所示。然后重启实例

>  docker restart confluence

## 运行mysql实例

> docker pull mysql

我们再跑一个mysql的实例

> docker run --name confluence_mysql -p 30206:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql

指定了密码和数据库端口映射

然后因为confluence还要指定数据库，那我这里先进bash创建一个数据库吧

> 最后因为要测试连接，直接用sqlyog创建了一个数据库
>
> 创建数据库的编码指定为utf8mb4,数据库排序为utf8mb4bin

mysql 8.0 默认使用 caching_sha2_password 身份验证机制 —— 从原来的 mysql_native_password 更改为 caching_sha2_password

> ALTER  USER  'root'  IDENTIFIED  WITH  mysql_native_password  BY  '123456'



## 启动confluence

> http://218.6.70.66:30166/

这是地址。然后傻瓜式一顿操作

这边记住一个serverID，因为可能后续破解confluence需要

> B784-FT4O-C8SB-AOZO

然后官网获取key

> 接下来的一顿操作，网上就可以百度到了

然后一顿操作到了数据库配置环节。这里使用链接字符串，因为要指定一些编码格式，防止中文乱码

> jdbc:mysql://218.6.70.66:30206/confluencedb?sessionVariables=transaction_isolation='READ-COMMITTED'

> 这里注意一下后缀，不同版本的mysql设置数据库的隔离级别是不一样的

[数据库隔离级别设置参考地址](https://confluence.atlassian.com/confkb/confluence-fails-to-start-and-throws-mysql-session-isolation-level-repeatable-read-is-no-longer-supported-error-241568536.html)

## 数据库连接之后

数据库连接完成之后，选择新建一个空网站，然后配置一下管理员












































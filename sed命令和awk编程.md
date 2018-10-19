---
title: sed命令和awk编程
date: 2018-09-30 14:00:58
tags: code
---



### sed -n

```
sed -n '1p' test
```

-n命令代表不打印test文件的所有行，这样上面命令就会打印第一行，而如果去掉-n会打印第一样和全部

```
sed -n '1,3p' test
```

打印1到3行

```
sed -n '/expressions/p' test
```

匹配模式，打印包含指定关键字``` expressions``` 的行

### sed -e

```
sed -n -e '/th/p' -e '/th/=' test	
```

多个命令时候-e才能发挥作用，上面代表打印符合行内容和行号

### sed -f


























































---
title: Sophia例子
date: 2018-12-26 09:00:58
tags: block chain
---



#### 常用名字系统

* Contract

  ```
  Contract.creator
  Contract.address
  Contract.balance//Chain.get_balance(Contract.address)
  ```

* Call

  ```
  Call.origin //调用者地址
  Call.caller //可能是调用者地址，或者合约地址
  Call.value //调用的时候转到合约的代币数量
  Call.gas_price
  Call.gas_left()//调用者实际调用后剩余的gas数量
  ```

* Chain

  ```
  Chain.get_balance(a : address)
  Chain.block_hash(h)
  Chain.block_height
  Chain.coinbase
  Chain.timestamp
  Chain.difficulty
  Chain.gas_limit
  ```


#### Evnts事件

```

```














































### Erlang入门

#### 给并发建模

erlang每个文件可以类似person.erl,dog.erl每个文件就是一个模块,例如aec_accounts.erl

```erlang
-module(aec_accounts).

%% API
-export([new/2,
         id/1,
         pubkey/1,
         balance/1,
         nonce/1,
         earn/2,
         spend/3,
         spend_without_nonce_bump/2,
         set_nonce/2,
         serialize/1,
         deserialize/2,
         serialize_for_client/1]).
```

模块名必须一个小写字母开头，第一行声明模块名字，和文件名一致。export后面衔接的名字，相当于暴露给外部模块的方法，类似于java的public方法。

比如，第一个new方法，后面加了/2代表，暴露的方法有两个参数

```erlang
new(Pubkey, Balance) ->
    Id = aec_id:create(account, Pubkey),
    #account{id = Id, balance = Balance}.
```






































---
title: Sophia基本语法
date: 2018-12-26 09:00:58
tags: code
---

### Sophia基本语法

* 这是一个强类型的语言

* Sophia没有模块，相反有静态合约。

* 每个合约实例都存在于公链或者状态通道中

* 合约可以通过 public 关键字声明API供外部调用，外部合约就可以调用它，

* 合约可以定义本地状态，这个状态必须在合约发布的时候被初始化，在一个叫 init 的初始化函数中，如下

  ```
  type state = { value : int }
  function init(val)       = { value = val }
  stateful function tick() = put(state{ value = state.value + 1 })
  ```

  声明本地状态的变量必须被初始化，并且要更改状态的函数必须声明 stateful 关键字。

* 合约可以继承另一个合约（可以多继承？），继承后，父类合约的 public 方法就会被包含到子类合约中，本地状态也一并继承，但是父类合约的私有方法不能被子类合约调用。


#### 合约基本变量

通过let绑定变量

```
let greeting = "Hello"
let score = 10
let newScore = 10 + score
let condition = false
```

如果要更改变量的值

```
let message = "Hello" // message has value "Hello"
let message = "Bye" // message has value "Bye"
message = "Hola" // Error
```

必须要加上 let  才能绑定新的值。这里还有一个，如下

```
function main(x : int) = 
    let score = 1
    let score = "ddd"
    score
```

score  这个变量通过 let 赋值，竟然可以从 int 类型变成 string 类型。

```
let score: int = 10
```

变量赋值的时候可以指定为 int 类型

```
 function main(x : int) = 
    let score:int = 1
    let score:string = "ddd"
    score
```

变量通过匿名函数赋值

```
let myInt = (5: int) + (10: int)
let add = (x: int, y: int) : int => x + y  // this is a function definition 
```

如上， add 这个变量，通过这个匿名函数赋值的。但是还不知道这个操作的意义在那里。



#### 变量类型声明

```
type intCoordinates = (int, int, int);
let buddy: intCoordinates = (10, 20, 20);
```

一个变量的类型除了是基本类型，还可以是多类型，这边自定义的类型，名字叫 intCoordinates 它是三个int 类型组成的，所以 buddy 在赋值的时候必须传三个整形。多类型还可以缩写

```
type coordinates('a) = ('a, 'a, 'a)
let buddy: coordinates(int) = (10,20,30)
```

当然也可以这样

```
  type coordinates('a) = ('a, 'a, 'a)
  type pp('a,'b) = ('a,'a,'b,'b)
  function main(x : int) = 
    let score:int = 1
    let score:string = "ddd"
    let buddy: coordinates(int) = (10,20,30)
    let my: pp(int,string) = (1,1,"aa","bb")
    score
```

#### List类型

 list 类型里面应该只能是同一种类型，比如

```
function main(x : int) = 
    let xs = [1,2,3,4]
    let xs1 = ["a","b"]
    let xs2 = ["a",1] //error
    1
```

往list 添加元素

```
 function main(x : int) = 
    let xs = [1,2,3,4]
    let xs1 = [2,6]
    let xs2 = 2::xs1
    1
```

但是似乎不能通过这个连接符去连接两个list

#### Tuple 类型

如果需要在集合放置不同类型的元素，可以用tuple 

```
let ageAndName = (24, "Lil' Language")
let my3dCoordinates = (20.0, 30.5, 100.0)
```

还可以这样赋值

```
let ageAndName: (int, string) = (24, "Lil' Language")
/* a tuple type alias */
type coord3d = (float, float, float)
let my3dCoordinates: coord3d = (20.0, 30.5, 100.0)
```

```
let my3dCoordinates = (20.0, 30.5, 100.0)
let (x, y, z) = my3dCoordinates /* now you've retrieved x, y, z */
let (_, y, _) = my3dCoordinates /* now you've retrieved y */
```

下划线是占位符，代表不需要赋值

#### Record 类型

这种类型类似于C语言的结构体 但是关键字是通过record来声明

```
record account = { name    : string,
                     balance : int }
```

以上是Record定义，声明之后就可以赋值了，me这个变量在赋值的时候，会自动知道你赋值的是一个account的record类型

```
record account = { name    : string,
                     balance : int }
  function main(x : int) = 
    let me={name="haha",balance=100}
    1
```

内部变量的访问

```
let name = me.name
```

但是，如果你没有声明这个record编译就不会通过，比如下面这样编译不通过，因为没有声明record

```
contract Identity =
  type state = ()
  function main(x : int) = 
    let me={name="haha",balance=100}
    let name=me.name
    1
```



#### map类型

```
contract Identity =
  type state = () 
  type accounts = map(string, address)
  function main()=
    let all:accounts={}
    all["ddd"]
  function get_balance(a : address, hah : map(address, int)) =
    hah[a]
```

如上例子，声明了一个type为accounts的类型，取值的时候直接 all["ddd"] 取值就可以。



#### State

其实官方教程很多都编译不过去，很多应该都是错的。state这个特殊的变量，合约的状态应该都放在state这个名字的变量中，注意state的类型是一个record类型，关键合约的变量都应该放在state里面

```
contract Identity =

  record state = { start_amount : int,
                 start_height : int,
                 dec          : int,
                 beneficiary  : address,
                 sold         : bool }
  function main(x : int) = x
  public function init(beneficiary:address, start:int, decrease:int) : state =
    { start_amount = start,
      start_height = Chain.block_height,
      beneficiary  = beneficiary,
      dec          = decrease,
      sold         = false }
```

当我们声明了这个state变量的时候，还必须实现init函数，不然会报错，提示需要init赋值这个state

当我们队state这个变量进行了一定的定义和初始化之后，在合约的全局中就可以对state这个record类型的变量进行一些隐式赋值，当我们想更改state的内容，那么相关的方法必须声明 stateful 关键字

```
contract Identity =

  record state = { start_amount : int,
                 start_height : int,
                 dec          : int,
                 beneficiary  : address,
                 sold         : bool }
 
  public function init(beneficiary:address, start:int, decrease:int) : state =
    { start_amount = start,
      start_height = Chain.block_height,
      beneficiary  = beneficiary,
      dec          = decrease,
      sold         = false }
  public stateful function main(test:int) =
    put(state{start_amount=test})
    1
```

这里做一个简单的测试

```
contract Identity =

  record state = { amount : int }
 
  public function init(x:int) : state =
    { amount = x }
  public stateful function main(test:int) =
    put(state{amount=test})
    state.amount
  public function main2(test:int)=
    put(state{amount=test})
    state.amount
```

如上代码，我们初始化amount为100，两个函数都对state进行赋值，但是一个有stateful的声明，这里对两个函数进行调用，发现都可以改state的值，我也不知道为什么，我觉得main2应该编译期就报错才对。

#### 条件语句

```
if(exp1) exp2 else exp3
let y = if(5>4) true else false
```

第二种有点像kotlin哈哈

```
switch (expr0) 
        pattern1 => expr1
        pattern2 => expr2
        _    => expr3
```

最后一个代表默认走的逻辑

#### 函数

```
  public stateful function main(test:int) =
    put(state{amount=test})
    state.amount
```

如上等号后面就是一个函数的体，函数体有严格的作用域范围，函数体必须要有缩进，推进是两个空格，当然一个空格也不报错，但是之后必须是都一个空格，最后一行是返回值。

```
(x) => x + 1
```

匿名函数，以上

#### 异常抛出

```
abort(reason : string) : 'a
```

```
abort("balance not sufficient")
```

直接输入异常内容后抛出就可以了，异常函数a 应该是一个类似于泛型的意思














































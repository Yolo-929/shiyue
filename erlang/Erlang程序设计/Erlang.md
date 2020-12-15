## 学习路线

1. 并发编程的概念
2. 顺序编程
3. 核心（并发，分布式）
4. Erlang库、跟踪调试，组织
5. 应用程序，外部软件与核心库的集成

---

## 并发编程

### 并发demo

- 并发建模

  ```erlang
  -module(person).
  -export([init/1]).
  
  init(Name) -> ...
  ```

- 主程序

  ```erlang
  -module(world).
  -exdport([start/0]).
  
  start() ->
  	Joe = spawn(person,init,["Joe"]),
  	Susannah = spawn(person,init,["Susannah"]),
  	Rover = spawn(dog,init,["Rover"]),
  	Rabbit1 = spawn(rabbit,init,["Flopsy"]).
  ```

- Joe进程发送消息

  ```erlang
  Susannah ! {self(),"Hope ..."}
  ```

- Susannah进程接收消息

  ```erlang
  receive
  	{From,Message} ->
  		...
  end
  ```

### 并发编程的优点

- 性能随核数提升：适应多核处理器，让代码可以并行执行
- 方便扩展：程序由多个小型独立进程组成，通过添加进程或者CPU芯片数量可以方便扩展系统，虚拟机会自动在可用CPU之间分配资源给进程执行。
- 容错性：由于独立进程和硬件冗余，一个进程的错误不会导致另一个进程的崩溃。且为了防范计算机或数据中心故障，还需要进行故障探测。而这些都内建于虚拟机中。
- 清晰性：用并发性语言描述并行世界

---

## 环境安装

```shell
#解压源代码
tar -xvf otp_src_20.3.tar.gz
cd otp_src_20.3
sudo mkdir -p /usr/local/erlang/21 && ./configure --prefix=/usr/local/erlang/21 --without-javac
make
sudo make install
```

###  编译配置

EMakeFile-新建并放在项目代码下

```erlang
  1 {                                                                                                  	 2  [
  3   'src/*'
  4  ]
  5  ,[
  6    {outdir, "./ebin"}
  7   ]
  8 }.
```

### 运行编译

```shell
$ erl -make #全部编译
$ erl -pa ebin/ #指定运行目录
```

### 测试

```shell
 code:soft_purge(xxx),code:load_file(xxx). #重载模块
?assertEqual(A,B),
f() #忘掉变量定义
```

---

## 物理结构-基本概念

### 元组（类比不可变长度数组）

属性数量固定的对象

### 列表（类比链表）

属性数量不定的对象

---

## 模块与函数

### 分隔符

- 逗号，分割函数普通语句，数据构造和模式中的参数
- 分号；分割子句(类比方法重载)
- 句号. 分割大函数

### fun()(类比匿名函数)

以fun为参数

```erlang
Even = fun(X) -> (X rem 2) =:= 0 end.
Even(19).
```

```erlang
filter(fun(X)->(X rem 2),List);
filter(fun event/1,List)
```

以fun为返回值

```erlang
Fruit = [apple,pear,orange].
MakeTest = fun(L) -> (fun(X) -> lists:member(X,L) end) end. "外部函数定义大类型，并返回内部fun
IsFruit = MakeTest(Fruit).  "内部函数定义返回业务值
IsFruit(apple).
```

demo(类比for循环)

```erlang
for(Max, Max, F) -> [F(Max)];
for(I, Max, F)-> [F(I)|for(I+1, Max, F)].
```

### 列表推导(简便列表元素处理)

```
[F(X) || X <- L].
%% begin end.可以看作一句表达式，在列表推导左侧使用
%% 右侧可以是判断函数，生成器(表达式序列-最后一个表达式的值) 好像会有叉积
```

位串生成器

```
 [A || <<A:1/bitstring>> <= <<1>> ].
```

### 内置函数

```
list_to_tuple
tuple_to_list
```

### 关卡(重载前提下再对输入参数做判断)

```
when ... ->
```

关卡分类

```
关卡序列 a;b;c

关卡 a,b,c
```

### case(执行结果+[关卡])

是一种模式匹配，可接收高阶函数，可以实现归集器

```erlang
case Expression of
	Pat1 (when Guard1) Exp1;
	...
end
```

### if(关卡-对输入的个性化过滤 返回布尔值 但不能是自己写的函数)

循环中判断 

```erlang
if
	Guard1 ->
		Exp1;
	Guard2 ->
		Exp2;
	...
end
```

### 构建列表

```
待处理列表 ：取出头部处理
结果列表 ：处理后插入头部[H|Result]，倒序(lists:reverse)
```

## 记录与映射组

### 声明记录(可设置默认值和空值undefined)

```
-record(Name,{key1 = Default1,
			key2 = Default2,
			key3}).
```

### 创建和更新记录

```
rr("record.hrl") 
rd(state, {a = 1})
X1 = #todo{status = aaa}
X2 = X2#todo{status = bbb}
rf(todo)
```

### 提取记录字段

```
#todo{status = S} = X1 %% 先模式匹配
X1#todo.status.
```

### 映射组

```
M1 = #{a => 1, b => 2}
M2 = M1#{c => 3}
M3 = M1#{b := 3}
#{b := B} = M1 %% 模式匹配
```

## 异常

### 捕获异常

try catch 可以针对模式匹配和异常类型模式匹配 _: _ （可以有堆栈信息）

```
try 
	Exprs
catch
	Class:ExceptionPattern:Stacktrace [when GuardSeq] ->
```

catch 单纯捕获 case模式匹配后进行处理 有堆栈信息

## 位语法

超过两个字节的整型用little

little big概念针对整型

构建还可以

```
<< <<1,2>>/binary, N>> %%解析
```

## 编码补充

```
动态代码载入
代码中调用是最新的
shell可以运行两个版本（所以soft_purge也会存在两个版本问题）

模式中的匹配操作符(别名)
fun1({tag1,A}=Z)

每个进程都有进程字典（一次性常量）

用++进行列表增长时，前者较短

shell显示不全用rp().
```

## 并发

### 基本函数

```
Pid = spawn(Mod, Func, Args)  #需要导出MFA,更新为新版
Pid = spawn(Func) 
Pid ! Message
receive .. end（一般为阻塞状态，等待接收消息，并保留该进程状态）
```

### 超时语法

```
receive
after Time -> 
	Expressions
end
```

Time为0时可以实现匹配后的函数不阻塞，可以在Expression中再次receive并直接返回

Time为infinity无限等待（相当于没加after？）

选择性接收并把未匹配的信息放入保存队列，保存队列会在新消息匹配成功或是定时时间到了重新放回邮箱。

### 注册邮箱（起名）

```
register(Atom, Pid)
unregister(Atom)
whereis(Atom) -> Pid|undefined
registered() -> [Atom :: atom()]
```

## ETS和DETS

### 表的类型

```
set
ordered_set
bag
duplicate_bag
```

### ets操作

```
ets:new（Name, [Option])
      Option :: Type | Access | named_table | {keypos,Pos}
              | {heir, Pid :: pid(), HeirData} | {heir, none} | Tweaks
ets:insert(TableID,X)
ets:lookup(TableID,Key) - 元组列表
ets:delete(TableID)
ets:match(TableId,Pattern)
```

### dets操作

```
dets:open_file(Filename)
dets:inserts(TableID,X)
dets:lookup(TableID,Key)-可以使用filename作为tableid
dets:close(TableID)
```

### 影响ets效率的因素

```
ordered_set底层用平衡二叉树实现（时间开销-logN）
其他是散列表实现（空间开销）
插入和查询操作时，在栈之间复制元组的数据结构，而二进制型会在堆外被多个进程和ets表共享，并进行引用回收。
```

## OTP-gen_server

```
由gen_server回调调用函数的函数
```

### start_link

(Name, Mod, Args, Options)

![这里写图片描述](https://img-blog.csdn.net/20151126111526344)

```
gen模块是很多behaviour的基础,对于gen进程的启动和同步操作进行了封装.
gen模块使用proc_lib:start
1、来更加安全的spawn出gen进程，然后阻塞在sync_wait这个函数里,等待gen进程的回应.
2、spawn出的gen进程会执行到Mod模块的init函数。然后gen_server会把{ok, Pid}发送给调用进程（Mod）告知gen进程已经完成准备工作,然后就进入了自己的循环函数gen_server:loop中，等待调用进程的下次消息。
start去回调init（执行的进程是分裂出的进程，而不是调用进程）        
```

### call

![这里写图片描述](https://img-blog.csdn.net/20151126111748017)

```
调用进程向gen进程发送call消息，和cast不同，调用进程会阻塞在wait_resp_mon函数里等待gen进程的回馈。收到消息后，gen进程会执行Mod:handle_call函数，并把执行的结果Reply直接发送给调用进程，然后自己再次进入循环等待新的消息。

默认情况下handle_call的返回是{reply,….}. 而调用进程阻塞在wait_resp_mon的默认超时时间是5s(-define(default_timeout, 5000)).
在spec里看到handle_call的返回值可以是{noreply,…}, 或者gen进程在处理其它事情而达到超时时间, 则调用进程会异常退出, 你也可以在调用gen_server:call/3来设置一个call命令的超时时间.
```

### cast

![这里写图片描述](https://img-blog.csdn.net/20151126111712475)

```
调用进程向gen_server进程发送cast消息，消息发送之后调用进程并不等待gen_server进程的消息回馈.
 
```

### **info**

![这里写图片描述](https://img-blog.csdn.net/20151126111900059)

```
可以给gen进程发送任何格式的信息，这类信息没有gen标签（未经显示调用call，cast），gen进程接收到这类消息会调用gen_server:handle_info来处理消息，处理过程与cast消息一样都不能反馈结果给调用进程，所以如果handle_info返回{reply，…}也会导致gen进程报错退出。
```

### **terminate**

![这里写图片描述](https://img-blog.csdn.net/20151126111933953)

```
在handle_*等回调函数处都可以返回{stop,…} 这样使得gen进程执行Mod:terminate, 进行进程退出前的收尾工作, 然后回到主循环gen进程再退出.
可以进行服务器重启
```

## Socket

```
string:tokens(binary_to_list(A),"\r\n").
```


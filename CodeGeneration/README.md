## 思想 --- 栈式抽象机

> 即将使用的结果永远保存在抽象机的顶部

### 前期准备

> 一定不能贸然实现代码，一定要先架构好，至少先把大致方向确定了，再实现内部细节！

#### 大体构想

- 枚举四元式类型
- 设置四元式处理的类
- 将所有要输出的中间代码保存在动态数组中，统一输出 --- 为了能正确生成Label
- 栈式抽象机 --- 可以理解成需要计算的元素总在栈顶。

#### 枚举四元式类型

#### 设计四元式类

> JAVA中用类代替了结构体，所以使用类对于正确性肯定没有影响。

### 中间代码生成

##### 注意语句分析的完整性！！

> 按照中缀表达式表示

##### 函数声明 ✔

```
源码形如：
	int foo( int a, int b, int c, int d)
中间代码：
	int foo()
	para int a
	para int b
	para int c
	para int d
```

##### 函数调用 ✔

出现情况 :: 单独成为一条语句或成为因子。

```
源码形如：
	i = tar(x,y)
中间代码：
	push x
	push y
	call tar
	i = RET
```

``funcRefParse()`` :: 返回一个表达式的临时变量标识。



##### 函数返回 ✔

```
源码形如：
	return (x)
中间代码：
	ret x
```

先保存表达式的值，然后再返回记录表达式值的临时变量。

##### 变量声明 ✔

数组定义也可以处理了。

##### 常数声明 ✔

第二个运算符为空，只有第一个运算符。

##### 表达式【可优化】  ✔

> --- 第一阶段的难点在于如何解析表达式 --- 递归下降法

**注：现在不用错误处理了，因此项和因子的返回值类型都可以重新定义。**

【超难点】

- 表达式一定要有返回值 --- 返回储存表达式最终结果的变量

  > 因为表达式不可能单独出现

- 在表达式分析的过程结束的时候就应该将内容（经过逆波兰转换的表达式）保存到全局的中间代码中 【不然没法处理啊…】

- 表达式的返回值 --- 应该通过genVar()来产生一个名称。

- ###### 适应变化

  - 表达式的返回值
  - 项的返回值
  - 因子的返回值
  - 函数调用的返回值

- ###### 表达式的计算

  - 生成逆波兰表达式

    中缀表达式转后缀表达式，假设没有括号

    从左往右扫描表达式串，对于表达式串的每一个元素
  
    - 如果是标识符，直接输出。
    - 如果是加法运算符：将之前的加法运算符输出，并且入栈。
    - 如果是乘法运算符：直接输出。
  
    如果栈不为空，输出栈中所有元素。
  
  - 由逆波兰表达式生成中间代码
  
    逆波兰表达式的计算
    
    > 在表达式计算中可以产生四元式。
    >
    > 一个运算符对应一个输出，对应一个临时变量。
    
    - 建立一个数据结构 --- 栈 --- 存储运算结果。
    - 每次遇到一个标识符将其压入栈中
    - 每次遇到一个运算符，从栈中弹出两个元素进行运算，并且将结果加入栈中。
    - 最后栈顶的元素就是表达式的结果。

##### 条件判断 --- 照原样生成即可 ✔

```
源码形如：
	x == y
中间代码：
	x == y
```

> 【疑问】如果条件判断只有一个表达式？？

##### 条件或无条件跳转 ✔

> 条件还有另一个名字，曰：cmp（比较）

**注意**

- ``mips``中处理比较的方式。
- 是先产生跳转指令，然后再设置label。用target保存label的名称。

跳转这里有点绕 --- mips的比较方式是将两个表达式的值相减然后与0进行比较。【需要在条件分析中完成这一步】

```
1. 目标条件取反
2. 左边为条件解析出的结果，右边为0 
3. 跳转
```

- if -- 控制语句将产生跳转
- 循环语句产生跳转

##### 带标号的语句 ✔

在语法分析的过程中产生

##### 数组赋值或取值 ✔

```
源码形如：
	a[i] = b * c[j]
中间代码：
	t1 = c[j] // 这个地方需要注意一下
	t2 = b * t1
	a[i] = t2 
```

##### 循环语句的处理

- 循环变量会不断变化 --- 但是好像没有影响
- 每次循环体运行结束需要跳到循环体开头

##### 读语句 ✔

> 区分int和char

##### 写语句

【疑问】

​	char型变量赋值给string不会出问题吧。

- 字符串按照原样输出，不存在转义的问题
- 字符串和表达式之间无空格
- 表达式为整型，直接输出整数
- 表达式为字符型，输出字符
- 每个printf结束都要输出换行符

#### 难点

表达式的解析，这是一个模块嵌套的问题。

- 判断是否为标识符 --- 这个在逆波兰表达式的转换中十分重要.

#### 中间代码的输出

- 先全部保存到vector中，然后遍历vector进行输出。
- 函数定义时参数有两种方法：：1. 每出现一个参数就生成一条中间语句。2. 将所有参数统一保存，统一生成。
- 函数调用时,形参的传递也有两种方式。（类似上面）

##### 不同类型的四元式输出

- 条件 --- 添加标志``CMP``

  1. 只有一个表达式
  2. 两个**整型**表达式做比较

- 条件或无条件跳转

  无条件跳转 --- GOTO

  条件跳转：$$<满足的跳转条件> Label$$

- 读语句

  1. 读入一个字符
  2. 读入一个整数

#### 待处理

- 临时变量编号的清零
- 如何减少寄存器的使用  --- 进入前保存varCount，出来前恢复varCount。
  - 将tmpVar设置为0
    - 复合语句
    - if语句的statement之前；else 语句分析之后
    - 表达式分析结束
    - 条件分析结束

#### 测试过程

> 不要抵触读代码

- 先把样例过了
- 拿**正确的测试程序(之前收集的)**进行覆盖性测试

### 中间代码生成mips汇编

> #### 优化方法

- 基本块内的优化
  - DAG图 --- 防止基本块内的表达式重复计算，但是要注意数组和函数引用的情况。
  - 窥孔优化
  - 常量合并和传播
- 全局优化
  - 到达定义分析
  - 活跃变量分析
  - 消除**全局**公共子表达式
  - 复制传播
  - 死代码删除

> #### 针对每一种中间代码生成汇编语句

##### 函数调用语句

> 统一规定（默认的）
>
> 函数参数保存在``$a0~$a3``中，如果多于4个参数，就保存在栈中。

jal 直接跳到函数名名称

函数结束时，记得加上jr ra。

##### 赋值语句

- 赋值语句的**左边**是取地址赋值，即包含两个步骤

  1. 获取变量所在的地址。

  2. 通过``sw``对变量所在的地址进行赋值。

  注意 ：数组元素的下标是取值。

- 赋值语句的右边是取值操作 -- 使用 ``lw``

```
中间代码
target :: 保存了赋值语句的目标
left :: 保存了赋值语句的右边

mips:
$t2 :: 保存赋值的目标地址
$k0 :: 保存局部变量的基地址
$k1 :: 保存局部变量的偏移
```

##### 标签

##### 数组定义

数组全部定义成全局变量，放在data段。

> 注意：data段是从0x00000000开始分配内存的。

```
.data
symbol:.space 28
array:.space 28 // 为数组分配空间，一个int类型占4字节
```

##### 变量定义

【变量定义到底放在哪比较合适：data区】

要区分全局变量和局部变量。

##### 常量定义

不在mips中定义，使用的时候直接赋值。

##### 函数定义

进入函数时，要保存ra寄存器的返回地址；

函数跳出之前，要恢复函数的运行栈。

- 参数相关

  > 假设参数小于四个: and :都不会被def，只会被use。

  为了区别参数和局部变量，参数的value值设置为``19981111``

##### 表达式

**注意**：在genTempAddr()函数中，做出了如下归一化处理：

```
string Optimizer::getTempReg(const string &tempReg, const string &dstReg) {
    string localOffset = tempReg.substr(1, tempReg.length() - 1);
    mipsFile << "move " << dstReg << ", $t" + to_string(stoi(localOffset) % 8) << endl;
    return dstReg; // 这里返回什么还没想好
}
// 可以保证表达式所用的寄存器编号不超过 8
```

注意一下表达式赋值的计算顺序：

先计算表达式右边的运算，然后赋值给左边。

```
表达式左边肯定是一个临时寄存器
表达式右边是两个因子
target :: 赋值目标
left :: 第一个操作数 用寄存器t8保存
right :: 第二个操作数 用寄存器t9保存
```



##### 条件

##### 无条件跳转语句

##### 读语句

- 整数
- 字符

##### 写语句

- 字符串 :: 最终要打印的字符串统一放在data段进行输出，包括换行符和空格。

  **注意这个地方必须要像4字节对齐**。

  ```
  需要定义时
  string1 :.asciiz "abcdefg!@$!$#%!#$^"
  
  需要打印时：
  la $a0, space
  li $v0, 4
  syscall // 就可以打印空格
  ```

- 整数

  ```
  syscall 11 // 输出整数
  ```

- 字符

  ```
  out << "li $a0 " << (int)item.target.at(0) << endl;
  out << "li $v0 11" << endl;
  out << "syscall" << endl;
  // 先一个强转，得到ascii值，然后直接用syscall 11来输出字符
  ```

##### 返回语句

1. 恢复寄存器 -- 根据函数定义时保存的寄存器来恢复
2. ``jr ra``

### 第三步 --- 优化方案

- 基本块内的优化

  建立DAG图,消除公共子表达式(针对每一个```基本块```)

- 删除冗余代码

  恒等式的删除：比如 x = x + 0; x = x - 0; a[3] = a[3] + 0;

  常数值传播 --- 临时变量的重复利用(一个函数内存在大量的临时变量只使用过一次就再也没有用的情况)--->针对```基本块```的重新规划

- 使用全局寄存器 ``s0~s7``来保存**高频使用的全局变量**的值。


### 寄存器分配

```
sp -> 整个程序的运行栈空间
fp -> 函数的起始地址
k0, k1 -> 用来计算数组、局部变量的地址（tmp）
/* 函数定义中需要保存的寄存器：
s0~s7 :: 全局寄存器
$a0 ~ $a3 :: 函数的参数
fp :: 函数运行栈的开始地址 --- 便于恢复栈指针
函数返回需要恢复的寄存器*/
```


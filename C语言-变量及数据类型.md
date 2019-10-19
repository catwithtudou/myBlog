---
title: C语言-变量及数据类型
date: 2019-10-19 15:32:21
tags: ["C"]
categories: "C"
---

# C语言-变量及数据类型

## 变量及数据类型

### 变量

#### 定义

变量是**计算机中一块特定的内存空间**

- 由一个或多个连续的字节组成

> 我们知道计算机处理的数据都是放在内存中处理,而若想要引用变量,那么就要找到它内存空间的地址,而空间里就存放着变量的值

不同数据存入具有不同内存地址的空间,相互独立

#### 命名

- 为什么要命名呢?

因为我们可以通过变量名来简单快速地找到在内存中存储的数据

即变量名就相当于变量的标识符,也相当于它内存空间的地

- 那么如何命名呢?

每个语言的命名规则都有它自己的一套规则,而现在我们来介绍C语言中变量的命名规则

1. C语言规定变量名（标示符）只能由字母、数字和下划线3种字符组成，且第一个字符必须为 字母或下划线 
2. 变量名不能包含除 _ 以外的任何特殊字符，如：%、# 、逗号、空格等 
3. 不可以使用保留字(类似于if,else,float等),因为它是保留给语言本身使用的

```c
void name()
{
    //变量名
    int 12aaa;
    int %asdas; 
    int _abc;
    int abc;
    int #sdas;
    int if; 
    int else;
    int float;
}
```

### 数据类型

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019093025.png)

| 数据类型     | 类型说明符 | 位数 |                           |
| ------------ | ---------- | ---- | ------------------------- |
| 整型         | int        | 32   | -2147483648 ~ +2147483647 |
| 短整型       | short int  | 16   | -32767 ~ +32768           |
| 长整型       | long int   | 32   | -2147483648 ~ +2147483647 |
| 单精度浮点数 | float      | 32   | -3.4E-38 ~ +3.4E+38       |
| 双精度浮点数 | double     | 64   | 1.7E-308 ~ 1.7E308        |
| 字符型       | char       | 8    | -128~+128                 |

> 限定符
>
> short和long满足实际需要的不同长度的整型数
>
> short 通常为16位
>
> long  通常为32位
>
> signed和unsigned(类型限定符)可用于限定char或任何整型
>
> unsigned  无符号即>=0
>
> signed  有符号
>
> 不带限定符的类型的位数取决于具体机器

这里的取值范围的计算我们来了解一下怎么换算的

首先我们需要了解

8bit(位)=1Byte(字节)

1024Byte(字节)=1KB

1024KB=1MB

1024MB=1GB

然后我们需要知道各个类型在各个机器上面占用的字节数

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019094911.png)

最后我们通过二进制的话换算就知道了

上面我提到的那个数据类型的取值范围是32位机器的字节数

> 顺便提一下如何查看本机是64位机器还是32位机器
>
> ![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019095421.png)

> 关于double和float类型的区别
>
> 单精度浮点数在机内占4个字节，用32位二进制描述。
> 双精度浮点数在机内占8个字节，用64位二进制描述。
>
> 浮点数在机内用指数型式表示，分解为：数符，尾数，指数符，指数四部分。
> 数符占1位二进制，表示数的正负。
> 指数符占1位二进制，表示指数的正负。
> 尾数表示浮点数有效数字，0.xxxxxxx,但不存开头的0和点
> 指数存指数的有效数字。
>
> double 和 float 的区别是double精度高，有效数字16位，float精度7位。但double消耗内存是float的两倍，double的运算速度比float慢得多，C语言中数学函数名称double 和 float不同，不要写错，能用单精度时不要用双精度（以省内存，加快运算速度）。

### 声明和使用变量

- 声明变量

  **DataType variableName;   数据类型 变量名**

- 定义时初始化变量

  **DataType variableName = value; 数据类型 变量名 = 值**

- 定义后初始化变量

  **DataType variableName; 数据类型 变量名;**

  **variableName = value; 变量名 = 值;**

#### 整型变量

> 注意变量名不要重复

```c
#include<stdio.h>

void main()
{
    //第一种
    int a = 2500;
    //第二种
    int b;
    b=2500;
    
    //打印
    printf("%d\n",b);
}
```

- 转换说明符 
  - 转换说明符确定变量在屏幕上 的显示方式 
  - %表示转换说明符的开头 
  - %d表示b输出为一个十进 制数字 
  - %%将会输出一个%

#### 单精度浮点型变量

位数：32位 

空间：4个字节 

取值范围：𝟏𝟎−𝟑𝟖 至 𝟏𝟎+𝟑𝟖 

7位有效数字

```c
#include<stdio.h>

void main()
{
    float width = 2.5f;
    float height = 4.0f;
    float s;
    s = width * height;
    printf("%f\n",s);
}
```

> 注意:
>
> 1、float类型变量赋值时需要在数值的末尾加上一个f 
>
> 2、float类型的占位符是%f 
>
> 3、%.2f可以控制数字的显示精度

#### 双精度浮点型变量

位数：64位

空间：8个字节 

取值范围：𝟏𝟎−𝟑𝟎𝟖 至 𝟏𝟎+𝟑𝟎𝟖 

16位有效数字 

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    double radius = 2.5;
    double area = 3.141592653 * radius * radius;
    printf("r:%lf  s:%lf\n",radius,area);
}
```

> 注意： 
>
> 1、通常小数被存储为double类型，如2.0和9.0 
>
> 2、数字后使用L时，数字成为long double类型，如4.76L 
>
> 3、%.2lf可以控制数字的显示精度

#### 字符型变量

位数：8位 

空间：1个字节 

取值范围：-128 ~ +127 

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char ch1 = 'a';//小写字母a
    char ch2 = 'A';//大写字母A
    char ch3 = ' ';//空格
     printf("字符\tASCII码\n");
     printf("%c\t%d\n", ch1, ch1);
     printf("%c\t%d\n", ch2, ch2);
     printf("%c\t%d\n", ch3, ch3);
     printf("%c\t%d\n", ch1 - 32, ch1 - 32);
}
```

#### ASCII码汇总表

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019133907.png)

> 随堂练习
>
> - 需求说明
>
>   接收用户输入的小写字母，转换为大写字母并显示
>
> 随堂练习
>
> - 需求说明
>
>   将字符型的数字转换成整型



#### 转义序列

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019134139.png)

#### scanf函数

C函数库包含了多个输入函数，`scanf`是最通用的一个，可以读取不同格式的数据 

```c
int a;
printf("请输入a: ");
scanf("%d",&num); //读取基本类型的值记得加'&'符号
```

| 转换字符串 | 含义                     |
| ---------- | ------------------------ |
| %d         | 把输入解释成有符号整型   |
| %c         | 把输入解释成字符型       |
| %s         | 把输入解释成字符串       |
| %f         | 把输入解释成单精度浮点数 |
| %lf        | 把输入解释成双精度浮点数 |

> 随堂练习
>
> - 需求说明
>
>   实际伤害 = 武器伤害 * (玩家力量 + 100) / 100
>
> - 现有武器伤害:256,要求输入玩家的力量,打印实际伤害

## 运算符

- 运算符

  **赋值运算符,算术运算符,关系运算符,逻辑运算符**

- 表达式

  **符号与操作数的组合**

  ```c
  a = (b+3) + (b-1)
  ```

### 赋值运算符

- 单等号 `=`

  计算顺序:**从右往左**

```c
double salary = 3200.0;
double total = salary * 2;
char sex = 'F';
int score = 98;
```

- 复杂赋值运算符

```c
int num=5;
x += 10; //x = x + 10
x -= 10; //x = x - 10
x *= 10; //x = x * 10
x /= 10; //x = x / 10
x %= 10; //x = x % 10;
```

### 算术运算符与表达式

- 算术运算符
  - 一元运算符:`++`,`--`
  - 二元运算符:`+`,`-`,`*`,`/`,`%`

- 算术运算符示例

  ```c
  int num1 = 5,num2 = 2;
  double result1,result2,result3,result4
  ```

#### 类型转换

**自动类型转换:**

- 原则:把表示范围小的类型的值转换到表示范围大的类型的值

```c
short -> int -> long -> float -> double
```

**强制类型转换:**

- 语法:`(类型名) 变量或数值`

```c
int num1=5,num2=2;
double result = (double)num1/num2;
printf("num1 / num2 = %lf\n",result);
```

### 关系运算符

**关系运算符可以比较大小,高低,长短**

- `>`,`<`
- `>=`,`<=`
- `==`,`!=`

> C语言中,0表示假,1(非零)表示真

### 逻辑运算符

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019145429.png)

### 位运算符

- & 按位与

| x    | y    | r    |
| ---- | ---- | ---- |
| 0    | 0    | 0    |
| 0    | 1    | 0    |
| 1    | 0    | 0    |
| 1    | 1    | 1    |

- | 按位或

| x    | y    | r    |
| ---- | ---- | ---- |
| 0    | 0    | 0    |
| 0    | 1    | 1    |
| 1    | 0    | 1    |
| 1    | 1    | 1    |

- ~ 按位非

| x    | r    |
| ---- | ---- |
| 0    | 1    |
| 1    | 0    |

- ^ 按位异或

| x    | y    | r    |
| ---- | ---- | ---- |
| 0    | 0    | 0    |
| 0    | 1    | 1    |
| 1    | 0    | 1    |
| 1    | 1    | 0    |

- << 左移

  右侧空位补0

  \>> 右移

  左侧空位补符号位

  ```c
  #include <stdio.h>
  #include <stdlib.h>
  
  int main()
  {
      //00000000000000000000000000000001
      int num = 1;
      //00000000000000000000000000000010
      num = num << 1;
      printf("%d\n",num);
      //00000000000000000000000000000001
      num = num >> 1;
      printf("%d\n",num);
      //10000000000000000000000000000010
      num = -2;
      //10000000000000000000000000000001
      num = num >> 1;
      printf("%d\n",num);
      //10000000000000000000000000000001
      num = -1;
      num = num >> 1;
      printf("%d\n",num);
  }
  ```

### sizeof运算符

**使用sizeof运算符可以获得数据类型占用内存空间的大小**

```c
int main()
{
    printf("sizeof(int) = %d\n",sizeof(int));
    //以下列举出long,float,double,char
    return 0;
}
```

### 运算符优先级

让我们先来猜猜看这个例子的结果是什么

```c
printf("%d\n",  !((18 + 45 % 3 * 5) > 16) );
```

- 同一行中的各运算符具有相同的优先级,各行间从上往下优先级逐行降低

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019151035.png)

> 记忆技巧
>
> 1.算术运算符 > 关系运算符 > 逻辑运算符
>
> 2.可以通过()控制表达式的运算顺序,()优先级最高
>
> 3.单目运算符包括! ~ ++ -- sizeof,优先级别高

## 分支结构

### if结构

- 先判断后执行

```c
if(条件为真)
{
    //代码块1
}
else
{
    //代码块2
}
```



![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019151514.png)

> 随堂练习
>
> - 需求说明
>
>   判断字符是否为小写

### 多重if结构

常用来进行区间判断

```c
if(条件为真)
{
    //代码块1
}
else if(条件为真)
{
    //代码块2
}
else
{
    //代码块3
}
```

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019152457.png)

> 随堂练习
>
> - 需求说明
>
>   用多重if语句来续写一段故事
>
>   鲜花的价格

### switch结构

```c
switch(表达式){
    case 常量1:
        语句1;
        break;
    case 常量2:
        语句2;
        break;
    ......
        default:
        语句;
}
```

- 注意
  - switch后的表达式只能是整型或字符型
  - case后常量表达式的值不能相同
  - case后允许多条语句,不需要大括号
  - 如果不添加break语句,需要注意执行顺序
  - case和default子句的先后顺序可以自动变动
  - default子句可以省略

### switch和多重if对比

- 相同点

  都是用来处理多分支条件的结构

- 不同点

  - switch:等值条件判断-条件是有限个的时候
  - 多重if:判断某个连续区间的情况

## 参考

<https://study.163.com/course/courseMain.htm?courseId=1003425004&_trace_c_p_k2_=45b2a79c654549a89db231a73169f426>
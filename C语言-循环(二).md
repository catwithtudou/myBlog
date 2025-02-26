---
title: C语言-循环(二)
date: 2019-10-28 21:53:54
tags: ["C"]
categories: "C"
---

# C语言-循环(二)

## For循环

- 语法:

  ```c
  for(表达式1;表达式2;表达式3)
  {
      语句;
  }
  ```

- `表达式1`:通常是为循环变量赋初值,比如 *i=0 或 cnt=20*

  `表达式2`:循环条件,是否继续执行循环,比如 *i<10 或 count>=5 或 cnt==20*

  `表达式3`:更新循环变量的值,比如:*i++ 或  cnt+=2 或 count--*

  **表达式123都可以省略,但';'是绝对不能省略的**
  
  

```c
  const int N = 20; //常量
  int i;
  for(i=0;i<N;i++)
      printf("再别康桥\n");
```

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191028205317.png)

### 使用for循环

**例一**

- 循环输入6个月的工资,计算其平均工资
  - 使用循环求和
  - 循环三要素

 **例二**

- 打印简易加法表

  - 例如

    ```out
    请输入一个数字:8
    0 + 8 = 8       1 + 7 = 8
    2 + 6 = 8       3 + 5 = 8
    4 + 4 = 8       5 + 3 = 8
    6 + 2 = 8       7 + 1 = 8
    ```

### for循环常见问题

- 忘记定义循环变量或初始化
- 循环条件缺少时会造成死循环
- 循环变量不更新也会造成死循环
- 不可省略分号

## braek与continue

### break语句

**作用:跳出循环,执行循环之后的语句**

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191028210750.png)

#### 随堂练习

- 循环输入玩家的年龄，如果年龄为负则停止输入，提示输入错误
  - 一旦输入的值为负,使用break跳出循环

- 循环录入用户性别，只能使用字符m/M(男)或f/F(女)。一旦输入错误，结束录入,且要求统计录入正确的次数

### continue语句

**作用:跳出本次循环,继续下次循环**

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191028211031.png)

> 随堂练习
>
> - 循环输入5个人的年龄,统计年龄为负的录入次数
>   - 如果录入正确则跳过,若输入错误则计数器+1

### break和continue对比

- 使用场合
  - break可用于switch结构和循环结构中
  - continue只能用于循环语句
- 作用(循环结构中)
  - break语句终止与某个循环,程序跳转到循环块外的下一条语句
  - continue跳出本次循环,进入下一次循环

#### 随堂练习

- 求1-100之间的偶数和 
  - 循环累加
  - 判断是否为偶数:模2是否为0
  - 如果为奇数就跳过,如果为偶数则执行累加

## 循环结构总结

- 相同点
  - 多次重复执行一个或多个任务时考虑使用循环来解决问题
- 区别
  - 除了语法不同,判断和执行的顺序也不同
  - 适用情况不同
    - 循环次数确定的情况下,通常选用for循环
    - 循环次数不确定的情况下,通常选用while或do-while循环

### 随堂练习

- 用'*"打印一个菱形图案

  - 例如

    ```out
       *
      ***
     *****
    *******
     *****
      ***
       *
    ```

  - ```c
    #include <stdio.h>
    #include <stdlib.h>
    int main()
    {
        int k;
        for(int i=4,k=1;i>0;i--,k+=2)
        {
            for(int j=1;j<i;j++)
            printf(" ");
            for(int l=0;l<k;l++)
            printf("*");
            printf("\n");
        }
        for(int i=1,k=5;i<4;i++,k-=2)
        {
            for(int j=0;j<i;j++)
                printf(" ");
            for(int l=0;l<k;l++)
                printf("*");
            printf("\n");
        }
        return 0;
    }
    ```

- 求阶乘
  
- 输入n,求n!并输出
  
- 输出1000以内能3整除且个位数为5的所有整数
- 求最大数和最小数
  
- 输入n个整数,找出其中的最大值和最小值
  
- 求100-999的水仙花数

  - 水仙花数的定义

    个位数的立方加上十分位数的立方再加上百分位数的立方等于它自身

## 参考

<https://study.163.com/course/courseMain.htm?courseId=1003425004&_trace_c_p_k2_=45b2a79c654549a89db231a73169f426>


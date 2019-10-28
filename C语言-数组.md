---
title: C语言-数组
date: 2019-10-29 01:01:01
tags: ["C"]
categories: "C"
---

# C语言-数组

## 什么是数组

**数组是一个变量,由数据类型相同的一组元素组成**

> 变量:内存中的一块空间
>
> 数组:内存中的一串连续的空间

注意**连续**,数组申请的空间是连续的,数组的深入理解就是从连续这里开始延伸的

数组的**结构和基本要素**有以下几点

- **标识符**:数组的名称,用于区分不同的数组
- **数组元素**:向数组中存放的数据
- **元素下标:**对数组元素进行编号
- **元素类型**:数组元素的数据类型

> 注意
>
> 1. 数组只有一个名称,即标识符
> 2. 元素下标标明了元素在数组中的位置,从0开始
> 3. 数组中的每个元素都可以通过下标来访问
> 4. 数组长度固定不变,避免数组越界
> 5. 数组里面每个元素的数据类型都是一致的
>
> ![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191028214852.png)

## 数组的使用

- 语法

  datatype arrayName[size];

```c
int num[25];
char array_of_name[30];
double curr_salary[35];

#define N 50
int emp_id[N];
const int SIZE = 100;
double prices[SIZE];
```

### 一维数组的初始化

```c
//正确:后面的元素个数与声明的一致
int years[5]={1,2,3,4,5};

//正确:后面2个元素未初始化,默认值为0
int num[5]={1,2,3};

//正确:元素个数为2
int days[]={1,15};

//错误:未知元素个数
int array[]={};
```

#### 一维数组的动态赋值

- 动态地从键盘录入信息并赋值

  ```c
  #include<stdio.h>
  int main()
  {
      int score[5];
      for(int i=0;i<5;i++)
          scanf("%d",&score[i]);
      return 0;
  }
  ```

> 随堂练习
>
> - 假如现在有一个数列:8,4,2,1,10,13
>   - 循环输出数列的值
>   - 求数列中所有值的和和平均值
>   - 猜数游戏:从键盘中任意输入一个数据,判断数列中是否包含此数

#### 随堂练习

- 需求说明:
  - 求数组中的最大值和最小组,并输出它们在数组中的位置
  - 定义一个整型数组,赋值后求出奇数个数和偶数个数
  - 查找输入的数组在数组中的下标,没有找到则下标为-1

## 数组排序

### 冒泡排序

![img](https://mmbiz.qpic.cn/mmbiz_gif/D67peceibeISwc3aGibUlvZ0XqVnbWtBRiaC1S2jpXRzXcZVn0aP6BYnkO2FJicNstxicHmf9wMIic5FV0I75ptv5jYA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

- 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
- 针对所有的元素重复以上的步骤，除了最后一个。
- 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>
#define Length 5

int main()
{
    int a[Length]={2,3,1,5,4};
    for(int i=0;i<Length-1;i++)
    {
        for(int j=0;j<Length-i-1;j++)
        {
            if(a[j]>a[j+1])
            {
                int temp = a[j];
                a[j] = a[j+1];
                a[j+1] = temp;
            }
        }
    }
    for(int i=0;i<Length;i++)
        printf("%d\n",a[i]);
}
```

### 选择排序

![img](https://mmbiz.qpic.cn/mmbiz_gif/D67peceibeISwc3aGibUlvZ0XqVnbWtBRiaB2dW1vA5SganRPChytYTFiaJL2PkXlL7XmhYmqIAzBHj0VvgJZs0vmA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

- 首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置
- 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。
- 重复第二步，直到所有元素均排序完毕。

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>
#define Length 5

int main()
{
    int a[Length]={2,3,1,5,4};
    for(int i=0;i<Length-1;i++)
    {
        for(int j=i+1;j<Length;j++)
        {
            if(a[i]>a[j])
            {
                int temp = a[i];
                a[i] = a[j];
                a[j] = temp;
            }
        }
    }
    for(int i=0;i<Length;i++)
        printf("%d\n",a[i]);
}

```

### 插入排序

![img](https://mmbiz.qpic.cn/mmbiz_gif/D67peceibeISwc3aGibUlvZ0XqVnbWtBRiaiatKZU4exjwcluduiclJOdZB0oZQicCrpIEaSJJg8iaia58viauSK3nhofqA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

- 将第一待排序序列第一个元素看做一个有序序列，把第二个元素到最后一个元素当成是未排序序列。
- 从头到尾依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置。（如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入到相等元素的后面。）

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>
#define Length 5

int main()
{
    int a[Length]={2,3,1,5,4};
    for(int i=1;i<Length;i++)
    {
        int temp = a[i];
        int j;
        for(j=i;j>0;j--)
        {
            if(temp<a[j-1])
            {
                a[j]=a[j-1];
            }else{
                break;
            }

        }
        a[j]=temp;

    }
    for(int i=0;i<Length;i++)
        printf("%d\n",a[i]);
}
```

## 数组的插入和删除

### 插入

假如我们现在要在一个数组下标为i的位置插入一位数据

- 首先我们需要考虑插入数据之前判断数组空间是否已满,若已满则不能插入直接退出
- 然后,我们通过将数组在i及i之后的数据都往后移动一步
- 最后,我们再在i位置赋上我们想要插入的那个数据

```c

```








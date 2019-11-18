---
title: C语言-指针与一维数组
date: 2019-11-18 18:32:21
tags: ["C"]
categories: "C"
---

# C语言-指针与一维数组

为什么引入指针的概念?

指针有如下好处：

- 为函数提供修改变量值的手段 

- 为C的动态内存分配系统提供支持 

- 可以改善某些子程序的效率

- 为动态数据结构（如例链表、队列、二叉树等）提供支持 

## 指针

**定义:指针是一个值为内存地址的变量(或数据对象)**

![M6Wixf.png](https://s2.ax1x.com/2019/11/18/M6Wixf.png)

> 注意:指针的本质在于它只是一个变量,特殊在于它存的值是内存地址

### 声明及初始化指针变量

- 基本用法

  `数据类型 * 指针变量名;`

```c
int * ptr_num;
char * ptr_name;
float * ptr_money;
double * p_price;
```

> 注意:
>
> 在头文件<stdio.h>中,NULL被定义为常量
>
> 即有 int * ptr_num=NULL;
>
> 指针的初值设为空,表示智珍珍不指向任何地址

### 取地址符&

```c
int num = 1024;
int * ptr_num;
//取num变量的地址赋值给ptr_num
ptr_num = &num;
```

![](http://img.zhengyua.cn/img/20191118184621.png)

### 间接运算符

```c
int num = 1024;
int * ptr_num;
ptr_num = &num;
*ptr_num = 1111;
//num = 1111;
```

### 寻址方式

- 直接(寻址)访问
  - 直接按变量地址来存取变量内容的访问方式
- 间接(寻址)访问
  - 通过指针变量来间接存取它所指向的变量的访问方式 

### 使用示例

- 例一

```c
#include <stdio.h>
void main()
{
    int num = 1024;
    int * ptr_num;
    ptr_num = &num;
    printf("num 值:%d\n",num);
    printf("num 内存地址:%p\n",&num);
    printf("ptr_num 值:%p\n",ptr_num);
    printf("ptr_num 内存地址:%p\n",&ptr_num);
    printf("Ptr_num 指向值:%d\n",*ptr_num);
}
```

- 例二

```c
#include <stdio.h> 
void main() { 
    int num1 = 1024; 
    int num2 = 2048; 
    int * ptr1, * ptr2; 
    ptr1 = &num1; 
    ptr2 = &num2; 
    printf("num1的值是%d\tnum1的地址是：%p\n", num1,ptr1);     printf("num2的值是%d\tnum2的地址是：%p\n", num2, ptr2); 
    //ptr2 = ptr1; 
    *ptr2 = *ptr1; 
    printf("重新赋值后：\n"); printf("num2的值是%d\tnum2的%p\n", num2, ptr2); 
}
```

> 随堂练习
>
> - 使用指针的来间接实现两数交换

### 小结

- 指针同样是一个**变量**，只不过该变量中存储的是另一个对象的内存地址

  ```c
  int * p;
  p = 100;
  ```

- 如果一个变量存储另一个对象的地址，则称该变量**指向**这个对象 

- 指针变量可以**赋值**，指针的指向在程序执行中可以改变

> 指针 p 在执行中某时刻指向变量x，在另一时刻也可以指向变量y

注意:

- 指针变量的命名规则和其他变量的命名规则一样 

- 指针不能与现有变量同名 

- 指针可存放 C 语言中的任何基本数据类型、数组和其他所有高级数据结构的地址 

- 若指针已声明为指向某种类型数据的地址，则它不能用于存储其他类型数据的地址 

  ```c
  int i;
  float * p;
  p = &i;
  ```

- 应为指针指定一个地址后，才能在语句中使用指针

  ```c
  int * p;
  *p = 100;
  ```

> 指针变量与其它类型变量的对比
>
> - 共性
>
>   - 在内存中占据一定大小的存储单元
>
>   - 先定义，再使用
>
> - 指针变量的特殊性
>
>   - 指针变量只能存放地址，而不能存放数据
>
>   - 必须初始化后才能使用，否则指向不确定的存储单元
>
>   - 只能指向同一基类型的变量，否则warning.
>
>   - 可参与的运算：加、减、关系、赋值

## 指针与数组

让我们先来复习一下数组的概念

- 存储在一块连续的内存空间中
- 数组名就是这块**连续内存空间的首地址**即&num[0]

因为数组申请的内存空间是连续的,所以用指针也可以来操作数组

```c
int num[5]={1,2,3,4,5};
int * ptr_num;
ptr_num = &num[0];
//ptr_num = socre;
```

### 指针元素引用方式

#### 数组名+下标

这是我们最常见的一种

```c
void main()
{int  a[10],i;
  
 for (i=0; i<10; i++)
    scanf("%d", &a[i]);
 for (i=0; i<10; i++)
    printf("%d ", a[i]);
} 
```

#### 数组名+偏移量

```c
void main()
{
 int  a[10] ,i;
 for (i=0; i<10; i++)
    scanf(“%d”,a+i);
 for (i=0; i<10; i++)
    printf(“%d”,*(a+i) );
} 
```

#### 数组变量+偏移量

```c
#include <stdio.h>
#include <stdlib.h>
void main()
{
    int  a[10],*p=a,i;
    for (i=0; i<10; i++)
        scanf(“%d”,&p[i]);
    for (i=0; i<10; i++)
        printf("%d ",p[i]);
}
```

> 注意
>
> p和a的区别是p有相应的内存空间
>
> 而a没有相应的内存空间
>
> 定义数组a申请的10个空间分配给了a[0]至a[9]

#### 指针变量+偏移量

```c
#include <stdio.h>
#include <stdlib.h>
void main()
{
    int  a[10],*p=a,i;
    for (i=0; i<10; i++)
        scanf("%d",p+i);
    for (i=0; i<10; i++)
        printf("%d",*(p+i));
}
```

#### 指针变量

```c
#include <stdio.h>
#include <stdlib.h>
void main()
{
    int  a[10], i;
    int  *p;
    for (p=a; p<(a+10); p++)
        scanf("%d",p);
    for (p=a; p<(a+10); p++)
        printf("%d",*p);
}
```

#### 总结

- a[i]或*(a+i)

- p[i]或*(p+i)

- *(p++)

### 指针运算

- 指针的算术运算

  - 指针的递增和递减

    ```c
    int i;
    int num[5]={1,2,3,4,5};
    int * ptr_num;
    ptr_num=num;
    for(i=0;i<5;i++)
    {
        printf("%d\n",*ptr_num++);
       //printf("%d\n",*(ptr_num++));
       //printf("%d\n",*(ptr_num)++);
       //printf("%d\n",(*ptr_num)++);
    }
    ```

    > 注意:
    >
    > 一个类型为T的指针的移动,以sizeof(T)为移动单位

  - 指针加上或减去某个整数值

    ```c
    int i;
    int num[5]={1,2,3,4,5};
    int * ptr_num;
    ptr_num=&num[1];
    ptr_num+=2;
    printf("%d\n",*ptr_num);
    ptr_num-=3;
    printf("%d\n",*ptr_num);
    ```

- 区分表达式

  - P++

    指向下一个地址单元

  - *P++

    间接引用p指向的数据单元，然后，p指向下一个地址单元

  - *(P++)

    同上（注意：虽然＋＋括在括号里，但由于＋＋在后边，因此仍然后加） 

  - *(++P)

    先将p指向下一个地址单元，然后间接引用p指向的数据单元

  - (*P)++

    P指向的数据+1

### 小结

- 数组名就是这块连续内存单元的首地址 
  - int num[50];//num是数组名，也是数组的首地址 
  - num的值与&num[0]的值是相同的 
  - 数组第i+1个元素可表示为： 
    - 第i+1个元素的地址：&num[i + 1] 或 num + i 
    - 第i+1个元素的值： num[i + 1]    或 \*(num + i + 1) 同样\*num++ 
  - 为指向数组的指针赋值： 
    - int * ptr_num = num; 或 int * ptr_num = &num[0]; 
  - 指针变量可以指向数组元素 
    - int * ptr_num = &num[4]; 或 int * ptr_num = num + 4;

#### 随堂练习

- 数组实现逆序

- 请说出下列程序输出结果

  ```c
  void main()
  {
      int  a[] = {1,2,3,4,5};
  	int  *p = NULL;
  	p = a;
  	printf("%d, ",*p);
  	printf("%d, ",*(++p));
  	printf("%d, ",*++p);
  	printf("%d, ",*(p--));
  	printf("%d, ",*p++);
  	printf("%d, ",*p);
  	printf("%d, ",++(*p));
  	printf("%d, ",*p);
  }
  ```

  


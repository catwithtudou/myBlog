---
title: C语言-循环(一)
date: 2019-10-19 15:42:21
tags: ["C"]
categories: "C"
---

# C语言-循环(一)

## 循环

- 复读机

  - 低级的复读机

    ```c
    printf("复读,第1\n");
    printf("复读,第2\n");
    printf("复读,第3\n");
    printf("复读,第4\n");
    printf("复读,第5\n");
    ```

  - 高级的复读机

    ```c
    for(int i=0;i<5;i++)
        printf("复读,第%d\n");
    ```

如果需要打印1000万句呢

- 低级的复读机

  ![](https://raw.githubusercontent.com/catwithtudou/photo/master/L3MRROKC%4013%5DWA%5D4KECTL%7EB.jpg)


所以,有了循环,你复读起来就更加美好了呢!

先来介绍简单的循环

### while循环

- 基本语法

```c
while(循环条件)
{
   循环操作语句   
}
```

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019153652.png)

#### 循环三要素

- 循环变量的初值
- 循环变量的判断
- 循环变量的更新

**先判断,再执行**

#### 随堂练习

- 需求说明

  使用循环模拟实现玩家PK对战

  - 双方初始HP均为100
  - 每次攻击5-15
  - HP最先到零或以下的被KO

```c
//提示
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
int main()
{
    //时间做种子
    srand((unsigned) time(NULL));
    //rand()取值范围:0-32767
    printf("%d\n",rand());
    return 0;
}
```

### do-while循环

- 基本语法

  ```c
  do{
      循环操作
  }while(循环条件);
  ```

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191019154313.png)

**先执行,再判断**

- 先执行一遍
- 符号条件,循环继续执行
- 否则循环退出

> 随堂练习
>
> ```c
> int a=1,b=10;
> 
> do{
> 
> b-=a;
> 
> a++;
> 
> }while(b--<0)
>  
> //b的值为多少    
> ```

#### 随堂练习

- 需求说明
  - 使用do-while结构打印菜单

### while和do-while区别

- 执行顺序不同
- 初始情况不满足循环条件时
  - while循环一次都不会执行
  - do-while(不管任何情况)至少执行一次

#### 随堂练习

- 需求说明
  - 使用不同的循环结构实现数字反转

## 参考

<https://study.163.com/course/courseMain.htm?courseId=1003425004&_trace_c_p_k2_=45b2a79c654549a89db231a73169f426>
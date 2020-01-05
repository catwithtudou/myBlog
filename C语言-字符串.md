---
title: C语言-字符串
date: 2019-12-08 19:01:25
tags: ["C"]
categories: "C"
---

# C语言-字符串

## 什么是字符串

- 字符串(character string)
  - 一个或多个字符的序列称为字符串 
  - C语言中形如 "My heart is still." 
  - 双引号不是字符串的一部分，仅用来告知编译器括起来的是字符串

>  注意： 
>
> 空字符不要和NULL混淆 
>
> 空字符是字符串的终止符，而NULL是一个 符号，表示不引用任何内容的内存地址

### C语言中的字符串 

- 使用字符数组存储

  ![](http://img.zhengyua.cn/img/20191208183307.png)

- 使用字符数组存放字符串

  ```c
  #include <stdio.h> 
  void main() 
  { 
      char name1[8] = {'J', 'a', 'c', 'k', 's', 'o', 'n', '\0'}; 
      char name2[]   = "Jackson"; 
      printf("%s\n", name1); 
      printf("%s\n", name2); 
  }
  ```

  ```c
  #include <stdio.h>
  void main() 
  { 
      char words[50]; 
      printf("请输入歌词："); //words是数组，不需要&符号 
      scanf("%s", words); 
      printf("%s\n", words); 
  }
  
  ```

  > 注意： 
  >
  > 声明存储字符串的数组时，数组大小至少比 所存储的字符数多1，因为编译器会自动在字 符串常量的末尾添加空字符\0
  >
  > 思考：
  >
  > 以上程序用户录入"My heart is still." 最终会输出怎样的结果？WHY?

### gets/puts函数补充

```c
#include <stdio.h>
void main() 
{ 
    char words1[50]; 
    char words2[50]; 
    printf("请输入歌词："); 
    gets(words1); 
    gets(words2); 
    puts("************\n"); 
    puts(words1); 
    puts(words2); 
}
```

> 注意： 
>
> 1、gets函数不对接受字符串的buffer进行边界 检测，会造成越界，从而产生bug 
>
> 2、可以使用fgets(words1, 20, stdin);代替 gets，20表示最多读入20-1个字符

### 随堂练习

需求说明： 

书写函数，返回传入字符串的长度 

分别使用gets函数和fgets函数

```c
void main() 
{ 
    char words[50]; 
    int len; printf("请输入歌词："); 
    fgets(words, 20, stdin); 
    len = myStrlen(words)； 
    printf("字符串长度：%d\n", len); 
}

int myStrlen(char str[]) 
{ 
    int count = 0;  //字符串长度 
    while(str[count] != '\0') 
    { 
        if(str[count] == '\n') 
        { 
            str[count] = '\0'; 
            break; 
        } 
        count++; 
    } 
    return count; 
}
```

### 小结

- 字符串由一个或多个字符的序列称为字符串 
- C语言中使用字符数组存储字符串 
- 所有的字符串都是字符数组，反之不成立 
- 三种方式录入字符串 
  - scanf(”%s”, str); 
  - gets(str); 
  - fgets(str, 50, stdin);

## 字符串操作

- 常用字符串处理函数 

  ![](http://img.zhengyua.cn/img/20191208183944.png)

  - 获得字符串的长度
  - 将字符串转换为大写
  - 将字符串转换为小写
  - 把第二个字符串复制到第一个字符数组中 
  - 根据ASCII码比较字符串
  - 字符串连接

### strlen函数

- 功能： 

  计算字符串的实际长度，不含字符串结束标志\0

  ```c
  #include <stdio.h>  //#include <string.h> 
  void main() 
  { 
      char words1[] = {'H', 'i', '\0', 'K','\0'}; 
      char words2[] = "Jackson"; 
      char words3[] = "你好，世界！"; 
      printf("words1长度为：%d\n", strlen(words1)); 
      printf("words2长度为：%d\n", strlen(words2)); 
      printf("words3长度为：%d\n", strlen(words3)); 
  }
  ```

### 字符串复制

- 功能： 

  把源字符数组中的字符串复制到目的字符数组中，连同结束标志\0一同复制

  ```c
  void main() 
  { 
      char str1[50]; 
      char str2[20]; 
      printf("输入目的字符串："); 
      gets(str1); 
      printf("输入源字符串："); 
      gets(str2); 
      strcpy(str1, str2);
      printf("****复制后****\n"); 
      printf("目的字符串：%s\n", str1); 
      printf("源字符串：%s\n", str2);
  }
  
  ```

### 字符串比较

- 功能： 

  - 将两个字符串从首字母开始，按照ASCII码的顺序进行逐个比较 
    - 字符串1 == 字符串2，返回0 
    - 字符串1  <  字符串2，返回正数 
    - 字符串1  >  字符串2，返回负数

  ```c
  #define USER_NAME "admin"
  #define PASSWORD  "admin“
  //…. 
  char userName[25], password[25]; 
  printf("用户名：");    gets(userName); 
  printf("密码：");       gets(password); if(login(userName, password))
  { 
      //调用登录成功后执行的函数 
      printf("登录成功！\n"); 
  }
  else
  { 
      printf("登录失败\n");
  }
  //验证传入的用户名密码是否跟默认值匹配
  int login(char userName[], char password[]) 
  { 
      if(strcmp(userName, USER_NAME) != 0 || strcmp(password, PASSWORD) != 0) 
      { 
          return 0;//用户名或密码不匹配就返回假 
      } 
      return 1; 
  }
  ```

### 字符串连接

- 功能： 

  将一个字符串连接到另一个字符串的末尾，组合成一个新字符串

  ```c
  void main() 
  { 
      char str1[50]; 
      char str2[20]; 
      printf("输入目的字符串："); 
      gets(str1); 
      printf("输入源字符串："); 
      gets(str2); 
      strcat(str1, str2);
      printf("****复制后****\n"); 
      printf("目的字符串：%s\n", str1); 
      printf("源字符串：%s\n", str2);
  }
  
  ```

### 随堂练习

- 需求说明： 
  - 实现字符串的加密与解密 
  - 加密方式：将字符串中每个字符加上它在字符串中的位置和一个偏移量5 
    - 例如：xuetang9中，第一个字符x在字符串中的位置为0，那么对应的密文是'm'+0+5

```c
//加密 
char* encrypt(char password[], char newPassword[])
{ 
    int i; 
    for(i = 0; i < strlen(password); i++)
    { 
        newPassword[i] = password[i] + i + KEY;
    } 
    newPassword[i] = '\0'; 
    return newPassword; 
}
```

## 指向字符串的指针

- 将指针指向字符串 
  - 可以指向常量字符串 
  - 也可以指向存储字符串的字符数组

![](http://img.zhengyua.cn/img/20191208184721.png)

## 数组和指针

- 数组形式和执行形式的不同 
  - 初始化字符数组时会把静态存储区的字符串拷贝到数组中
  - 初始化指针时只把字符串的地址拷贝给指针

```c
#include <stdio.h> 
void main() 
{ 
    char str[] = "For the Horde"; 
    char *ptr_str = "For the Horde"; 
    printf("字符串常量的地址：%p\n", "For the Horde"); 
    printf("字符数组的首地址：%p\n", str); 
    printf("字符指针的取值 %p\n", ptr_str); 
}
```

## 字符串数组与字符指针数组

- 字符串数组

  ```c
  char names1[5][10] = {"西施", "貂蝉", "王昭君", "杨玉环", "赵飞燕"};
  ```

- 字符指针数组

  ```c
  char  * names2[10 ] = {"西施", "貂蝉", "王昭君", "杨玉环", "赵飞燕"};
  ```

```c
printf("sizeof names1:%d\n", sizeof(names1)); printf("sizeof names2:%d\n", sizeof(names2));
```

## 字符串排序

```c
void str_sort(char *strArray[], int count)
{ 
    char * temp; 
    int i, j; 
    for(i = 0; i < count; i++)
    { 
        for(j = 0; j < count - i - 1; j++) 
        { 
            if(strcmp(strArray[j], strArray[j + 1]) > 0)
            { 
                temp = strArray[j]; 
                strArray[j] = strArray[j + 1];
                strArray[j + 1] = temp; 
            } 
        } 
    } 
}
```

## 总结

- 字符串与字符数组的区别是字符串的末尾有一个空字符'\0'以标识字符串结束 
- gets() 和 puts() 函数分别用于字符串的输入和输出 
- 在 string.h 中定义了很多字符串处理函数函数，比较常用的有 
  - strcpy()、strcat()、strcmp() 和 strlen() 

- 可以使用字符指针数组指向字符串常量（字面量） 
- 字符串可以作为参数，函数传递机制同数组作为参数，为引用方式
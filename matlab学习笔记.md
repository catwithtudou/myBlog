---
title: matlab学习笔记
date: 2019-06-23 16:19:46
tags: "matlab"
categories: "matlab"
---

### 矩阵和数组

#### 数组创建

> a = [1 2 3 4]
> a = [1 2 3; 4 5 6; 7 8 9]
> z = zeros(5,1)

#### 矩阵和数组运算

- 使用单一的算术运算符或函数来处理矩阵中的所有值

  > a+10
  >
  > sin(a)

- 转置矩阵

  > a'

- 使用 `*` 运算符执行标准矩阵乘法，这将计算行与列之间的内积

  >确认矩阵乘以其逆矩阵可返回单位矩阵
  >
  >p=a*inv(a)

- MATLAB 将数字存储为浮点值，算术运算可以区分实际值与其浮点表示之间的细微差别

  >`format` 命令可以显示更多小数位数
  >
  >format long
  >
  >将显示内容重置为更短格式
  >
  >format short
  >
  >`format` 仅影响数字显示，而不影响 MATLAB 对数字的计算或保存方式

- 执行元素级乘法（而非矩阵乘法），请使用 `.*` 运算符

  > p=a.*a

- 乘法、除法和幂的矩阵运算符分别具有执行元素级运算的对应数组运算符

  > 计算 `a` 的各个元素的三次方
  >
  > a.^3

#### 串联

- *串联*是连接数组以便形成更大数组的过程。实际上，第一个数组是通过将其各个元素串联起来而构成的。成对的方括号 `[]` 即为串联运算符

  > A=[a,a]
  >
  > ```matlab
  > A = 3×6
  > 
  >   1     2     3     1     2     3
  >   4     5     6     4     5     6
  >   7     8    10     7     8    10
  > ```

- 使用逗号将彼此相邻的数组串联起来称为*水平*串联。每个数组必须具有相同的行数。同样，如果各数组具有相同的列数，则可以使用分号*垂直*串联

  > A=[a; a]
  >
  > ```matlab
  > A = 6×3
  > 
  >   1     2     3
  >   4     5     6
  >   7     8    10
  >   1     2     3
  >   4     5     6
  >   7     8    10
  > 
  > ```

#### 复数

- 复数包含实部和虚部，虚数单位是 `-1` 的平方根

  > sqrt(-1)
  >
  > ans=0.0000+1.0000i

- 要表示复数的虚部，请使用 `i` 或 `j`

  > c=[3+4i,4+3j;-i,10j]

### 数组索引

- 每个变量都是一个可包含许多数字的数组。如果要访问数组的选定元素，请使用索引

  > 以 4×4 幻方矩阵 `A` 为例：
  >
  > A=magic(4)
  >
  > ```
  > A = 4×4
  > 
  >  16     2     3    13
  >   5    11    10     8
  >   9     7     6    12
  >   4    14    15     1
  > ```

- 引用数组中的特定元素有两种方法。最常见的方法是指定行和列下标

  > A(4,2)
  >
  > ans=14

- 使用单一下标按顺序向下遍历每一列

  > A(8)
  >
  > ans=14

- 使用单一下标引用数组中特定元素的方法称为*线性索引*

  如果尝试在赋值语句右侧引用数组外部元素，MATLAB 会引发错误

  不过，您可以在赋值语句左侧指定当前维外部的元素。数组大小会增大以便容纳新元素

  > A(4,5)=17
  >
  > ```
  > A = 4×5
  > 
  >  16     2     3    13     0
  >   5    11    10     8     0
  >   9     7     6    12     0
  >   4    14    15     1    17
  > ```

- 要引用多个数组元素，请使用冒号运算符，这使您可以指定一个格式为 `start:end` 的范围

  > 列出 `A` 前三行及第二列中的元素
  >
  > A(1:3,2)
  >
  > ```
  > ans = 3×1
  > 
  >   2
  >  11
  >   7
  > ```

- 单独的冒号（没有起始值或结束值）指定该维中的所有元素

  > 例如，选择 `A` 第三行中的所有列：
  >
  > A(3,:)

- 此外，冒号运算符还允许您使用较通用的格式 `start:step:end` 创建等距向量值

  > B=0:10:100
  >
  > ```
  > B = 1×11
  > 
  >   0    10    20    30    40    50    60    70    80    90   100
  > ```
  >
  > 如果省略中间的步骤（如 `start:end` 中），MATLAB 会使用默认步长值 `1`

### 工作区变量

***工作区*包含在 MATLAB® 中创建或从数据文件或其他程序导入的变量**

> 例如，下列语句在工作区中创建变量 `A` 和 `B`
>
> ```
> A = magic(4);
> B = rand(3,5,2);
> ```
>
> 使用 `whos` 可以查看工作区的内容
>
> ```
> whos
> ```
>
> 此外，桌面上的“工作区”窗格也会显示变量
>
> ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/workspace_zh_CN.png)

- 退出 MATLAB 后，工作区变量不会保留。使用 `save` 命令保存数据以供将来使用

  > ```
  > save myfile.mat
  > ```
  >
  > 通过保存，系统会使用 `.mat` 扩展名将工作区保存在当前工作文件夹中一个名为 MAT 文件的压缩文件中。
  >
  > 要清除工作区中的所有变量，请使用 `clear` 命令。
  >
  > 使用 `load` 将 MAT 文件中的数据还原到工作区
  >
  > ```
  > load myfile.mat
  > ```

### 文本和字符

- 当您处理文本时，将字符序列括在单引号中。可以将文本赋给变量

  > ```
  > myText='Hello,world';
  > ```

- 如果文本包含单引号，请在定义中使用两个单引号

  >```
  >otherText='You''re right'
  >```

- 与所有 MATLAB® 变量一样，`myText` 和 `otherText` 为数组。其*类*或数据类型为 `char`

  > ```
  > whos myText
  > ```
  >
  > ```
  > Name        Size            Bytes  Class    Attributes
  > 
  > myText      1x12               24  char        
  > ```

-  可以使用方括号串联字符数组，就像串联数值数组一样

  ```
  longText=[myText,'-',otherText]
  ```

  ```
  longText = 
  'Hello, world - You're right'
  ```

- 要将数值转换为字符，请使用 `num2str` 或 `int2str` 等函数

  ```
  f71;
  c=(f-32)/1.8;
  tempText=['Temperature is ',num2str(c),'C']
  ```

  ```
  tempText = 
  'Temperature is 21.6667C'
  ```

### 调用函数

**MATLAB® 提供了大量执行计算任务的函数。在其他编程语言中，函数等同于*子例程*或*方法***

- 调用函数时输入参数在括号内

  ```
  A=[1 3 5];
  max(A)
  
  ans=5
  ```

- 如果存在多个输入参数,请使用逗号加以分隔

  ```
  B=[10 6 4];
  max(A,B)
  
  ans=1x3
  10 6 5
  ```

- 通过将函数赋值给变量，返回该函数的输出

  ```
  maxA=max(A)
  ```

- 如果存在多个输出参数，请将其括在方括号中

  ```
  [maxA,location]=max(A)
  
  maxA=5
  location=3
  ```

- 将任何字符输入括在单引号中

  ```
  disp('hello world')
  
  hello world
  ```

- 要调用不需要任何输入且不会返回任何输出的函数，请只键入函数名称

  ```
  clc
  
  清空命令行窗口
  ```

### 二维图和三维图

#### 线图

- 要创建二维线图，请使用 `plot` 函数

  ```
  例如，绘制从 0 到 2π 之间的正弦函数值：
  x=0:pi/100:2*pi;
  y=sin(x);
  plot(x,y)
  ```

  ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/gs2dand3dplotsexample_01_zh_CN.png)

- 可以标记轴并添加标题

  ```
  xlabel('x')
  ylabel('sin(x)')
  title('Plot of the Sine Function')
  ```

  ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/gs2dand3dplotsexample_02_zh_CN.png)

- 通过向 `plot` 函数添加第三个输入参数，您可以使用红色虚线绘制相同的变量

  ```
  plot(x,y,'r--')
  ```

  ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/gs2dand3dplotsexample_03_zh_CN.png)

  `'r--'` 为*线条设定*。每个设定可包含表示线条颜色、样式和标记的字符。标记是在绘制的每个数据点上显示的符号，例如，`+`、`o` 或 `*`。例如，`'g:*'` 请求绘制使用 `*` 标记的绿色点线。

- 为第一幅绘图定义的标题和标签不再被用于当前的*图窗*窗口中。默认情况下，每次调用绘图函数、重置坐标区及其他元素以准备新绘图时，MATLAB® 都会清空图窗

  要将绘图添加到现有图窗中，请使用 `hold on`。在使用 `hold off` 或关闭窗口之前，当前图窗窗口中会显示所有绘图

  ```
  x=0:pi/100:2*pi;
  y=sin(x);
  plot(x,y)
  
  hold on
  
  y2=cos(x);
  plot(x,y2,'r:')
  legend('sin','cos')
  
  hold off	
  ```

  ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/gs2dand3dplotsexample_04_zh_CN.png)

#### 三维绘图

- 三维图通常显示一个由带两个变量的函数（即 *z = f (x,y*)）定义的曲面图。

  要计算 *z*，请首先使用 `meshgrid` 在此函数的域中创建一组 (*x,y*) 点

  ```
  [X,Y]=meshgrid(-2:.2:2);
  Z=X.*exp(-x.^2-Y.^2);
  surf(X,Y,Z)
  ```

  ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/gs2dand3dplotsexample_05_zh_CN.png)

  `surf` 函数及其伴随函数 `mesh` 以三维形式显示曲面图。`surf` 使用颜色显示曲面图的连接线和面。`mesh` 生成仅以颜色标记连接定义点的线条的线框曲面图

#### 子图

- 使用 `subplot` 函数可以在同一窗口的不同子区域显示多个绘图。

  `subplot` 的前两个输入表示每行和每列中的绘图数。第三个输入指定绘图是否处于活动状态

  ```
  例如，在图窗窗口的 2×2 网格中创建四个绘图
  
  t=0:pi/100:2*pi;
  [X,Y,Z]=cylinder(4*cos(t));
  subplot(2,2,1);mesh(X);title('X');
  subplot(2,2,2);mesh(Y);title('Y');
  subplot(2,2,3);mesh(Z);title('Z');
  subplot(2,2,4);mesh(X,Y,Z);title('X','Y','Z');
  
  ```

### 编程和脚本

***脚本*是最简单的一种 MATLAB® 程序。脚本是一个包含多行连续 MATLAB 命令和函数调用的扩展名为 `.m` 的文件。在命令行中键入脚本名称即可运行该脚本**

#### 示例脚本

- 要创建脚本,请使用 `edit` 命令

  ```
  edit plotrand
  ```

- 这会打开一个名为plotrand.m的空白文件

  ```matlab
  %输入一些绘制随机数据的向量的代码：
  n= 50;
  r= rand(n,1);
  plot(r)
  
  %然后，添加在绘图中的均值处绘制一条水平线的代码
  m=mean(r);
  hold on;
  plot([0,n],[m,m])
  hold off
  titile('Mean of Random Uniform Data')
  ```

- 编写代码时，最好添加描述代码的注释。注释有助于其他人员理解您的代码，并且有助您在稍后返回代码时再度记起。使用百分比 (`%`) 符号添加注释。

- 将文件保存在当前文件夹中。要运行脚本，请在命令行中键入脚本名称

  ```matlab
  plotrand
  ```

  还可以从编辑器通过按**运行**按钮 ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/run_ts_16_zh_CN.png) 运行脚本

#### 循环及条件语句

- 在脚本中，可以使用关键字 `for`、`while`、`if` 和 `switch` 循环并有条件地执行代码段

  ```matlab
  %例如，创建一个名为 calcmean.m 的脚本，该脚本使用 for 循环来计算 5 个随机样本的均值和总均值
  
  nsamples=5;
  npoints=50;
  
  for k=1:nsamples
      cirrentData= rand (npoints,1);
      sampleMean(k)= mean(currentData);
  end
  overallMean=mean(sampleMean)
  
  %现在，修改 for 循环，以便在每次迭代时查看结果。在命令行窗口中显示包含当前迭代次数的文本，并从 sampleMean 的赋值中删除分号
  
  for k=1:nsamples
      iterationString=['Iteration #',ins2str(k)];
      disp(iterationString)
      currentData=rand(npoints,1);
      sampleMean(k)=mean(currentData);
  end
  overallMean = mean(sampleMean)
  ```

  ```matlab
  Iteration #1
  
  sampleMean =
  
      0.3988
  
  Iteration #2
  
  sampleMean =
  
      0.3988    0.4950
  
  Iteration #3
  
  sampleMean =
  
      0.3988    0.4950    0.5365
  
  Iteration #4
  
  sampleMean =
  
      0.3988    0.4950    0.5365    0.4870
  
  Iteration #5
  
  sampleMean =
  
      0.3988    0.4950    0.5365    0.4870    0.5501
  
  
  overallMean =
  
      0.4935
  ```

- 在编辑器中，在 `calcmean.m` 末尾添加根据 `overallMean` 的值显示不同消息的条件语句

  ```matlab
  if overallMean < .49
     disp('Mean is less than expected')
  elseif overallMean > .51
     disp('Mean is greater than expected')
  else
     disp('Mean is within the expected range')
  end
  ```

#### 脚本位置

MATLAB 在特定位置中查找脚本及其他文件。要运行脚本，该文件必须位于当前文件夹或*搜索路径*中的某个文件夹内。

默认情况下，MATLAB 安装程序创建的 `MATLAB` 文件夹位于此搜索路径中。如果要将程序存储在其他文件夹，或者要运行其他文件夹中的程序，请将其添加到此搜索路径。在当前文件夹浏览器中选中相应的文件夹，右键点击，然后选择**添加到路径**

### 帮助和文档

**所有 MATLAB® 函数都有辅助文档，这些文档包含一些示例，并介绍函数输入、输出和调用语法。从命令行访问此信息有多种方法**

- 使用 `doc` 命令在单独的窗口中打开函数文档

  ```
  doc mean
  ```

- 在键入函数输入参数的左括号之后暂停，此时命令行窗口中会显示相应函数的提示（函数文档的语法部分）

  ```matlab
  mean(
  ```

- 使用 `help` 命令可在命令行窗口中查看相应函数的简明文档

  ```matlab
  help mean
  ```

### 矩阵和幻方矩阵

#### 关于矩阵

**在 MATLAB® 环境中，矩阵是由数字组成的矩形数组。有时，1×1 矩阵（即标量）和只包含一行或一列的矩阵（即向量）会附加特殊含义。MATLAB 采用其他方法来存储数值数据和非数值数据，但刚开始时，通常最好将一切内容都视为矩阵。MATLAB 旨在尽可能简化运算。其他编程语言一次只能处理一个数字，而 MATLAB 允许您轻松快捷地处理整个矩阵**

#### 输入矩阵

开始学习 MATLAB 的最佳方法是了解如何处理矩阵。启动 MATLAB 并按照每个示例操作。

您可以采用多种不同方法在 MATLAB 中输入矩阵：

- 输入元素的明确列表。
- 从外部数据文件加载矩阵。
- 使用内置函数生成矩阵。
- 使用您自己的函数创建矩阵，并将其保存在文件中。

首先，以元素列表的形式输入丢勒的矩阵。您只需遵循一些基本约定：

- 使用空格或逗号分隔行的元素。
- 使用分号 `;` 表示每行末尾。
- 使用方括号 `[ ]` 将整个元素列表括起来。

要输入丢勒矩阵，只需在命令行窗口中键入即可

```
A = [16 3 2 13; 5 10 11 8; 9 6 7 12; 4 15 14 1]
```

MATLAB 显示刚才您输入的矩阵：

```
A = 
    16     3     2    13
     5    10    11     8
     9     6     7    12
     4    15    14     1
```

此矩阵与雕刻中的数字一致。输入矩阵之后，MATLAB 工作区会自动记住此矩阵。您可以将其简称为 `A`。现在，您已经在工作区中输入 `A`，让我们看看它为什么如此有趣吧。它有什么神奇的地方呢？

#### 矩阵求和,转置和对角矩阵

您可能已经注意到，幻方矩阵的特殊属性与元素的不同求和方法相关。如果沿任何行或列求和，或者沿两条主对角线中的任意一条求和，您将始终得到相同数字

```matlab
sum(A)

ans =
  34 34 34 34

%MATLAB 会优先处理矩阵的列，因此获取行总和的一种方法是转置矩阵，计算转置的列总和，然后转置结果

%MATLAB 具有两个转置运算符。撇号运算符（例如，A'）执行复共轭转置。它会围绕主对角线翻转矩阵，并且还会更改矩阵的任何复数元素的虚部符号。点撇号运算符 (A.') 转置矩阵，但不会影响复数元素的符号。对于包含所有实数元素的矩阵，这两个运算符返回相同结果。

A'

ans =
    16     5     9     4
     3    10     6    15
     2    11     7    14
    13     8    12     1

sum(A')'

%生成包含行总和的列向量
ans =
    34
    34
    34
    34

%有关避免双重转置的其他方法，请在 sum 函数中使用维度参数：

sum(A,2)

ans =
    34
    34
    34
    34

%使用 sum 和 diag 函数可以获取主对角线上的元素的总和
diag(A)

ans =
    16
    10
     7
     1

sum(diag(A))

ans =
   34

%从数学上讲，另一条对角线（即所谓的反对角线）并不是十分重要，因此 MATLAB 没有对此提供现成的函数。但原本用于图形的函数 fliplr 可以从左往右地翻转矩阵

sum(diag(fliplr(A)))
ans =
    34
    
```

#### magic函数

**MATLAB 实际包含一个内置函数，该函数可创建几乎任意大小的幻方矩阵。此函数称为 [`magic`](https://ww2.mathworks.cn/help/matlab/ref/magic.html) 也就不足为奇了：**

```matlab
B = magic(4)
B = 
    16     2     3    13
     5    11    10     8
     9     7     6    12
     4    14    15     1
%此矩阵几乎与丢勒雕刻中的矩阵相同，并且具有所有相同的“神奇”性质；唯一区别在于交换了中间两列

%您可以交换 B 的中间两列，使其看起来像丢勒 A。针对 B 中的每一行，按照指定顺序（1、3、2、4）对列进行重新排列：

A = B(:,[1 3 2 4])
A = 
    16     3     2    13
     5    10    11     8
     9     6     7    12
     4    15    14     1
```

#### 生成矩阵

MATLAB 软件提供了四个用于生成基本矩阵的函数。

| [`zeros`](https://ww2.mathworks.cn/help/matlab/ref/zeros.html) | 全部为零           |
| ------------------------------------------------------------ | ------------------ |
| [`ones`](https://ww2.mathworks.cn/help/matlab/ref/ones.html) | 全部为 1           |
| [`rand`](https://ww2.mathworks.cn/help/matlab/ref/rand.html) | 均匀分布的随机元素 |
| [`randn`](https://ww2.mathworks.cn/help/matlab/ref/randn.html) | 正态分布的随机元素 |

下面给出了一些示例：

```matlab
Z = zeros(2,4)
Z =
     0     0     0     0
     0     0     0     0

F = 5*ones(3,3)
F =
     5     5     5
     5     5     5
     5     5     5

N = fix(10*rand(1,10))
N =
     9     2     6     4     8     7     4     0     8     4

R = randn(4,4)
R =
    0.6353    0.0860   -0.3210   -1.2316
   -0.6014   -2.0046    1.2366    1.0556
    0.5512   -0.4931   -0.6313   -0.1132
   -1.0998    0.4620   -2.3252    0.3792
```



### 表达式

#### 变量

- MATLAB 不需要任何类型声明或维度说明。当 MATLAB 遇到新的变量名称时，它会自动创建变量，并分配适当大小的存储。如果此变量已存在，MATLAB 会更改其内容，并根据需要分配新存储.

  这些表达式涉及整个矩阵

  ```matlab
  %创建一个名为 num_students 的 1×1 矩阵，并将值 25 存储在该矩阵的单一元素中
  num_students= 25
  
  %要查看分配给任何变量的矩阵，只需输入变量名称即可
  
  %MATLAB 区分大小写
  
  %尽管变量名称可以为任意长度，MATLAB 仅使用名称的前 N 个字符（其中 N 是函数 namelengthmax 返回的数字），并忽略其余字符
  
  %应使每个变量名称的前 N 个字符保持唯一，以便 MATLAB 能够区分变量
  ```

#### 数字

**MATLAB 使用传统的十进制记数法以及可选的小数点和前导加号或减号来表示数字。科学记数法使用字母 `e` 来指定 10 次方的缩放因子。虚数使用 `i` 或 `j` 作为后缀**

```asciiarmor
合法数字案列:
3              -99            0.0001
9.6397238      1.60210e-20    6.02252e23
1i             -3.14159j      3e5i
```

- MATLAB 使用 IEEE® 浮点标准规定的 *long* 格式在内部存储所有数字。浮点数的有限*精度*约为 16 位有效小数位数，有限*范围*约为 10-308 至 10+308

- 以双精度格式表示的数字的最大精度为 52 位。任何需要 52 位以上的双精度数字都会丢失一定精度

  ```matlab
  x = 36028797018963968;
  y = 36028797018963972;
  x == y
  ans =
        1
  ```

  

- 整数的可用精度为 8 位、16 位、32 位和 64 位

  ```matlab
  %将相同数字存储为 64 位整数会保留精度
  x = uint64(36028797018963968);
  y = uint64(36028797018963972);
  x == y
  ans =
        0
  ```

- MATLAB 软件存储复数的实部和虚部。该软件根据上下文采用不同方法来处理各个部分的量值

  ```matlab
  %例如，sort 函数根据量值进行排序，如果量值相等，则根据相位角度排序。
  
  sort([3+4j,4+3j])
  ans = 
     4.0000+ 3.0000i    3.0000+ 4.0000i
  
  %这是由相位角度所致
  angle(3+4i)
  ans = 0.9273
  angle(4+3i)
  ans= 0.6435
     
  ```

#### 矩阵运算符

| `+`   | 加法         |
| ----- | ------------ |
| -     | 减法         |
| `*`   | 乘法         |
| `/`   | 除法         |
| `\`   | 左除         |
| `^`   | 幂           |
| `'`   | 复共轭转置   |
| `( )` | 指定计算顺序 |

#### 数组运算符

| `+`  | 加法           |
| ---- | -------------- |
| `-`  | 减法           |
| `.*` | 逐元素乘法     |
| `./` | 逐元素除法     |
| `.\` | 逐元素左除     |
| `.^` | 逐元素幂       |
| `.'` | 非共轭数组转置 |

```matlab
%构建表
n=(0:9)';
pows=[n n.^2 2.^n]
%构建一个平方和2次幂的表:
pows =
     0     0     1
     1     1     2
     2     4     4
     3     9     8
     4    16    16
     5    25    32
     6    36    64
     7    49   128
     8    64   256
     9    81   512
     
%初等数学函数逐元素处理数组元素.因此

format short g
x=(1:0.1:2)';
logs=[x log10(x)]

%构建一个对数表
logs =
      1.0            0 
      1.1      0.04139
      1.2      0.07918
      1.3      0.11394
      1.4      0.14613
      1.5      0.17609
      1.6      0.20412
      1.7      0.23045
      1.8      0.25527
      1.9      0.27875
      2.0      0.30103
      
```

#### 函数

**MATLAB 提供了大量标准初等数学函数，包括 [`abs`](https://ww2.mathworks.cn/help/matlab/ref/abs.html)、[`sqrt`](https://ww2.mathworks.cn/help/matlab/ref/sqrt.html)、[`exp`](https://ww2.mathworks.cn/help/matlab/ref/exp.html) 和 [`sin`](https://ww2.mathworks.cn/help/matlab/ref/sin.html)。生成负数的平方根或对数不会导致错误；系统会自动生成相应的复数结果。MATLAB 还提供了许多其他高等数学函数，包括贝塞尔函数和伽玛函数。其中的大多数函数都接受复数参数。**

- 有关初等数学函数的列表，请键入

```
help elfun
```

- 有关更多高等数学函数和矩阵函数的列表，请键入

```
help specfun
help elmat
```

- 一些特殊函数提供了有用的常量值

  | [`pi`](https://ww2.mathworks.cn/help/matlab/ref/pi.html)     | 3.14159265...           |
  | ------------------------------------------------------------ | ----------------------- |
  | [`i`](https://ww2.mathworks.cn/help/matlab/ref/i.html)       | 虚数单位 G−1            |
  | [`j`](https://ww2.mathworks.cn/help/matlab/ref/j.html)       | 与 `i` 相同             |
  | [`eps`](https://ww2.mathworks.cn/help/matlab/ref/eps.html)   | 浮点相对精度 *ε*=2−52   |
  | [`realmin`](https://ww2.mathworks.cn/help/matlab/ref/realmin.html) | 最小浮点数 2−1022       |
  | [`realmax`](https://ww2.mathworks.cn/help/matlab/ref/realmax.html) | 最大浮点数 (2−*ε*)21023 |
  | [`Inf`](https://ww2.mathworks.cn/help/matlab/ref/inf.html)   | 无穷大                  |
  | [`NaN`](https://ww2.mathworks.cn/help/matlab/ref/nan.html)   | 非数字                  |

- 通过将非零值除以零或计算明确定义的溢出的数学表达式(超过realmax),会生成无穷大.通过尝试计算 `0/0` 或 `Inf`-`Inf` 等没有明确定义的数值的表达式，会生成非数字。

- 函数名称不会保留,可使用如下新变量覆盖任何函数名称

  eps = 1.e-6

  并在后续计算中使用该值.可使用以下命令恢复原始函数

  clear eps

#### 表达式示例

```matlab
rho = (1+sqrt(5))/2
rho =
    1.6180

a = abs(3+4i)
a =
     5

z = sqrt(besselk(4/3,rho-i))
z =
   0.3730+ 0.3214i

huge = exp(log(realmax))
huge =
  1.7977e+308

toobig = pi*huge
toobig =
   Inf
```



### 输入命令

#### format 函数

**[`format`](https://ww2.mathworks.cn/help/matlab/ref/format.html) 函数控制所显示的值的数值格式。此函数仅影响数字显示方式，而不会影响 MATLAB® 软件如何计算或保存数字**

```matlab
x = [4/3 1.2345e-6]

format short

   1.3333    0.0000

format short e

   1.3333e+000  1.2345e-006

format short g

   1.3333  1.2345e-006

format long

   1.33333333333333   0.00000123450000

format long e

   1.333333333333333e+000    1.234500000000000e-006

format long g

   1.33333333333333               1.2345e-006

format bank

   1.33          0.00

format rat

   4/3          1/810045

format hex
 
   3ff5555555555555   3eb4b6231abfd271
```



如果矩阵的最大元素大于 103 或小于 10-3，MATLAB 会对短格式和长格式应用常用缩放因子。

除了上面显示的 `format` 函数，

```
format compact
```

会不显示在输出中出现的多个空行。这样，您可以在屏幕或窗口中查看更多信息。如果要进一步控制输出格式，请使用 [`sprintf`](https://ww2.mathworks.cn/help/matlab/ref/sprintf.html) 和 [`fprintf`](https://ww2.mathworks.cn/help/matlab/ref/fprintf.html) 函数。

#### 取消输出

**如果您在仅键入语句后按 Return 或 **Enter**，MATLAB 会在屏幕上自动显示结果。但是，如果使用分号结束行，MATLAB 会执行计算，但不会显示任何输出**

#### 输入长语句

**如果语句无法容纳在一行中，请使用省略号（三个句点）`...`，后跟 Return 或 Enter 以指示该语句在下一行继续**

```
s = 1 -1/2 + 1/3 -1/4 + 1/5 - 1/6 + 1/7 ...
      - 1/8 + 1/9 - 1/10 + 1/11 - 1/12;
```

#### 命令行编辑

使用键盘上的各个箭头键和控制键可以重新调用、编辑和重用先前键入的语句

您只需按 ↑ 键，而不必重新键入整行。系统将重新显示键入的语句。使用 ← 键移动光标并插入缺少的 `r`。反复使用 ↑ 键可重新调用前面的行。键入几个字符并按 ↑ 键可查找前文中以这些字符开头行。还可以从命令历史记录中复制以前执行的语句

### 索引

#### 下标

**`A` 的行 `i` 和列 `j` 中的元素通过 `A(i,j)` 表示**

```matlab
A(1,4) + A(2,4) + A(3,4) + A(4,4)

ans =
     34
%但这不是计算某列总和的最佳方法

%此外，还可以使用单一下标 A(k) 引用矩阵的元素。单一下标是引用行和列向量的常见方法。但是，也可以对满二维矩阵应用单一下标。在这种情况下，数组被视为一个由原始矩阵的列构成的长列向量

%如果尝试使用矩阵外部元素的值，则会生成错误

%相反，如果将值存储在矩阵外部元素中，则会增大大小以便容纳新元素

X = A;
X(4,5) = 17

X =
    16     3     2    13     0
     5    10    11     8     0
     9     6     7    12     0
     4    15    14     1    17
```

#### 冒号运算符

```matlab
1:10
%是包含从 1 到 10 之间的整数的行向量：

1     2     3     4     5     6     7     8     9    10
%要获取非单位间距，请指定增量。例如，

100:-7:50
%为

100    93    86    79    72    65    58    51
%而

0:pi/4:pi
%为

0    0.7854    1.5708    2.3562    3.1416
%包含冒号的下标表达式引用部分矩阵：

A(1:k,j)
%表示 A 第 j 列中的前 k 个元素。因此，

sum(A(1:4,4))
%计算第四列的总和。但是，执行此计算有一种更好的方法。冒号本身引用矩阵行或列中的所有元素，而关键字 end 引用最后一个行或列。因此，

sum(A(:,end))
%计算 A 最后一列中的元素的总和：

ans =
     34
%为什么 4×4 幻方矩阵的幻数和等于 34？如果将介于 1 到 16 之间的整数分为四个总和相等的组，该总和必须为

sum(1:16)/4
%当然，也即

ans =
     34
     
```

#### 串联

***串联*是连接小矩阵以便形成更大矩阵的过程。实际上，第一个矩阵是通过将其各个元素串联起来而构成的。成对的方括号 `[]` 即为串联运算**



```matlab
%例如，从 4×4 幻方矩阵 `A` 开始，组成
B = [A  A+32; A+48  A+16]
B =

    16     3     2    13    48    35    34    45
     5    10    11     8    37    42    43    40
     9     6     7    12    41    38    39    44
     4    15    14     1    36    47    46    33
    64    51    50    61    32    19    18    29
    53    58    59    56    21    26    27    24
    57    54    55    60    25    22    23    28
    52    63    62    49    20    31    30    17
```

#### 删除行和列

- 只需使用一对方括号即可从矩阵中删除行和列

  ```matlab
  %删除第二列
  X=A;
  X(:,2)=[]
  
  %如果您删除矩阵中的单个元素，结果将不再是矩阵
  X(1,2) = [] %将会导致错误
  
  %使用单一下标可以删除一个元素或元素序列，并将其余元素重构为一个行向量。因此
  
  X(2:2:10) = []
  %生成
  
  X =
      16     9     2     7    13    12     1
  ```

#### 标量扩展

可以采用多种不同方法将矩阵和标量合并在一起

```matlab
%例如，通过从每个元素中减去标量而将其从矩阵中减去。幻方矩阵的元素平均值为 8.5，因此

B = A - 8.5

%形成一个列总和为零的矩阵：
B =
      7.5     -5.5     -6.5      4.5
     -3.5      1.5      2.5     -0.5
      0.5     -2.5     -1.5      3.5
     -4.5      6.5      5.5     -7.5

sum(B)

ans =
     0     0     0     0
```

- 通过标量扩展，MATLAB 会为范围中的所有索引分配一个指定标量

  ```matlab
  B(1:2,2:3) = 0
  %将 B 的某个部分清零：
  
  B =
        7.5        0        0      4.5
       -3.5        0        0     -0.5
        0.5     -2.5     -1.5      3.5
       -4.5      6.5      5.5     -7.5
  ```

#### 逻辑下标

根据逻辑和关系运算创建的逻辑向量可用于引用子数组

```matlab
%假定 X 是一个普通矩阵，L 是一个由某个逻辑运算生成的同等大小的矩阵。那么，X(L) 指定 X 的元素，其中 L 的元素为非零。

%通过将逻辑运算指定为下标表达式，可以在一个步骤中完成这种下标。假定您具有以下数据集：

x = [2.1 1.7 1.6 1.5 NaN 1.9 1.8 1.5 5.1 1.8 1.4 2.2 1.6 1.8];

%NaN 是用于缺少的观测值的标记，例如，无法响应问卷中的某个项。要使用逻辑索引删除缺少的数据，请使用 isfinite(x)，对于所有有限数值，该函数为 true；对于 NaN 和 Inf，该函数为 false：

x = x(isfinite(x))
x =
  2.1 1.7 1.6 1.5 1.9 1.8 1.5 5.1 1.8 1.4 2.2 1.6 1.8
  
%现在，存在一个似乎与其他项很不一样的观测值，即 5.1。这是一个离群值。下面的语句可删除离群值，在本示例中，即比均值大三倍标准差的元素：

x = x(abs(x-mean(x)) <= 3*std(x))
x =
  2.1 1.7 1.6 1.5 1.9 1.8 1.5 1.8 1.4 2.2 1.6 1.8
  
```

- 标量扩展对于另一示例，请使用逻辑索引和标量扩展将非质数设置为 0，以便高亮显示丢勒幻方矩阵中的质数的位置

  ```matlab
  A(~isprime(A)) = 0
  
  A =
       0     3     2    13
       5     0    11     0
       0     0     7     0
       0     0     0     0
  ```

#### find函数

**[`find`](https://ww2.mathworks.cn/help/matlab/ref/find.html) 函数可用于确定与指定逻辑条件相符的数组元素的索引。`find` 以最简单的形式返回索引的列向量。转置该向量以便获取索引的行向量**

```matlab
%例如，再次从丢勒的幻方矩阵开始。（请参阅 magic 函数。）
k = find(isprime(A))'
%使用一维索引选取幻方矩阵中的质数的位置：
k =
 2     5     9    10    11    13

%使用以下命令按 k 确定的顺序将这些质数显示为行向量
A(k)
ans =
     5     3     2    11     7    13     

%将 k 用作赋值语句的左侧索引时，会保留矩阵结构：
A(k) = NaN

A =
    16   NaN   NaN   NaN
   NaN    10   NaN     8
     9     6   NaN    12
     4    15    14     1    
```

### 数组类型

#### 多维数组

MATLAB® 环境中的多维数组是具有多个下标的数组。创建多维数组的一种方法是调用具有多个参数的 [`zeros`](https://ww2.mathworks.cn/help/matlab/ref/zeros.html)、[`ones`](https://ww2.mathworks.cn/help/matlab/ref/ones.html)、[`rand`](https://ww2.mathworks.cn/help/matlab/ref/rand.html) 或 [`randn`](https://ww2.mathworks.cn/help/matlab/ref/randn.html)。

```matlab
R = randon(3,4,5);
%创建一个 3×4×5 数组，共包含 3*4*5 = 60 个正态分布的随机元素
```

- 三维数组可表示在矩形网格中采样的三维物理数据，例如室内温度。或者也可以表示矩阵序列 *A*(k) 或与时间相关的矩阵示例 *A*(*t*)。

  ```matlab
  %MATLAB 与丢勒的 4 阶幻方矩阵版本的区别在于交换了两个列。通过交换列，可以生成许多不同的幻方矩阵。
  p = perms(1:4);
  %生成 4! = 24 置换1:4。第 k 个置换为行向量 p(k,:)。然后，
  A = magic(4);
  M = zeros(4,4,24);
  for k = 1:24
     M(:,:,k)= A(:,p(k,:));
     end
  %将含有 24 个幻方矩阵的序列存储在三维数组 M 中。M 大小为
  size(M)
  
  ans =
       4     4    24
  ```

  

![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/ch5pgmgmatlab3_zh_CN.gif)

```matlab
sum(M,d)

%通过改变第 d 个下标来计算总和。因此
sum(M,1)

%是一个含有 24 个行向量副本的 1×4×24 数组
34    34    34    34

sum(M,2)

%是一个含有 24 个列向量副本的 4×1×24 数组

34    
34    
34    
34

S = sum(M,3)
%在序列中添加 24 个矩阵。结果的大小为 4×4×1，因此它看似是 4×4 数组：
S =
   204   204   204   204
   204   204   204   204
   204   204   204   204
   204   204   204   204
```

#### 元胞数组

**MATLAB 中的元胞数组是以其他数组的副本为元素的多维数组。使用 [`cell`](https://ww2.mathworks.cn/help/matlab/ref/cell.html) 函数可以创建空矩阵的元胞数组。但是，更普遍的做法是，通过将其他内容的集合括入花括号 `{}` 中来创建元胞数组。花括号还可以与下标配合使用，以便访问各个元胞的内容**

```matlab
C={A sum(A) prod(prod(A))}

%生成一个 1×3 元胞数组。这三个元胞包含幻方矩阵、列总和的行向量及其所有元素的乘积。当显示 C 时，您会看到

C = 
    [4x4 double]    [1x4 double]    [20922789888000]
    
    %这是因为前两个元胞太大，无法在此有限空间中输出，但第三个元胞仅包含 16! 一个数字，因此有空间可以输出此数字。
    
    
```

- 请牢记以下两大要点。第一，要检索某个元胞的内容，请在花括号中使用下标。例如，`C{1}` 检索幻方矩阵，`C{3}` 为 16!。第二，元胞数组包含其他数组的*副本*，而不包含指向这些数组的*指针*。

  ```matlab
  %使用三维数组可以存储相同大小的矩阵序列。元胞数组可用于存储不同大小的矩阵序列。例如
  
  M = cell(8,1);
  for n = 1:8
     M{n} = magic(n);
  end
  M
  %生成具有不同顺序的幻方矩阵序列
  M = 
      [           1]
      [ 2x2  double]
      [ 3x3  double]
      [ 4x4  double]
      [ 5x5  double]
      [ 6x6  double]
      [ 7x7  double]
      [ 8x8  double]
      
  %使用以下命令可以检索 4×4 幻方矩阵
  
  M{4}
  ```

  ![img](https://ww2.mathworks.cn/help/matlab/learn_matlab/ch5pgmgmatlaba_zh_CN.gif)

#### 字符与文本

- 使用单引号在 MATLAB 中输入文本

  ```matlab
  s='Hello'
  %一个 1×5 字符数组
  
  ```

- 字符在内部作为数字存储，而不会采用浮点格式存储

  ```matlab
  a = double(s)
  
  %将字符数组转换为数值矩阵，该矩阵包含每个字符的 ASCII 代码的浮点表示。结果为
  
  a =
      72    101    108    108    111
      
  %语句
  
  s = char(a)
  %是刚才转换的逆转换。
  
  %将数字转换为字符可以调查计算机上的各种可用字体。基本 ASCII 字符集中的可打印字符由整数 32:127 表示。（小于 32 的整数表示不可打印的控制字符。）这些整数使用以下语句按相应的 6×16 数组的形式排列
  
  F = reshape(32:127,16,6)';
  ```

- 扩展 ASCII 字符集中的可打印字符由 `F+128` 表示。将这些整数解释为字符时，结果取决于当前使用的字体。键入语句

  ```
  char(F)
  char(F+128)
  ```

  然后改变命令行窗口所用的字体。要更改字体，请在**主页**选项卡上的**环境**部分中，点击**预设** > **字体**。如果代码行中包含制表符，请使用等宽字体（例如，`Monospaced`）以便在不同行中对齐制表符位置。

- ```matlab
  %使用方括号进行串联可将文本变量联接到一起
  h = [s, ' world']
  h =
     Hello world
     
     
  v =  [s; 'world']
  v =
     Hello
     world
     
  %请注意，必须在 h 中的 'w' 前插入一个空格，并且 v 中的两个单词的长度必须相同。生成的数组均为字符数组；h 为 1×11，v 为 2×5。   
  ```

- 要操作包含不同长度的行的文本主体，您有两种选择，即使用填充的字符数组或使用字符向量元胞数组。创建字符数组时，数组各行的长度必须相同。（使用空格填充较短行的末尾。）[`char`](https://ww2.mathworks.cn/help/matlab/ref/char.html) 函数可执行这种填充操作

  ```matlab
  S = char('A','rolling','stone','gathers','momentum.')
  
  %生成一个 5×9 字符数组：
  S =
  A        
  rolling  
  stone    
  gathers  
  momentum.
  
  %再者，您也可以将文本存储在元胞数组中。例如，
  
  C = {'A';'rolling';'stone';'gathers';'momentum.'}
  
  %创建一个不需要任何填充的 5×1 元胞数组，因为该数组的各行可以具有不同的长度：
  
  C = 
      'A'
      'rolling'
      'stone'
      'gathers'
      'momentum.'
  %使用以下语句可以将填充后的字符数组转换为字符向量元胞数组：
  
  C = cellstr(S)
  %使用以下语句可以逆转此过程
  
  S = char(C)    
  ```

  

#### 结构体

- 结构体是多维 MATLAB 数组，包含可按文本*字段标志符*访问的元素

  ```matlab
  S.name = 'Ed Plum';
  S.score = 83;
  S.grade = 'B+'
  %创建一个具有三个字段的标量结构体：
  
  S = 
       name: 'Ed Plum'
      score: 83
      grade: 'B+'
  ```

- 与 MATLAB 环境中的所有其他内容一样，结构体也为数组，因此可以插入其他元素

  ```matlab
  %在本示例中，数组的每个元素都是一个具有若干字段的结构体。可以一次添加一个字段，
  
  S(2).name = 'Toni Miller';
  S(2).score = 91;
  S(2).grade = 'A-';
  
  %也可以使用一个语句添加整个元素：
  
  S(3) = struct('name','Jerry Garcia',... 
                 'score',70,'grade','C')
                 
  %现在，结构体非常大以致仅输出摘要：
  
  S = 
  1x3 struct array with fields:
      name
      score
      grade
      
  ```

  

- 将不同字段重新组合为其他 MATLAB 数组的方法有许多种。这些方法大多基于*逗号分隔列表*的表示法

  ```matlab
  键入
  
  S.score
  
  与键入
  
  S(1).score, S(2).score, S(3).score
  
  %相同，这是一个逗号分隔列表。
  ```

- 如果将生成此类列表的表达式括在方括号中，MATLAB 会将该列表中的每一项都存储在数组中

  ```matlab
  %在本示例中，MATLAB 创建一个数值行向量，该向量包含结构体数组 S 的每个元素的 score 字段：
  
  scores = [S.score]
  scores =
      83    91    70
  
  avg_score = sum(scores)/length(scores)
  avg_score =
     81.3333
  
  %要根据某个文本字段（例如，name）创建字符数组，请对 S.name 生成的逗号分隔列表调用 char 函数
  
  names = char(S.name)
  names =
     Ed Plum    
     Toni Miller
     Jerry Garcia
     
  %同样，通过将生成列表的表达式括入花括号中，可以根据 name 字段创建元胞数组：
  
  names = {S.name}
  names = 
      'Ed Plum'    'Toni Miller'    'Jerry Garcia'   
  ```

- 要将结构体数组的每个元素的字段赋值给结构体外部的单独变量，请指定等号左侧的每个输出，并将其全部括在方括号中：

  ```matlab
  [N1 N2 N3] = S.name
  N1 =
     Ed Plum
  N2 =
     Toni Miller
  N3 =
     Jerry Garcia
  ```

#### 动态字段名称

**访问结构体中的数据的最常用方法是指定要引用的字段的名称。访问结构体数据的另一种方法是使用动态字段名称。这些名称将字段表示为变量表达式，MATLAB 会在运行时计算这些表达式**

```matlab
%此处显示的点-括号语法将 expression 作为动态字段名称：

structName.(expression)
%使用标准 MATLAB 索引语法创建此字段的索引。例如，要在字段名称中计算 expression，并在行 7 中的 1 至 25 列内获取该字段的值，请使用

structName.(expression)(7,1:25)
```

```matlab
function avg=avgscore(testscores,student,first,last)
for k=first:last
    scores(k)=testscores.(student).week(k);
    end
avg= sum(scores)/(last-first+1);

%您可以运行此函数，并对动态字段 student 使用不同值。首先，对包含 25 周内的分数的结构体进行初始化：

testscores.Ann_Lane.week(1:25) = ...
  [95 89 76 82 79 92 94 92 89 81 75 93 ...
   85 84 83 86 85 90 82 82 84 79 96 88 98];

%现在，运行 avgscore，并在运行时使用动态字段名称为 testscores 结构体提供学生姓名字段：

avgscore(testscores, 'Ann_Lane', 7, 22)
ans = 
   85.2500

avgscore(testscores, 'William_King', 7, 22)
ans = 
   87.7500
   
```

### 线性代数

#### MATLAB环境中的矩阵

**MATLAB 环境使用术语*矩阵*来表示包含以二维网格排列的实数或复数的变量。更广泛而言，*数组*为向量、矩阵或更高维度的数值网格。MATLAB 中的所有数组都是矩形，在这种意义上沿任何维度的分量向量的长度均相同。矩阵中定义的数学运算是线性代数的主题**

- 创建矩阵

  MATLAB 提供了许多函数，用于创建各种类型的矩阵

  ```matlab
  %例如，您可以使用基于帕斯卡三角形的项创建一个对称矩阵
  A = pascal(3)
  
  A =
         1     1     1
         1     2     3
         1     3     6
  %您也可以创建一个非对称幻方矩阵，它的行总和与列总和相等：
  B = magic(3)
  B =
         8     1     6
         3     5     7
         4     9     2
         
  %另一个示例是由随机整数构成的 3×2 矩形矩阵：在这种情况下，randi 的第一个输入描述整数可能值的范围，后面两个输入描述行和列的数量
  C = randi(10,3,2)
  
  C =
  
       9    10
      10     7
       2     1
  ```

- 列向量为 m×1 矩阵，行向量为 1×n 矩阵，标量为 1×1 矩阵。要手动定义矩阵，请使用方括号 `[ ]` 来表示数组的开始和结束。在括号内，使用分号 `;` 表示行的结尾。在标量（1×1 矩阵）的情况下，括号不是必需的

  ```matlab
  u = [3; 1; 4]
  
  v = [2 0 -1]
  
  s = 7
  u =
         3
         1
         4
  
  v =
         2     0    -1
  
  s =
         7
  ```

- 矩阵的加法和减法

  矩阵和数组的加减法是逐个元素执行的，或者说是*按元素*执行的

  ```matlab
  %例如，A 加 B 之后再减去 A 又可以得到 B
  X = A + B
  X =
         9     2     7
         4     7    10
         5    12     8
  Y = X - A
  Y =
         8     1     6
         3     5     7
         4     9     2
  %加法和减法要求两个矩阵具有兼容的维度。如果维度不兼容，将会导致错误：
  X = A + C
  Error using  + 
  Matrix dimensions must agree.
  ```

- 向量乘积和转置

  ```matlab
  %长度相同的行向量和列向量可以按任一顺序相乘。其结果是一个标量（称为内积）或一个矩阵（称为外积）
  
  u = [3; 1; 4];
  v = [2 0 -1];
  x = v*u
  x =
  
       2
       
  X = u*v
  X =
  
       6     0    -3
       2     0    -1
       8     0    -4
  ```

  对于实矩阵，*转置*运算对 aij 和 aji 进行交换。对于复矩阵，还要考虑是否用数组中复数项的复共轭来形成*复共轭转置*。MATLAB 使用撇号运算符 (`'`) 执行复共轭转置，使用点撇号运算符 (`.'`) 执行无共轭的转置。对于包含所有实数元素的矩阵，这两个运算符返回相同结果

  ```matlab
  B = magic(3)
  B =
  
       8     1     6
       3     5     7
       4     9     2
  X = B'
  X =
  
       8     3     4
       1     5     9
       6     7     2
  
  %对于向量，转置会将行向量变为列向量（反之亦然）：
  
  x = v'
  
  x =
         2
         0
        -1
  
  %如果 x 和 y 均为实数列向量，则乘积 x*y 不确定，但以下两个乘积
  
  x'*y
  和
  
  y'*x
  %产生相同的标量结果。此参数使用很频繁，它有三个不同的名称内积、标量积或点积。甚至还有一个专门的点积函数，称为 dot。
  ```

  对于复数向量或矩阵 `z`，参量 `z'` 不仅可转置该向量或矩阵，而且可将每个复数元素转换为其复共轭数。也就是说，每个复数元素的虚部的正负号将会发生更改

  ```matlab
  z = [1+2i 7-3i 3+4i; 6-2i 9i 4+7i]
  z =
  
     1.0000 + 2.0000i   7.0000 - 3.0000i   3.0000 + 4.0000i
     6.0000 - 2.0000i   0.0000 + 9.0000i   4.0000 + 7.0000i
     
  %z 的复共轭转置为：
  z'
  ans =
  
     1.0000 - 2.0000i   6.0000 + 2.0000i
     7.0000 + 3.0000i   0.0000 - 9.0000i
     3.0000 - 4.0000i   4.0000 - 7.0000i
     
  %非共轭复数转置（其中每个元素的复数部分保留其符号）表示为 z.'：
  
  z.'
  ans =
  
     1.0000 + 2.0000i   6.0000 - 2.0000i
     7.0000 - 3.0000i   0.0000 + 9.0000i
     3.0000 + 4.0000i   4.0000 + 7.0000i
  
  %对于复数向量，两个标量积 x'*y 和 y'*x 互为复共轭数，而复数向量与其自身的标量积 x'*x 为实数
  ```

- 矩阵乘法

  对于复数向量，两个标量积 `x'*y` 和 `y'*x` 互为复共轭数，而复数向量与其自身的标量积 `x'*x` 为实数

  ```matlab
  A = pascal(3);
  B = magic(3);
  m = 3; 
  n = 3;
  for i = 1:m
       for j = 1:n
          C(i,j) = A(i,:)*B(:,j);
  %矩阵可以在右侧乘以列向量，在左侧乘以行向量：
  
  u = [3; 1; 4];
  x = A*u
  x =
  
       8
      17
      30
  v = [2 0 -1];
  y = v*B
  y =
  
  12 -7 10
  
  矩形矩阵乘法必须满足维度兼容性条件：由于 A 是 3×3 矩阵，C 是 3×2 矩阵，因此可将二者相乘得到 3×2 结果（共同的内部维度会消去）：
  
  X = A*C
  X =
  
  
  
      24    17
      47    42
      79    77
  但是，乘法不能以相反的顺序执行：
  
  Y = C*A
  Error using  * 
  Incorrect dimensions for matrix multiplication. Check that the number of columns 
  in the first matrix matches the number of rows in the second matrix. To perform 
  elementwise multiplication, use '.*'.end
  end
  
  %MATLAB 使用星号表示矩阵乘法，如 C = A*B 中所示。矩阵乘法不适用交换律；即 A*B 通常不等于 B*A：
  
  X = A*B
  X =
        15    15    15
        26    38    26
        41    70    39
  Y = B*A
  Y =
        15    28    47
        15    34    60
        15    28    43
  
  %矩阵可以在右侧乘以列向量，在左侧乘以行向量：
  
  u = [3; 1; 4];
  x = A*u
  x =
  
       8
      17
      30
  v = [2 0 -1];
  y = v*B
  y =
  
  12 -7 10
  
  
  %矩形矩阵乘法必须满足维度兼容性条件：由于 A 是 3×3 矩阵，C 是 3×2 矩阵，因此可将二者相乘得到 3×2 结果（共同的内部维度会消去）：
  
  X = A*C
  X =
  
  
  
      24    17
      47    42
      79    77
  %但是，乘法不能以相反的顺序执行：
  
  Y = C*A
  Error using  * 
  Incorrect dimensions for matrix multiplication. Check that the number of columns 
  in the first matrix matches the number of rows in the second matrix. To perform 
  elementwise multiplication, use '.*'.
  
  %您可以将任何内容与标量相乘：
  s = 10;
  w = s*y
  w =
  
     120   -70   100
  ```

- 单位矩阵

  ```
  eye(m,n)
  
  %返回 m×n 矩形单位矩阵，eye(n) 返回 n×n 单位方阵。
  ```

- 矩阵求逆

  如果矩阵 `A` 为非奇异方阵（非零行列式），则方程 AX = I 和 XA = I 具有相同的解 X。此解称为 `A` 的*逆矩阵*，表示为 A-1。[`inv`](https://ww2.mathworks.cn/help/matlab/ref/inv.html) 函数和表达式 `A^-1` 均可对矩阵求逆

  ```matlab
  A = pascal(3)
  A =
         1     1     1
         1     2     3
         1     3     6
  X = inv(A)
  X =
  
      3.0000   -3.0000    1.0000
     -3.0000    5.0000   -2.0000
      1.0000   -2.0000    1.0000
  A*X
  ans =
  
      1.0000         0         0
      0.0000    1.0000   -0.0000
     -0.0000    0.0000    1.0000
     
     
  %通过 det 计算的行列式表示由矩阵描述的线性变换的缩放因子。当行列式正好为零时，矩阵为奇异矩阵，因此不存在逆矩阵
  d = det(A)
  d =
  
       1
  
  %有些矩阵接近奇异矩阵，虽然存在逆矩阵，但计算容易出现数值误差。cond 函数计算逆运算的条件数，它指示矩阵求逆结果的精度。条件数的范围是从 1（数值稳定的矩阵）到 Inf（奇异矩阵）。
  
  c = cond(A)
  c =
  
     61.9839
  %很少需要为某个矩阵构造显式逆矩阵。求解线性方程组 Ax = b 时，常常会误用 inv。从执行时间和数值精度方面而言，求解此方程的最佳方法是使用矩阵反斜杠运算符，即 x = A\b。有关详细信息
  ```

  
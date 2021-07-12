> 笔记来源：[尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）](https://www.bilibili.com/video/BV1PJ411n7xZ "尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）")
>
> 同步更新：https://gitee.com/vectorx/NOTE_JVM
>
> https://codechina.csdn.net/qq_35925558/NOTE_JVM
>
> https://github.com/uxiahnan/NOTE_JVM

[toc]

# 1. 概述

![](https://img-blog.csdnimg.cn/img_convert/630bca9b1bbeeeb4f772fea5f94d43fc.png)![](https://img-blog.csdnimg.cn/img_convert/74bb5f23ad01f05a39f8eb171d205390.png)![img](https://img-blog.csdnimg.cn/img_convert/bb3c57508713e377f3f4084409abaa30.png)
# 2. 加载与存储指令

![0ca8044c-f78d-4787-aeac-c986a35f9cdf](https://img-blog.csdnimg.cn/img_convert/3585fe67d5d83aff5db707d8eedccbae.png)
![16e3afaf-b7d8-4a23-8897-9fe02586aafd](https://gitee.com/vectorx/ImageCloud/raw/master/img/20210424190846.png
)![08e01fd0-a33e-47e4-8fd2-34c2935db71d](https://img-blog.csdnimg.cn/img_convert/8e911caaa5c0502b038af324c16edce8.png)
<hr/>

## 2.1. 局部变量压栈指令

> iload 从局部变量中装载int类型值
>
> lload 从局部变量中装载long类型值
>
> fload 从局部变量中装载float类型值
>
> dload 从局部变量中装载double类型值
>
> aload 从局部变量中装载引用类型值（refernce）
>
> iload_0 从局部变量0中装载int类型值
>
> iload_1 从局部变量1中装载int类型值
>
> iload_2 从局部变量2中装载int类型值
>
> iload_3 从局部变量3中装载int类型值
>
> lload_0 从局部变量0中装载long类型值
>
> lload_1 从局部变量1中装载long类型值
>
> lload_2 从局部变量2中装载long类型值
>
> lload_3 从局部变量3中装载long类型值
>
> fload_0 从局部变量0中装载float类型值
>
> fload_1 从局部变量1中装载float类型值
>
> fload_2 从局部变量2中装载float类型值
>
> fload_3 从局部变量3中装载float类型值
>
> dload_0 从局部变量0中装载double类型值
>
> dload_1 从局部变量1中装载double类型值
>
> dload_2 从局部变量2中装载double类型值
>
> dload_3 从局部变量3中装载double类型值
>
> aload_0 从局部变量0中装载引用类型值
>
> aload_1 从局部变量1中装载引用类型值
>
> aload_2 从局部变量2中装载引用类型值
>
> aload_3 从局部变量3中装载引用类型值
>
> iaload 从数组中装载int类型值
>
> laload 从数组中装载long类型值
>
> faload 从数组中装载float类型值
>
> daload 从数组中装载double类型值
>
> aaload 从数组中装载引用类型值
>
> baload 从数组中装载byte类型或boolean类型值
>
> caload 从数组中装载char类型值
>
> saload 从数组中装载short类型值

## 局部变量压栈常用指令集

| xload_n        | xload_0 | xload_1 | xload_2 | xload_3 |
| -------------- | ------- | ------- | ------- | ------- |
| <b>iload_n</b> | iload_0 | iload_1 | iload_2 | iload_3 |
| <b>lload_n</b> | lload_0 | lload_1 | lload_2 | lload_3 |
| <b>fload_n</b> | fload_0 | fload_1 | fload_2 | fload_3 |
| <b>dload_n</b> | dload_0 | dload_1 | dload_2 | dload_3 |
| <b>aload_n</b> | aload_0 | aload_1 | aload_2 | aload_3 |

## 局部变量压栈指令剖析

![1](https://img-blog.csdnimg.cn/img_convert/a34d465c4c8c83b3fcedc3ba31401732.png)

```java
public void load(int num, Object obj, long count, boolean flag, short[] arr) {
	System.out.println(num);
    System.out.println(obj);
    System.out.println(count);
    System.out.println(flag);
    System.out.println(arr);
}
```

![3](https://img-blog.csdnimg.cn/img_convert/deb49e69ed62ed9d71c7059748299b59.png)
<hr/>

## 2.2. 常量入栈指令

> aconst_null 将null对象引用压入栈
>
> iconst_m1 将int类型常量-1压入栈
>
> iconst_0 将int类型常量0压入栈
>
> iconst_1 将int类型常量1压入栈
>
> iconst_2 将int类型常量2压入栈
>
> iconst_3 将int类型常量3压入栈
>
> iconst_4 将int类型常量4压入栈
>
> iconst_5 将int类型常量5压入栈
>
> lconst_0 将long类型常量0压入栈
>
> lconst_1 将long类型常量1压入栈
>
> fconst_0 将float类型常量0压入栈
>
> fconst_1 将float类型常量1压入栈
>
> dconst_0 将double类型常量0压入栈
>
> dconst_1 将double类型常量1压入栈
>
> bipush 将一个8位带符号整数压入栈
>
> sipush 将16位带符号整数压入栈
>
> ldc 把常量池中的项压入栈
>
> ldc_w 把常量池中的项压入栈（使用宽索引）
>
> ldc2_w 把常量池中long类型或者double类型的项压入栈（使用宽索引）

## 常量入栈常用指令集

|   xconst_n   | 范围                                                    | xconst_null | xconst_m1 | xconst_0 | xconst_1 | xconst_2 | xconst_3 | xconst_4 | xconst_5 |
| :----------: | ------------------------------------------------------- | ----------- | :-------: | :------: | :------: | :------: | :------: | :------: | :------: |
| **iconst_n** | [-1, 5]                                                 |             | iconst_m1 | iconst_0 | iconst_1 | iconst_2 | iconst_3 | iconst_4 | iconst_5 |
| **lconst_n** | 0, 1                                                    |             |           | lconst_0 | lconst_1 |          |          |          |          |
| **fconst_n** | 0, 1, 2                                                 |             |           | fconst_0 | fconst_1 | fconst_2 |          |          |          |
| **dconst_n** | 0, 1                                                    |             |           | dconst_0 | dconst_1 |          |          |          |          |
| **aconst_n** | null, String literal, Class literal                     | aconst_null |           |          |          |          |          |          |          |
|  **bipush**  | 一个字节，2^8^，[-2^7^, 2^7^ - 1]，即[-128, 127]        |             |           |          |          |          |          |          |          |
|  **sipush**  | 两个字节，2^16^，[-2^15^, 2^15^ - 1]，即[-32768, 32767] |             |           |          |          |          |          |          |          |
|   **ldc**    | 四个字节，2^32^，[-2^31^, 2^31^ - 1]                    |             |           |          |          |          |          |          |          |
|  **ldc_w**   | 宽索引                                                  |             |           |          |          |          |          |          |          |
|  **ldc2_w**  | 宽索引，long或double                                    |             |           |          |          |          |          |          |          |

## 常量入栈指令剖析

![437a717e-98e2-4847-b52e-e6632d0745a4](https://img-blog.csdnimg.cn/img_convert/fafa61a6702b5fc88179c404ba736029.png)
![ffd7246e-2e46-41e0-9fd6-1e65ace5dbd1](https://img-blog.csdnimg.cn/img_convert/c0a6284e1deaf76669089e669916f440.png)

<table>
    <tbody>  
        <tr>
            <th>类型</th> 
            <th>常数指令</th> 
            <th>范围</th> 
       </tr>
       <tr>
            <td rowspan="4">int(boolean,byte,char,short)</td>
            <td>iconst</td>
            <td>[-1, 5]</td>
       </tr>	
       <tr>
            <td>bipush</td>
            <td>[-128, 127]</td>
       </tr>
       <tr>
            <td>sipush</td>
            <td>[-32768, 32767]</td>
       </tr> 
       <tr>
            <td>ldc</td>
            <td>any int value</td>
       </tr>
       <tr>
            <td rowspan="2">long</td>
            <td>lconst</td>
            <td>0, 1</td>
       </tr>	
       <tr>
            <td>ldc</td>
            <td>any long value</td>
       </tr>
       <tr>
            <td rowspan="2">float</td>
            <td>fconst</td>
            <td>0, 1, 2</td>
       </tr>	
       <tr>
            <td>ldc</td>
            <td>any float value</td>
       </tr>
       <tr>
            <td rowspan="2">double</td>
            <td>dconst</td>
            <td>0, 1</td>
       </tr>	
       <tr>
            <td>ldc</td>
            <td>any double value</td>
       </tr> 
       <tr>
            <td rowspan="2">reference</td>
            <td>aconst</td>
            <td>null</td>
       </tr>
       <tr>
            <td>ldc</td>
            <td>String literal, Class literal</td>
       </tr>
   <tbody> 
</table>


![566b9397-5afe-4a3f-9e17-9ebf504dfc80](https://img-blog.csdnimg.cn/img_convert/59982d71dc70f7d7b873f50130281c21.png)
![b59702d2-4c93-44df-87f1-01a5dfe53b61](https://img-blog.csdnimg.cn/img_convert/cd990ebc801bf53b4f7b1966d9974345.png)
<hr/>

## 2.3. 出栈装入局部变量表指令

> istore 将int类型值存入局部变量
>
> lstore 将long类型值存入局部变量
>
> fstore 将float类型值存入局部变量
>
> dstore 将double类型值存入局部变量
>
> astore 将将引用类型或returnAddress类型值存入局部变量
>
> istore_0 将int类型值存入局部变量0
>
> istore_1 将int类型值存入局部变量1
>
> istore_2 将int类型值存入局部变量2
>
> istore_3 将int类型值存入局部变量3
>
> lstore_0 将long类型值存入局部变量0
>
> lstore_1 将long类型值存入局部变量1
>
> lstore_2 将long类型值存入局部变量2
>
> lstore_3 将long类型值存入局部变量3
>
> fstore_0 将float类型值存入局部变量0
>
> fstore_1 将float类型值存入局部变量1
>
> fstore_2 将float类型值存入局部变量2
>
> fstore_3 将float类型值存入局部变量3
>
> dstore_0 将double类型值存入局部变量0
>
> dstore_1 将double类型值存入局部变量1
>
> dstore_2 将double类型值存入局部变量2
>
> dstore_3 将double类型值存入局部变量3
>
> astore_0 将引用类型或returnAddress类型值存入局部变量0
>
> astore_1 将引用类型或returnAddress类型值存入局部变量1
>
> astore_2 将引用类型或returnAddress类型值存入局部变量2
>
> astore_3 将引用类型或returnAddress类型值存入局部变量3
>
> iastore 将int类型值存入数组中
>
> lastore 将long类型值存入数组中
>
> fastore 将float类型值存入数组中
>
> dastore 将double类型值存入数组中
>
> aastore 将引用类型值存入数组中
>
> bastore 将byte类型或者boolean类型值存入数组中
>
> castore 将char类型值存入数组中
>
> sastore 将short类型值存入数组中
>
> wide指令
>
> wide 使用附加字节扩展局部变量索引

## 出栈装入局部变量表常用指令集

|   xstore_n   | xstore_0 | xstore_1 | xstore_2 | xstore_3 |
| :----------: | :------: | :------: | :------: | :------: |
| **istore_n** | istore_0 | istore_1 | istore_2 | istore_3 |
| **lstore_n** | lstore_0 | lstore_1 | lstore_2 | lstore_3 |
| **fstore_n** | fstore_0 | fstore_1 | fstore_2 | fstore_3 |
| **dstore_n** | dstore_0 | dstore_1 | dstore_2 | dstore_3 |
| **astore_n** | astore_0 | astore_1 | astore_2 | astore_3 |

## 出栈装入局部变量表指令剖析

![1](https://img-blog.csdnimg.cn/img_convert/52b46ba6b57aa1ab8581cb022da7e58e.png)
![2](https://img-blog.csdnimg.cn/img_convert/4adce45129332dd04b89f4aa8ffc6e28.png)
![3](https://img-blog.csdnimg.cn/img_convert/e4665e8fc25e2d63bff2e8423b60b1dc.png)
<hr/>

# 3. 算术指令

> ## 整数运算
>
> iadd 执行int类型的加法
>
> ladd 执行long类型的加法
>
> isub 执行int类型的减法
>
> lsub 执行long类型的减法
>
> imul 执行int类型的乘法
>
> lmul 执行long类型的乘法
>
> idiv 执行int类型的除法
>
> ldiv 执行long类型的除法
>
> irem 计算int类型除法的余数
>
> lrem 计算long类型除法的余数
>
> ineg 对一个int类型值进行取反操作
>
> lneg 对一个long类型值进行取反操作
>
> iinc 把一个常量值加到一个int类型的局部变量上
>
> ## 逻辑运算
>
> ### 移位操作
>
> ishl 执行int类型的向左移位操作
>
> lshl 执行long类型的向左移位操作
>
> ishr 执行int类型的向右移位操作
>
> lshr 执行long类型的向右移位操作
>
> iushr 执行int类型的向右逻辑移位操作
>
> lushr 执行long类型的向右逻辑移位操作
>
> ### 按位布尔运算
>
> iand 对int类型值进行“逻辑与”操作
>
> land 对long类型值进行“逻辑与”操作
>
> ior 对int类型值进行“逻辑或”操作
>
> lor 对long类型值进行“逻辑或”操作
>
> ixor 对int类型值进行“逻辑异或”操作
>
> lxor 对long类型值进行“逻辑异或”操作
>
> ### 浮点运算
>
> fadd 执行float类型的加法
>
> dadd 执行double类型的加法
>
> fsub 执行float类型的减法
>
> dsub 执行double类型的减法
>
> fmul 执行float类型的乘法
>
> dmul 执行double类型的乘法
>
> fdiv 执行float类型的除法
>
> ddiv 执行double类型的除法
>
> frem 计算float类型除法的余数
>
> drem 计算double类型除法的余数
>
> fneg 将一个float类型的数值取反
>
> dneg 将一个double类型的数值取反

## 算术指令集

<table>
    <tbody>  
        <tr>
            <th colspan="2">算数指令</th> 
            <th>int(boolean,byte,char,short)</th> 
            <th>long</th>
            <th>float</th> 
      		<th>double</th> 
       </tr>
       <tr>
            <td colspan="2">加法指令</td>
            <td>iadd</td>
            <td>ladd</td>
            <td>fadd</td>
            <td>dadd</td>
       </tr>	
       <tr>
            <td colspan="2">减法指令</td>
            <td>isub</td>
            <td>lsub</td>
            <td>fsub</td>
            <td>dsub</td>
       </tr> 
       <tr>
            <td colspan="2">乘法指令</td>
            <td>imul</td>
            <td>lmul</td>
            <td>fmul</td>
            <td>dmul</td>
       </tr> 
       <tr>
            <td colspan="2">除法指令</td>
            <td>idiv</td>
            <td>ldiv</td>
            <td>fdiv</td>
            <td>ddiv</td>
       </tr>
       <tr>
            <td colspan="2">求余指令</td>
            <td>irem</td>
            <td>lrem</td>
            <td>frem</td>
            <td>drem</td>
       </tr>
       <tr>
            <td colspan="2">取反指令</td>
            <td>ineg</td>
            <td>lneg</td>
            <td>fneg</td>
            <td>dneg</td>
       </tr>
       <tr>
            <td colspan="2">自增指令</td>
            <td>iinc</td>
            <td></td>
            <td></td>
            <td></td>
       </tr>
       <tr>
            <td rowspan="4">位运算指令</td>
            <td>按位或指令</td>
            <td>ior</td>
            <td>lor</td>
            <td></td>
            <td></td>
       </tr> 
       <tr>
            <td>按位或指令</td>
            <td>ior</td>
            <td>lor</td>
            <td></td>
            <td></td>
       </tr> 
       <tr>
            <td>按位与指令</td>
            <td>iand</td>
            <td>land</td>
            <td></td>
            <td></td>
       </tr>
       <tr>
            <td>按位异或指令</td>
            <td>ixor</td>
            <td>lxor</td>
            <td></td>
            <td></td>
       </tr> 
       <tr>
            <td colspan="2">比较指令</td>
            <td></td>
            <td>lcmp</td>
            <td>fcmpg / fcmpl</td>
            <td>dcmpg / dcmpl</td>
       </tr> 
   <tbody> 
</table>


![](https://img-blog.csdnimg.cn/img_convert/39ac5dc0cb406c2d75b50b10226322b0.png)

> 注意：NaN(Not a Number)表示不是一个数字

## 算术指令举例

### 举例1

```java
public static int bar(int i) {
	return ((i + 1) - 2) * 3 / 4;
}
```

![a54c2ac8-dd36-49f4-a49d-9afd725e8365](https://img-blog.csdnimg.cn/img_convert/256d8a8ec2309b6d396795e9a7e79959.png)

### 举例2

```java
public void add() {
	byte i = 15;
	int j = 8;
	int k = i + j;
}
```

![image-20210424210710750](https://img-blog.csdnimg.cn/img_convert/58c6064f2d2103610c6e2f9c9472f122.png)
![2](https://img-blog.csdnimg.cn/img_convert/d0257760ed00864d7e36421c2df971ca.png)
![3](https://img-blog.csdnimg.cn/img_convert/df724aebb307c6dda0780bbf5d4e1f92.png)

![img](https://img-blog.csdnimg.cn/img_convert/f2edaef3312398b63decea146718f2d6.gif)

### 举例3

```java
public static void main(String[] args) {
	int x = 500;
	int y = 100;
	int a = x / y;
	int b = 50;
	System.out.println(a + b);
}
```

![c43c0407-020f-4ec4-bd27-e4c109640b39](https://img-blog.csdnimg.cn/img_convert/ce924815ec9c6ddc5cd98f18538c250e.png)
![04282df1-4e52-4c3d-a47b-84023159b624](https://img-blog.csdnimg.cn/img_convert/918a9850ced5114086db35ce59e651af.png)
<hr/>

# 4. 类型转换指令

> ## 宽化类型转换
>
> i2l 把int类型的数据转化为long类型
>
> i2f 把int类型的数据转化为float类型
>
> i2d 把int类型的数据转化为double类型
>
> l2f 把long类型的数据转化为float类型
>
> l2d 把long类型的数据转化为double类型
>
> f2d 把float类型的数据转化为double类型
>
> ## 窄化类型转换
>
> i2b 把int类型的数据转化为byte类型
>
> i2c 把int类型的数据转化为char类型
>
> i2s 把int类型的数据转化为short类型
>
> l2i 把long类型的数据转化为int类型
>
> f2i 把float类型的数据转化为int类型
>
> f2l 把float类型的数据转化为long类型
>
> d2i 把double类型的数据转化为int类型
>
> d2l 把double类型的数据转化为long类型
>
> d2f 把double类型的数据转化为float类型

|            | **byte** | **char** | **short** | **int** | **long** | **float** | **double** |
| :--------: | :------: | :------: | :-------: | :-----: | :------: | :-------: | :--------: |
|  **int**   |   i2b    |   i2c    |    i2s    |    ○    |   i2l    |    i2f    |    i2d     |
|  **long**  | l2i i2b  | l2i i2c  |  l2i i2s  |   l2i   |    ○     |    l2f    |    l2d     |
| **float**  | f2i i2b  | f2i i2c  |  f2i i2s  |   f2i   |   f2l    |     ○     |    f2d     |
| **double** | d2i i2b  | d2i i2c  |  d2i i2s  |   d2i   |   d2l    |    d2f    |     ○      |

类型转换指令可以将两种不同的数值类型进行相互转换。这些转换操作一般用于实现用户代码中的显式类型转換操作，或者用来处理字节码指令集中数据类型相关指令无法与数据类型一一对应的问题。

## 4.1. 宽化类型转换剖析

> 宽化类型转换( Widening Numeric Conversions)
>
> 1. 转换规则
>
> Java虚拟机直接支持以下数值的宽化类型转换（ widening numeric conversion,小范围类型向大范围类型的安全转换）。也就是说，并不需要指令执行，包括
>
> > 从int类型到long、float或者 double类型。对应的指令为：i21、i2f、i2d
> >
> > 从long类型到float、 double类型。对应的指令为：i2f、i2d
> >
> > 从float类型到double类型。对应的指令为：f2d
>
> 简化为：int-->long-->float-> double
>
>
> 2. 精度损失问题
>
> > 2.1. 宽化类型转换是不会因为超过目标类型最大值而丢失信息的，例如，从int转换到long,或者从int转换到double,都不会丢失任何信息，转换前后的值是精确相等的。
> >
> > 2.2. 从int、long类型数值转换到float,或者long类型数值转换到double时，将可能发生精度丢失一一可能丢失掉几个最低有效位上的值，转换后的浮点数值是根据IEEE754最接近含入模式所得到的正确整数值。
>
> 尽管宽化类型转换实际上是可能发生精度丢失的，但是这种转换永远不会导致Java虚拟机抛出运行时异常
>
>
> 3. 补充说明
>
> 从byte、char和 short类型到int类型的宽化类型转换实际上是不存在的。对于byte类型转为int,拟机并没有做实质性的转化处理，只是简单地通过操作数栈交換了两个数据。而将byte转为long时，使用的是i2l,可以看到在内部，byte在这里已经等同于int类型处理，类似的还有 short类型，这种处理方式有两个特点：
>
> 一方面可以减少实际的数据类型，如果为 short和byte都准备一套指令，那么指令的数量就会大増，而虚拟机目前的设计上，只愿意使用一个字节表示指令，因此指令总数不能超过256个，为了节省指令资源，将 short和byte当做int处理也在情理之中。
>
> 另一方面，由于局部变量表中的槽位固定为32位，无论是byte或者 short存入局部变量表，都会占用32位空间。从这个角度说，也没有必要特意区分这几种数据类型。

## 4.2. 窄化类型转换剖析

> 窄化类型转换( Narrowing Numeric Conversion)
>
> 1. 转换规则
>
> Java虚拟机也直接支持以下窄化类型转换：
>
> > 从主int类型至byte、 short或者char类型。对应的指令有：i2b、i2c、i2s
> >
> > 从long类型到int类型。对应的指令有：l2i
> >
> > 从float类型到int或者long类型。对应的指令有：f2i、f2l
> >
> > 从double类型到int、long或者float类型。对应的指令有：d2i、d2l、d2f
>
>
> 2. 精度损失问题
>
> 窄化类型转换可能会导致转换结果具备不同的正负号、不同的数量级，因此，转换过程很可能会导致数值丢失精度。
>
> 尽管数据类型窄化转换可能会发生上限溢出、下限溢出和精度丢失等情况，但是Java虚拟机规范中明确规定数值类型的窄化转换指令永远不可能导致虚拟机抛出运行时异常
>
> 3. 补充说明
>
> > 3.1. 当将一个浮点值窄化转换为整数类型T(T限于int或long类型之一)的时候，将遵循以下转换规则：
> >
> > > 如果浮点值是NaN,那转换结果就是int或long类型的0.
> > >
> > > 如果浮点值不是无穷大的话，浮点值使用IEEE754的向零含入模式取整，获得整数值Vv如果v在目标类型T(int或long)的表示范围之内，那转换结果就是v。否则，将根据v的符号，转换为T所能表示的最大或者最小正数
> >
> > 3.2. 当将一个double类型窄化转换为float类型时，将遵循以下转换规则
> >
> > > 通过向最接近数舍入模式舍入一个可以使用float类型表示的数字。最后结果根据下面这3条规则判断
> > >
> > > 如果转换结果的绝对值太小而无法使用float来表示，将返回float类型的正负零
> > >
> > > 如果转换结果的绝对值太大而无法使用float来表示，将返回float类型的正负无穷大。
> > >
> > > 对于double类型的NaN值将按规定转換为float类型的NaN值。

<hr/>

# 5. 对象的创建与访问指令

> ## 对象操作指令
>
> new 创建一个新对象
>
> getfield 从对象中获取字段
>
> putfield 设置对象中字段的值
>
> getstatic 从类中获取静态字段
>
> putstatic 设置类中静态字段的值
>
> checkcast 确定对象为所给定的类型。后跟目标类，判断栈顶元素是否为目标类 / 接口的实例。如果不是便抛出异常
>
> instanceof 判断对象是否为给定的类型。后跟目标类，判断栈顶元素是否为目标类 / 接口的实例。是则压入 1，否则压入 0
>
>
> ## 数组操作指令
>
> newarray 分配数据成员类型为基本上数据类型的新数组
>
> anewarray 分配数据成员类型为引用类型的新数组
>
> arraylength 获取数组长度
>
> multianewarray 分配新的多维数组

Java是面向对象的程序设计语言，虚拟机平台从字节码层面就对面向对象做了深层次的支持。有一系列指令专门用于对象操作，可进一步细分为创建指令、字段访问指令、数组操作指令、类型检查指令。

## 5.1. 创建指令

| 创建指令       | 含义             |
| :------------- | :--------------- |
| new            | 创建类实例       |
| newarray       | 创建基本类型数组 |
| anewarray      | 创建引用类型数组 |
| multilanewarra | 创建多维数组     |

![img](https://img-blog.csdnimg.cn/img_convert/44ec8eda0028b78f02951aa5edc14751.png)

## 5.2. 字段访问指令

| 字段访问指令         | 含义                                                   |
| :------------------- | :----------------------------------------------------- |
| getstatic、putstatic | 访问类字段（static字段，或者称为类变量）的指令         |
| getfield、 putfield  | 访问类实例字段（非static字段，或者称为实例变量）的指令 |

![img](https://img-blog.csdnimg.cn/img_convert/c95da4b9bcb174f367617ca977451b14.png)
![img](https://img-blog.csdnimg.cn/img_convert/6f7033f9caaf3216c6ca6795de12f6a4.png)

## 5.3. 数组操作指令

| 数组指令    | byte(boolean) | char    | short   | long    | long    | float   | double  | reference |
| ----------- | ------------- | ------- | ------- | ------- | ------- | ------- | ------- | --------- |
| **xaload**  | baload        | caload  | saload  | iaload  | laload  | faload  | daload  | aaload    |
| **xastore** | bastore       | castore | sastore | iastore | lastore | fastore | dastore | aastore   |

![img](https://img-blog.csdnimg.cn/img_convert/a25e46492ce58084d3bb1ee9a4255ac4.png)
![img](https://img-blog.csdnimg.cn/img_convert/4d55c1e67881b686d4dcb75118f257c5.png)

## 5.4. 类型检查指令

| 类型检查指令 | 含义                             |
| ------------ | -------------------------------- |
| instanceof   | 检查类型强制转换是否可以进行     |
| checkcast    | 判断给定对象是否是某一个类的实例 |

![img](https://img-blog.csdnimg.cn/img_convert/df1e056d6e977d38b6c5974a428a040a.png)
<hr/>

# 6. 方法调用与返回指令

> ## 方法调用指令
>
> invokcvirtual 运行时按照对象的类来调用实例方法
>
> invokespecial 根据编译时类型来调用实例方法
>
> invokestatic 调用类（静态）方法
>
> invokcinterface 调用接口方法
>
> ## 方法返回指令
>
> ireturn 从方法中返回int类型的数据
>
> lreturn 从方法中返回long类型的数据
>
> freturn 从方法中返回float类型的数据
>
> dreturn 从方法中返回double类型的数据
>
> areturn 从方法中返回引用类型的数据
>
> return 从方法中返回，返回值为void

## 6.1. 方法调用指令

| 方法调用指令    | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| invokevirtual   | 调用对象的实例方法                                           |
| invokeinterface | 调用接口方法                                                 |
| invokespecial   | 调用一些需要特殊处理的实例方法，包括实例初始化方法（构造器）、私有方法和父类方法 |
| invokestatic    | 调用命名类中的类方法（static方法）                           |
| invokedynamic   | 调用动态绑定的方法                                           |

![img](https://img-blog.csdnimg.cn/img_convert/6b6c265e506611b1fa05134d7ede3f30.png)

## 6.2. 方法返回指令

| 方法返回指令 | void   | int     | long    | float   | double  | reference |
| ------------ | ------ | ------- | ------- | ------- | ------- | --------- |
| **xreturn**  | return | ireturn | lreturn | freutrn | dreturn | areturn   |

![image-20210425222017858](https://img-blog.csdnimg.cn/img_convert/8940d3d81dff02c08bf83cf0c66f3fea.png)
![img](https://img-blog.csdnimg.cn/img_convert/386375c9f516af716a5d8dec10177444.png)

```java
public int methodReturn() {
    int i = 500;
    int j = 200;
    int k = 50;
    
    return (i + j) / k;
}
```

![image-20210425222245665](https://img-blog.csdnimg.cn/img_convert/3fbd1f2ca9f4300eee5e0c4a0227a441.png)
<hr/>

# 7. 操作数栈管理指令

> ## 通用(无类型）栈操作
>
> nop 不做任何操作
>
> pop 弹出栈顶端一个字长的内容
>
> pop2 弹出栈顶端两个字长的内容
>
> dup 复制栈顶部一个字长内容
>
> dup_x1 复制栈顶部一个字长的内容，然后将复制内容及原来弹出的两个字长的内容压入栈
>
> dup_x2 复制栈顶部一个字长的内容，然后将复制内容及原来弹出的三个字长的内容压入栈
>
> dup2 复制栈顶部两个字长内容
>
> dup2_x1 复制栈顶部两个字长的内容，然后将复制内容及原来弹出的三个字长的内容压入栈
>
> dup2_x2 复制栈顶部两个字长的内容，然后将复制内容及原来弹出的四个字长的内容压入栈
>
> swap 交换栈顶部两个字长内容

![img](https://img-blog.csdnimg.cn/img_convert/316306469dcd1360c16578931cd064fa.png)
![img](https://img-blog.csdnimg.cn/img_convert/0d8d5d90fc84398f114eae5d1119cf6b.png)
<hr/>

# 8. 控制转移指令

> ## 比较指令
>
> lcmp 比较long类型值
>
> fcmpl 比较float类型值（当遇到NaN时，返回-1）
>
> fcmpg 比较float类型值（当遇到NaN时，返回1）
>
> dcmpl 比较double类型值（当遇到NaN时，返回-1）
>
> dcmpg 比较double类型值（当遇到NaN时，返回1）
>
> ## 条件分支指令
>
> ifeq 如果等于0，则跳转
>
> ifne 如果不等于0，则跳转
>
> iflt 如果小于0，则跳转
>
> ifge 如果大于等于0，则跳转
>
> ifgt 如果大于0，则跳转
>
> ifle 如果小于等于0，则跳转
>
> ## 比较条件分支指令
>
> if_icmpeq 如果两个int值相等，则跳转
>
> if_icmpne 如果两个int类型值不相等，则跳转
>
> if_icmplt 如果一个int类型值小于另外一个int类型值，则跳转
>
> if_icmpge 如果一个int类型值大于或者等于另外一个int类型值，则跳转
>
> if_icmpgt 如果一个int类型值大于另外一个int类型值，则跳转
>
> if_icmple 如果一个int类型值小于或者等于另外一个int类型值，则跳转
>
> ifnull 如果等于null，则跳转
>
> ifnonnull 如果不等于null，则跳转
>
> if_acmpeq 如果两个对象引用相等，则跳转
>
> if_acmpne 如果两个对象引用不相等，则跳转
>
> ## 多条件分支跳转指令
>
> tableswitch 通过索引访问跳转表，并跳转
>
> lookupswitch 通过键值匹配访问跳转表，并执行跳转操作
>
> ## 无条件跳转指令
>
> goto 无条件跳转
>
> goto_w 无条件跳转（宽索引）

## 8.1. 比较指令

> 比较指令的作用是比较占栈顶两个元素的大小，并将比较结果入栽。
>
> 比较指令有： dcmpg,dcmpl、 fcmpg、fcmpl、lcmp
>
> 与前面讲解的指令类似，首字符d表示double类型，f表示float,l表示long.
>
> 对于double和float类型的数字，由于NaN的存在，各有两个版本的比较指令。以float为例，有fcmpg和fcmpl两个指令，它们的区别在于在数字比较时，若遇到NaN值，处理结果不同。
>
> 指令dcmpl和 dcmpg也是类似的，根据其命名可以推测其含义，在此不再赘述。
>
> 举例
>
> 指令 fcmp和fcmpl都从中弹出两个操作数，并将它们做比较，设栈顶的元素为v2,顶顺位第2位的元素为v1,若v1=v2,则压入0:若v1>v2则压入1:若v1<v2则压入-1.
>
> 两个指令的不同之处在于，如果遇到NaN值， fcmpg会压入1,而fcmpl会压入-1

## 8.2. 条件跳转指令

| <    | <=   | ==   | !=   | >=   | >    | null   | not null  |
| ---- | ---- | ---- | ---- | ---- | ---- | ------ | --------- |
| iflt | ifle | ifeq | ifng | ifge | ifgt | ifnull | ifnonnull |

![img](https://img-blog.csdnimg.cn/img_convert/e59234f41d6946ead781b686455783d9.png)
![img](https://img-blog.csdnimg.cn/img_convert/85374206e3cdbb00f45971b1429c6d34.png)

## 8.3. 比较条件跳转指令

| <         | <=        | ==                   | !=                   | >=        | >         |
| --------- | --------- | -------------------- | -------------------- | --------- | --------- |
| if_icmplt | if_icmple | if_icmpeq、if_acmpeq | if_icmpne、if_acmpne | if_icmpge | if_icmpgt |

![img](https://img-blog.csdnimg.cn/img_convert/7ffa349a864e62f3e0298b63acb904ae.png)

## 8.4. 多条件分支跳转

![img](https://img-blog.csdnimg.cn/img_convert/f6f973613d257f1d172e0fcee504cd97.png)
![img](https://img-blog.csdnimg.cn/img_convert/57c7b57b096f8a8cfcdcdd1b5d4e179b.png)
![img](https://img-blog.csdnimg.cn/img_convert/a709c4fcd4bf627ad47d53ac062db47b.png)

## 8.5. 无条件跳转

![img](https://img-blog.csdnimg.cn/img_convert/9e86f7429623e68e353ce9d1962a1267.png)
<hr/>

# 9. 异常处理指令

> ## 异常处理指令
>
> athrow 抛出异常或错误。将栈顶异常抛出
>
> jsr 跳转到子例程
>
> jsr_w 跳转到子例程（宽索引）
>
> rct 从子例程返回

![img](https://img-blog.csdnimg.cn/img_convert/196b81557c33474cf2cfa2032aa65340.png)
![img](https://img-blog.csdnimg.cn/img_convert/9e9883544cea41b6148e535da4417d78.png)
![img](https://img-blog.csdnimg.cn/img_convert/daa25784fc259c12af58bb094d6ffc52.png)
![img](https://img-blog.csdnimg.cn/img_convert/cba1429ffc09988d78008f5309a6374a.png)
<hr/>

# 10. 同步控制指令

> ### 线程同步
>
> montiorenter 进入并获取对象监视器。即：为栈顶对象加锁
>
> monitorexit 释放并退出对象监视器。即：为栈顶对象解锁

Java虚拟机支持两种同步结构：方法级的同步和方法内部一段指令序列的同步，这两种同步都是使用monitor来支持的

## 10.1. 方法级的同步

![img](https://img-blog.csdnimg.cn/img_convert/20b7bf14a51e1be4f90dac5e305d2ea3.png)

```java
private int i = 0;
public synchronized void add() {
	i++;
}
```

![img](https://img-blog.csdnimg.cn/img_convert/098e9fef1897cf213d14f145db04c9f1.png)
![img](https://img-blog.csdnimg.cn/img_convert/84ff4ef05baf1b5774f43b203d6a6e23.png)

## 10.2. 方法内指令指令序列的同步

![img](https://img-blog.csdnimg.cn/img_convert/697b683f5fba682a6ed9772950f719a7.png)
![img](https://img-blog.csdnimg.cn/img_convert/71ab99d8f145e61b31daa03a233f2596.png)
![img](https://img-blog.csdnimg.cn/img_convert/ac36b8a792107c77956ef642afba2154.png)
![img](https://img-blog.csdnimg.cn/img_convert/89199776b49d72ff70615f9b0ca8cbe2.png)

<hr/>
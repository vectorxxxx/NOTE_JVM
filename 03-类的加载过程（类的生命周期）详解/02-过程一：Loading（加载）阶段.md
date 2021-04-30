# 过程一：Loading（加载）阶段

## 1. 加载完成的操作

**加载的理解**

<span style="color:#FF0000;">所谓加载，简而言之就是将Java类的字节码文件加载到机器内存中，并在内存中构建出Java类的原型——类模板对象。</span>所谓类模板对象，其实就是Java类在]VM内存中的一个快照，JVM将从字节码文件中解析出的常量池、类字段、类方法等信息存储到类模板中，这样]VM在运行期便能通过类模板而获取Java类中的任意信息，能够对Java类的成员变量进行遍历，也能进行Java方法的调用。

反射的机制即基于这一基础。如果JVM没有将Java类的声明信息存储起来，则JVM在运行期也无法反射。

**加载完成的操作**

<span style="color:#FF0000;">加载阶段，简言之，查找并加载类的二进制数据，生成Class的实例。</span>

在加载类时，Java虚拟机必须完成以下3件事情：

- 通过类的全名，获取类的二进制数据流。

- 解析类的二进制数据流为方法区内的数据结构（Java类模型）

- 创建java.lang.Class类的实例，表示该类型。作为方法区这个类的各种数据的访问入口

## 2. 二进制流的获取方式

对于类的二进制数据流，虚拟机可以通过多种途径产生或获得。<mark>（只要所读取的字节码符合JVM规范即可）</mark>

- 虚拟机可能通过文件系统读入一个class后缀的文件<span style="color:#FF0000;">（最常见）</span>
- 读入jar、zip等归档数据包，提取类文件。
- 事先存放在数据库中的类的二进制数据
- 使用类似于HTTP之类的协议通过网络进行加载
- 在运行时生成一段class的二进制信息等
- 在获取到类的二进制信息后，Java虚拟机就会处理这些数据，并最终转为一个java.lang.Class的实例。

如果输入数据不是ClassFile的结构，则会抛出ClassFormatError。

## 3. 类模型与Class实例的位置

**类模型的位置**

加载的类在JVM中创建相应的类结构，类结构会存储在方法区（JDKl.8之前：永久代；J0Kl.8及之后：元空间）。

**Class实例的位置**

类将.class文件加载至元空间后，会在堆中创建一个Java.lang.Class对象，用来封装类位于方法区内的数据结构，该Class对象是在加载类的过程中创建的，每个类都对应有一个Class类型的对象。

![image-20210430221037898](https://gitee.com/vectorx/ImageCloud/raw/master/img/20210430221040.png)

```java
Class clazz = Class.forName("java.lang.String");
//获取当前运行时类声明的所有方法
Method[] ms = clazz.getDecla#FF0000Methods();
for (Method m : ms) {
    //获取方法的修饰符
    String mod = Modifier.toString(m.getModifiers());
    System.out.print(mod + "");
    //获取方法的返回值类型
    String returnType = (m.getReturnType()).getSimpleName();
    System.out.print(returnType + "");
    //获取方法名
    System.out.print(m.getName() + "(");
    //获取方法的参数列表
    Class<?>[] ps = m.getParameterTypes();
    if (ps.length == 0) {
        System.out.print(')');
    }
    for (int i = 0; i < ps.length; i++) {
        char end = (i == ps.length - 1) ? ')' : ',';
        //获取参教的类型
        System.out.print(ps[i].getSimpleName() + end);
    }
}
```

## 4. 数组类的加载

创建数组类的情况稍微有些特殊，因为<mark>数组类本身并不是由类加载器负责创建</mark>，而是由JVM在运行时根据需要而直接创建的，但数组的元素类型仍然需要依靠类加载器去创建。创建数组类（下述简称A）的过程：

- 如果数组的元素类型是引用类型，那么就遵循定义的加载过程递归加载和创建数组A的元素类型；
- JVM使用指定的元素类型和数组维度来创建新的数组类。

如果数组的元素类型是引用类型，数组类的可访问性就由元素类型的可访问性决定。否则数组类的可访问性将被缺省定义为public。
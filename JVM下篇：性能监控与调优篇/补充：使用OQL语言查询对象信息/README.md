> 笔记来源：[尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）](https://www.bilibili.com/video/BV1PJ411n7xZ "尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）")
>
> 同步更新：https://gitee.com/vectorx/NOTE_JVM
>
> https://codechina.csdn.net/qq_35925558/NOTE_JVM
>
> https://github.com/uxiahnan/NOTE_JVM

[toc]

# 补充：使用OQL语言查询对象信息

MAT支持一种类似于SQL的查询语言OQL（Object Query Language）。OQL使用类SQL语法，可以在堆中进行对象的查找和筛选。

## 1. SELECT子句

在MAT中，Select子句的格式与SQL基本一致，用于指定要显示的列。Select子句中可以使用“＊”，查看结果对象的引用实例（相当于outgoing references）。

```sql
SELECT * FROM java.util.Vector v
```

使用“OBJECTS”关键字，可以将返回结果集中的项以对象的形式显示。

```sql
SELECT objects v.elementData FROM java.util.Vector v

SELECT OBJECTS s.value FROM java.lang.String s
```

在Select子句中，使用“AS RETAINED SET”关键字可以得到所得对象的保留集。

```sql
SELECT AS RETAINED SET *FROM com.atguigu.mat.Student
```

“DISTINCT”关键字用于在结果集中去除重复对象。

```sql
SELECT DISTINCT OBJECTS classof(s) FROM java.lang.String s
```

## 2. FROM子句

From子句用于指定查询范围，它可以指定类名、正则表达式或者对象地址。

```sql
SELECT * FROM java.lang.String s
```

使用正则表达式，限定搜索范围，输出所有com.atguigu包下所有类的实例


```sql
SELECT * FROM "com\.atguigu\..*"
```

使用类的地址进行搜索。使用类的地址的好处是可以区分被不同ClassLoader加载的同一种类型。

```sql
select * from 0x37a0b4d
```

## 3. WHERE子句

Where子句用于指定OQL的查询条件。OQL查询将只返回满足Where子句指定条件的对象。Where子句的格式与传统SQL极为相似。

返回长度大于10的char数组。

```sql
SELECT *FROM Ichar[] s WHERE s.@length>10
```

返回包含“java”子字符串的所有字符串，使用“LIKE”操作符，“LIKE”操作符的操作参数为正则表达式。

```sql
SELECT * FROM java.lang.String s WHERE toString(s) LIKE ".*java.*"
```

返回所有value域不为null的字符串，使用“＝”操作符。

```sql
SELECT * FROM java.lang.String s where s.value!=null
```

返回数组长度大于15，并且深堆大于1000字节的所有Vector对象。

```sql
SELECT * FROM java.util.Vector v WHERE v.elementData.@length>15 AND v.@retainedHeapSize>1000
```

## 4. 内置对象与方法

OQL中可以访问堆内对象的属性，也可以访问堆内代理对象的属性。访问堆内对象的属性时，格式如下，其中alias为对象名称：

[ &lt;alias&gt;. ] &lt;field&gt; . &lt;field&gt;. &lt;field&gt;

访问java.io.File对象的path属性，并进一步访问path的value属性：

```sql
SELECT toString(f.path.value) FROM java.io.File f
```

显示String对象的内容、objectid和objectAddress。

```sql
SELECT s.toString(),s.@objectId, s.@objectAddress FROM java.lang.String s
```

显示java.util.Vector内部数组的长度。

```sql
SELECT v.elementData.@length FROM java.util.Vector v
```

显示所有的java.util.Vector对象及其子类型

```sql
select * from INSTANCEOF java.util.Vector
```


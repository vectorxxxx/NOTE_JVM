## Class字节码整体概览

```java
01-魔数：Class文件的标志
    magic 魔数     
02-Class文件版本号
    minor_version 副版本号(小版本)
    major_version 主版本号(大版本)
03-常量池：存放所有常量
    constant_pool_count 常量池计数器
    constant_pool       常量池表
04-访问标识
   access_flags 访问标识
05-类索引、父类索引、接口索引集合
    this_class       类索引
    super_class      父类索引
    interfaces_count 接口技术器
    interfaces       接口索引集合
06-字段表集合
    fields_count 字段计数器
    fields       字段表
07-方法表集合
    methods_count 方法计数器
    methods       方法表
08-属性表集合
    attributes_count 属性计数器
    attributes       属性表
```

## Class字节码文件结构
<table>
    <tbody>  
        <tr>
            <th></th> 
            <th>类型</th> 
            <th>名称</th> 
            <th>说明</th> 
            <th>长度</th> 
            <th>数量</th> 
       </tr>
       <tr>
            <td>魔数</td>
            <td>u4</td>
            <td>magic</td>
            <td>魔数,识别Class文件格式</td>
            <td>4个字节</td>     
            <td>1</td>
       </tr>
       <tr>
            <td rowspan="2">版本号</td>
            <td>u2</td>
            <td>minor_version</td>
            <td>副版本号(小版本)</td>
            <td>2个字节</td>     
            <td>1</td>
       </tr>
       <tr>
            <td>u2</td>
            <td>major_version</td>
            <td>主版本号(大版本)</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>	
        <tr>
            <td rowspan="2">常量池集合</td>
            <td>u2</td>
            <td>constant_pool_count</td>
            <td>常量池计数器</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>
        <tr>
            <td>cp_info</td>
            <td>constant_pool</td>
            <td>常量池表</td>
            <td>n个字节</td>     
            <td>constant_pool_count - 1</td>
        </tr>
        <tr>
            <td>访问标识</td>
            <td>u2</td>
            <td>access_flags</td>
            <td>访问标识</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>	
        <tr>
            <td rowspan="4">索引集合</td>
            <td>u2</td>
            <td>this_class</td>
            <td>类索引</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>
        <tr>
            <td>u2</td>
            <td>super_class</td>
            <td>父类索引</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>
        <tr>
            <td>u2</td>
            <td>interfaces_count</td>
            <td>接口计数器</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>
        <tr>
            <td>u2</td>
            <td>interfaces</td>
            <td>接口索引集合</td>
            <td>2个字节</td>     
            <td>interfaces_count</td>
        </tr>    
        <tr>
            <td rowspan="2">字段表集合</td>
            <td>u2</td>
            <td>fields_count</td>
            <td>字段计数器</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>
        <tr>
            <td>field_info</td>
            <td>fields</td>
            <td>字段表</td>
            <td>n个字节</td>     
            <td>fields_count</td>
        </tr>	
        <tr>
            <td rowspan="2">方法表集合</td>
            <td>u2</td>
            <td>methods_count</td>
            <td>方法计数器</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>
        <tr>
            <td>method_info</td>
            <td>methods</td>
            <td>方法表</td>
            <td>n个字节</td>     
            <td>methods_count</td>
        </tr>
        <tr>
            <td rowspan="2">属性表集合</td>
            <td>u2</td>
            <td>attributes_count</td>
            <td>属性计数器</td>
            <td>2个字节</td>     
            <td>1</td>
        </tr>
        <tr>
            <td>attribute_info</td>
            <td>attributes</td>
            <td>属性表</td>
            <td>n个字节</td>     
            <td>attributes_count</td>
        </tr>	
   <tbody> 
</table>

## Class文件数据类型
| 数据类型 | 定义                                                         | 说明                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 无符号数 | 无符号数可以用来描述数字、索引引用、数量值或按照utf-8编码构成的字符串值。 | 其中无符号数属于基本的数据类型。 以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节 |
| 表       | 表是由多个无符号数或其他表构成的复合数据结构。               | 所有的表都以“_info”结尾。 由于表没有固定长度，所以通常会在其前面加上个数说明。 |


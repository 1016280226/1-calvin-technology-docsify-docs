# 笔记二 Groovy 基本数据类型

## 1. <font color="#ff6702"><b>String</b></font> 类型

- **`String`** ：字符串

- 字符串标识：**`‘ ’`**（单引号），**`“ ”`**（双引号），**`""" """`** （三引号）
- 常用方法，如下列表：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>方法</th>
     <th>含义</th>
     <th>返回</th>
    </tr>
    <tr>
      <td><b>contains</b></td>
      <td>包含特定内容。</td></td>
      <td>true 或者 false</td>
    </tr>
    <tr>
      <td><b>size/length</b></td>
      <td>字符串数量大小长度。</td>
      <td></td>
    </tr>
    <tr>
      <td><b>toString</b></td>
      <td>转换成string类型。</td>
      <td></td>
    </tr>
    <tr>
      <td><b>indexOf</b></td>
      <td>元素的索引。 </td>
      <td></td>
    </tr>
    <tr>
      <td><b>endWith</b></td>
      <td>是否指定字符串结尾。 </td>
      <td></td>
    </tr>
    <tr>
      <td><b>minus（-）/plus（+）</b></td>
      <td>去掉、增加字符串。 </td>
      <td></td>
    </tr>
    <tr>
      <td><b>reverse</b></td>
      <td>反向排序。</td>
      <td></td>
    </tr>
    <tr>
      <td><b>substring</b></td>
      <td>字符串截取。</td>
      <td></td>
    </tr>   
    <tr>
      <td><b>split</b></td>
      <td>字符串分割（默认空格分隔）</td>
      <td>返回List列表</td>
    </tr>   
</table>



> 注意：**`‘’`** 单引号和 **`“ ”`** 双引号的区别：
>
> - 如果使用 **`‘’`** 单引号在单引号中有变量的话，变量不生效。
> - 如果使用 **`“ ”`** 双引号中有变量的话，变量是生效的。

示例：

```groovy
name = "张三"
name1 = "张三,李四,王五"
name2 = "zhangsan"
println("单引号输出结果：" + '${name}')
println("双引号输出结果：" + "${name}")
println("contains方法使用结果：" + "${name}".contains("张"))
println("size方法使用结果：" + "${name}".size())
println("length方法使用结果：" + "${name}".length())
println("toString方法使用结果：" + "${name}".toString())
println("indexOf方法使用结果：" + "${name}".indexOf("三"))
println("endsWith方法使用结果：" + "${name}".endsWith("三"))
println("minus方法使用结果：" + "${name}".minus("三"))
println("plus方法使用结果：" + "${name}".plus("大烧饼"))
println("reverse方法使用结果：" + "${name}".reverse())
println("substring方法使用结果：" + "${name}".substring(0,1))
println("toUpperCase方法使用结果：" + "${name2}".toUpperCase())
List<String> splitArrays =  "${name1}".split(",");
for (i in splitArrays) {
    println("split方法使用结果：" + i)
}
```

<font color="green"><b>输出结果:</b></font>

<img src="./statics/images/groovy/groovy_string_type_01.png" style="zoom:100%;" />




## 2. <font color="#ff6702"><b>List</b></font> 类型

- **`List`** 列表符号：**`[]`**
- 常用方法，如下列表：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>方法</th>
     <th>含义</th>
    </tr>
    <tr>
      <td><b>+ - += -=</b></td>
      <td>元素增加减少。</td></td>
    </tr>
    <tr>
      <td><b>isEmpty</b></td>
      <td>判断是否为空。</td>
    </tr>
    <tr>
      <td><b>add/<< </b></td>
      <td>添加元素。</td>
    </tr>
    <tr>
      <td><b>intersect([2,3])</b></td>
      <td>取交集。</td>
    </tr>
    <tr>
      <td><b>disjoint([1])</b></td>
      <td>判断是否有交集。</td>
    </tr>
    <tr>
      <td><b>unique</b></td>
      <td>去重。 </td>
    </tr>
    <tr>
      <td><b>reverse/sort</b></td>
      <td>反转/升序</td>
    </tr>
    <tr>
      <td><b>flatten</b></td>
      <td>合并嵌套的列表</td>
    </tr>  
    <tr>
      <td><b>join</b></td>
      <td>将元素按照参数链接</td>
    </tr>  
    <tr>
      <td><b>sum/min/max</b></td>
      <td>求和、最小值、最大值</td>
    </tr> 
    <tr>
      <td><b>contains</b></td>
      <td>包含特定元素</td>
    </tr> 
    <tr>
      <td><b>remove/removeAll</b></td>
      <td>删除元素、删除所有元素</td>
    </tr> 
    <tr>
      <td><b>each{}</b></td>
      <td>遍历</td>
    </tr> 
</table>

示例：

```groovy
List arrays  = [1,2,3]
println("定义一个列表:" + arrays.toString())
arrays = arrays + 4
println("使用 + 添加元素:" + arrays.toString())
arrays = arrays << 5
println("使用 << 添加元素:" + arrays.toString())
arrays.add(6)
println("使用 add 方法:" + arrays.toString())

List arrays1 = [[9,10,11],[11,12,13]]
println("使用 flatten 方法:" + arrays1.flatten())
arrays1 = arrays1.flatten()
println("使用 unique 方法:" + arrays1.unique())
println("使用 reverse 方法:" + arrays1.reverse())
println("使用 join 方法:" + arrays1.join("-"))
println("使用 sum 方法:" + arrays1.sum())
println("使用 remove 方法:" + arrays1.remove(0))
arrays1.each{
    println("使用 each 方法:" + it)
}
```

<font color="green"><b>输出结果:</b></font>

<img src="./statics/images/groovy/groovy_list_type_02.png" style="zoom:100%;" />




## 3. <font color="#ff6702"><b>Map</b></font> 类型

- **`map`** 符号：**`[:]`**

- 常用方法，如下列表：

<style>
    .table11_3 table {
        width:100%;
        margin:15px 0;
        border:3px;
    }
    .table11_3 th {
        background-color:#46D77B;
        color:#706D75
    }
    .table11_3,.table11_3 th,.table11_3 td {
        font-size:1em;
        text-align:center;
        padding:14px;
        border-collapse:collapse;
    }
    .table11_3 th {
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 td {
        text-align:left;
        border: 1px solid #fe2047;
        border-width:1px 0 1px 0;
        border:1px inset #ffffff;
    }
    .table11_3 tr {
        border: 1px solid #ffffff;
    }
    .table11_3 tr:nth-child(odd){
        background-color:#F3F5F5;
    }
    .table11_3 tr:nth-child(even){
        background-color:#ffffff;
    }
</style>
<table class=table11_3>
    <tr>
     <th>方法</th>
     <th>含义</th>
    </tr>
    <tr>
      <td><b>size</b></td>
      <td>map大小</td></td>
    </tr>
    <tr>
      <td><b>['key']/.key get()</b></td>
      <td>获取map中key对应的value</td>
    </tr>
    <tr>
      <td><b>isEmpty()</b></td>
      <td>判断是否为空。</td>
    </tr>
    <tr>
      <td><b>containKey</b></td>
      <td>是否为包含指定Key。</td>
    </tr>
    <tr>
      <td><b>containValue</b></td>
      <td>是否包含指定的Value。</td>
    </tr>
    <tr>
      <td><b>keySet</b></td>
      <td>生成Key列表</td>
    </tr>
    <tr>
      <td><b>remove</b></td>
      <td>删除元素(k-v)</td>
    </tr> 
    <tr>
      <td><b>each{}</b></td>
      <td>遍历map</td>
    </tr> 
</table>
示例：

```groovy
Map map = [1:2,3:4,5:6,7:8]
println("map 定义：" + map.toString())
println("map[key] 获取value：" + map[1])
println("keySet方法：" + map.keySet())
println("values方法：" + map.values())
map += [9:10]
println("+ 元素方法：" + map.toString())
```

<font color="green"><b>输出结果:</b></font>

<img src="./statics/images/groovy/groovy_map_type_03.png" style="zoom:100%;" />
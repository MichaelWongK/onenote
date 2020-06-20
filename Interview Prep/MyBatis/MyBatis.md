XML转义字符

| &lt;   | <    | 小于号 |
| ------ | ---- | ------ |
| &gt;   | >    | 大于号 |
| &amp;  | &    | 和     |
| &apos; | ’    | 单引号 |
| &quot; | "    | 双引号 |

 

用转义字符进行替换

例如  

```
SELECT * FROM  user WHERE  age  &lt;= 30 AND age &gt;= 18
```

 

 

**另外：**xml格式中不允许出现类似“>”这样的字符，但是都可以使用<![CDATA[ ]]>符号进行说明，将此类符号不进行解析 

*上面的可以写成这个：
*

```
<![CDATA[ SELECT * FROM  user WHERE  age  <= 30 AND age >= 18 ]]>
```
# yaml语法

yaml：即是标记语言，又不是标记语言。yaml配置后缀：`.yaml`、`yml`

**基本特性：**

- 大小写敏感；
- 用缩进表示层级，且缩进只能使用空格，不允许用tab键；
- 缩进空格数不重要，只需要相同层级的元素左对齐即可；
- "#"表示注释，只支持单行注释

**数据类型：**

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）；
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）；
- 纯量（scalars）：单个的、不可再分的值；

**对象：**

对象键值对使用冒号结构表示`key: value`，冒号后面要加一个空格，且可以嵌套。

示例：

```yaml
key:
  child-key0: value0
  child-key1: value1
# 行内写法：
key: {child-key0:value0,child-key1:value1}
```

较为复杂的对象格式，可以使用问号加一个空格代表一个复杂的key，配合一个冒号加一个空格代表一个value。

```yaml
# key为数组:[key1,key2];value为数组:[value1,value2]
?  
  - key1
  - key2
:
  - value1
  - value2
```

**数组：**

支持两种写法：

- 行内写法：`key: [value1,value2,...]`

- 非行内写法：用`-`开头

  ```yaml
  key:
    - value1
    - value2
  ```

**纯量：**

纯量表示基本值，不可再分割的值。

- 字符串：可以用双引号或单引号包裹特殊字符；可以拆成多行，每一行会转化一个空格
- 布尔值
- 整数
- 浮点
- null
- 时间
- 日期
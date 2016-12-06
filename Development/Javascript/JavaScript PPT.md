# 第一张

## 数据类型

### 基础类型

> 除了null,可以通过typeof运算符识别

类型        | 名称           | 备注
--------- | ------------ | -------------------------------------
string    | 字符串          |
number    | 数字           |
boolean   | 布尔           |
undefined | 未定义          | 使用var声明变量但未对其加以初始化时，这个变量的值就是undefined
null      | 空            |
object    | 对象,是一种复杂数据类型 |

1. **数组和函数不是不是**
2. **null值表示一个空对象指针，而这也正是使用typeof操作符检测null时会返回"object"**
3. **undefined值是派生自null值的，因此ECMA-262规定对它们的相等性测试要返回true,管null和undefined有这样的关系，但它们的用途完全不同**(undefined == null)

### 类型转换

```javascript
var name = null;
if (name)? "default": name;
```

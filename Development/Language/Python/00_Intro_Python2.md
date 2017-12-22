Python2
=======================

# 简介
Python 是一个高层次的结合了解释性、编译性、互动性和面向对象的脚本语言。Python 的设计具有很强的可读性，相比其他语言经常使用英文关键字，其他语言的一些标点符号，它具有比其他语言更有特色语法结构。

1. Python 是一种解释型语言： 这意味着开发过程中没有了编译这个环节。类似于PHP和Perl语言。
2. Python 是交互式语言： 这意味着，您可以在一个Python提示符，直接互动执行写你的程序。
3. Python 是面向对象语言: 这意味着Python支持面向对象的风格或代码封装在对象的编程技术。
4. Python 是初学者的语言：Python 对初级程序员而言，是一种伟大的语言，它支持广泛的应用程序开发，从简单的文字处理到 WWW 浏览器再到游戏。

# 基础语法
## 中文编码
Python中默认的编码格式是 ASCII 格式，在没修改编码格式时无法正确打印汉字，所以在读取中文时会报错。
解决方法为只要在文件开头加入 # -*- coding: UTF-8 -*- 或者 #coding=utf-8 就行了

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
#coding=utf-8

print "你好，世界";
```
**注意**：Python3.X 源码文件默认使用utf-8编码，所以可以正常解析中文，无需指定 UTF-8 编码。
**注意**：如果你使用编辑器，同时需要设置 py 文件存储的格式为 UTF-8，否则会出现类似以下错误信息：

## 标识符
在 Python里，标识符由字母、数字、下划线组成。
在 Python中，所有标识符可以包括英文、数字以及下划线(_)，但不能以数字开头。
Python中的标识符是区分大小写的。
以下划线开头的标识符是有特殊意义的。以单下划线开头 _foo 的代表不能直接访问的类属性，需通过类提供的接口进行访问，不能用 from xxx import * 而导入；
以双下划线开头的 __foo 代表类的私有成员；以双下划线开头和结尾的 __foo__ 代表 Python 里特殊方法专用的标识，如 __init__() 代表类的构造函数；
Python 可以同一行显示多条语句，方法是用分号 ;



## 行和缩进
Python 的代码块不使用大括号 {} 来控制类，函数以及其他逻辑判断。python 最具特色的就是用缩进来写模块。

## 多行语句
Python语句中一般以新行作为为语句的结束符。
但是我们可以使用斜杠（ \ ）将一行的语句分为多行显示，如下所示：
```python
total = item_one + \
        item_two + \
        item_three
```

## 引号
Python 可以使用引号( ' )、双引号( " )、三引号( ''' 或 """ ) 来表示字符串，引号的开始与结束必须的相同类型的。
其中三引号可以由多行组成，编写多行文本的快捷语法，常用于文档字符串，在文件的特定地点，被当做注释。
```python
word = 'word'
sentence = "这是一个句子。"
paragraph = """这是一个段落。
包含了多个语句"""
```

## 注释
python中单行注释采用 # 开头。
python 中多行注释使用三个单引号(''')或三个双引号(""")。

```python

'''
这是多行注释，使用单引号。
这是多行注释，使用单引号。
'''

"""
这是多行注释，使用双引号。
这是多行注释，使用双引号。
"""
```

## 命令行参数
执行脚本传入参数，使用sys模块，sys.argv[0] 代表文件本身路径，所带参数从sys.argv[1] 开始。
```python
#!/usr/bin/python

import sys
print sys.argv
```

## 用户输入
```python
#!/usr/bin/python
v=raw_input("\nPress the enter key to exit.\n")
```

# 函数
函数是组织好的，可重复使用的，用来实现单一，或相关联功能的代码段。函数能提高应用的模块性，和代码的重复利用率。你已经知道Python提供了许多内建函数，比如print()。但你也可以自己创建函数，这被叫做用户自定义函数。

## 定义函数
你可以定义一个由自己想要功能的函数，以下是简单的规则：
1. 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号()。
2. 任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数。
3. 函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明。
4. 函数内容以冒号起始，并且缩进。
5. return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。

## 可更改(mutable)与不可更改(immutable)对象
在 python 中，strings, tuples, 和 numbers 是不可更改的对象，而 list,dict 等则是可以修改的对象。
1. 不可变类型：变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变a的值，相当于新生成了a。
2. 可变类型：变量赋值 la=[1,2,3,4] 后再赋值 la[2]=5 则是将 list la 的第三个元素值更改，本身la没有动，只是其内部的一部分值被修改了。

python 函数的参数传递：
1. 不可变类型：类似 c++ 的值传递，如 整数、字符串、元组。如fun（a），传递的只是a的值，没有影响a对象本身。比如在 fun（a）内部修改 a 的值，只是修改另一个复制的对象，不会影响 a 本身。
2. 可变类型：类似 c++ 的引用传递，如 列表，字典。如 fun（la），则是将 la 真正的传过去，修改后fun外部的la也会受影响
python 中一切都是对象，严格意义我们不能说值传递还是引用传递，我们应该说传不可变对象和传可变对象。

## 参数
以下是调用函数时可使用的正式参数类型：
### 必备参数
必备参数须以正确的顺序传入函数。调用时的数量必须和声明时的一样。

### 关键字参数
关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值。
使用关键字参数允许函数调用时参数的顺序与声明时不一致，因为 Python 解释器能够用参数名匹配参数值。
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

#可写函数说明
def printinfo( name, age ):
   "打印任何传入的字符串"
   print "Name: ", name;
   print "Age ", age;
   return;

#调用printinfo函数
printinfo( age=50, name="miki" );
```

### 默认参数
调用函数时，缺省参数的值如果没有传入，则被认为是默认值。
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

#可写函数说明
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print "Name: ", name;
   print "Age ", age;
   return;
```

### 不定长参数
你可能需要一个函数能处理比当初声明时更多的参数。这些参数叫做不定长参数，和上述2种参数不同，声明时不会命名。
```python
def functionname([formal_args,] *var_args_tuple ):
   "函数_文档字符串"
   function_suite
   return [expression]
```

## 匿名函数
python 使用 lambda 来创建匿名函数。
lambda只是一个表达式，函数体比def简单很多。
lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去。
lambda函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数。
虽然lambda函数看起来只能写一行，却不等同于C或C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率。
### 语法
```python
lambda [arg1 [,arg2,.....argn]]:expression
```

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2;

# 调用sum函数
print "相加后的值为 : ", sum( 10, 20 )
print "相加后的值为 : ", sum( 20, 20 )
```

## 变量作用域
一个程序的所有的变量并不是在哪个位置都可以访问的。访问权限决定于这个变量是在哪里赋值的。
变量的作用域决定了在哪一部分程序你可以访问哪个特定的变量名称。两种最基本的变量作用域如下：
1. 全局变量
2. 局部变量
定义在函数内部的变量拥有一个局部作用域，定义在函数外的拥有全局作用域。局部变量只能在其被声明的函数内部访问，而全局变量可以在整个程序范围内访问。调用函数时，所有在函数内声明的变量名称都将被加入到作用域中。

全局变量想作用于函数内，需加 global
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

globvar = 0

def set_globvar_to_one():
    global globvar    # 使用 global 声明全局变量
    globvar = 1

```

## 返回函数
```python
def funx(x):
    def funy(y):
        return x * y
    return funy    #return funy返回的是一个对象，可理解为funx是funy的一个对象
```

# 变量
每个变量在内存中创建，都包括变量的标识，名称和数据这些信息。
每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。
等号（=）运算符左边是一个变量名,等号（=）运算符右边是存储在变量中的值

## 标准数据类型
Python 定义了一些标准类型，用于存储各种类型的数据。Python有五个标准的数据类型：
### Numbers（数字）

Python支持四种不同的数字类型：
1. int（有符号整型）
2. long（长整型[也可以代表八进制和十六进制]）
长整型也可以使用小写 l，但是还是建议您使用大写 L，避免与数字 1 混淆。Python使用 L 来显示长整型。

3. float（浮点型）
4. complex（复数）
复数由实数部分和虚数部分构成，可以用 a + bj,或者 complex(a,b) 表示， 复数的实部 a 和虚部 b 都是浮点型。

### String（字符串）
python的字串列表有2种取值顺序:
* 从左到右索引默认0开始的，最大范围是字符串长度少1
* 从右到左索引默认-1开始的，最大范围是字符串开头
可以使用变量 [头下标:尾下标]截取相应的字符串，其中下标是从 0 开始算起，可以是正数或负数，下标可以为空表示取到头或尾。加号（+）是字符串连接运算符，星号（*）是重复操作。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

str = 'Hello World!'

print str           # 输出完整字符串
print str[0]        # 输出字符串中的第一个字符
print str[2:5]      # 输出字符串中第三个至第五个之间的字符串
print str[2:]       # 输出从第三个字符开始的字符串
print str * 2       # 输出字符串两次
print str + "TEST"  # 输出连接的字符串
```

#### 格式化
Python 支持格式化字符串的输出 。尽管这样可能会用到非常复杂的表达式，但最基本的用法是将一个值插入到一个有字符串格式符 %s 的字符串中。
```python
#!/usr/bin/python

print "My name is %s and weight is %d kg!" % ('Zara', 21)
```
符   号 |描述
--|--
    %c |格式化字符及其ASCII码
    %s | 格式化字符串
    %d | 格式化整数
    %u | 格式化无符号整型
    %o | 格式化无符号八进制数
    %x | 格式化无符号十六进制数
    %X | 格式化无符号十六进制数（大写）
    %f | 格式化浮点数字，可指定小数点后的精度
    %e | 用科学计数法格式化浮点数
    %E | 作用同%e，用科学计数法格式化浮点数
    %g | %f和%e的简写
    %G | %f 和 %E 的简写
    %p | 用十六进制数格式化变量的地址

### List（列表）
List（列表） 是 Python 中使用最频繁的数据类型。列表可以完成大多数集合类的数据结构实现。它支持字符，数字，字符串甚至可以包含列表（即嵌套）。列表用 [ ] 标识，是 python 最通用的复合数据类型。列表中值的切割也可以用到变量 [头下标:尾下标] ，就可以截取相应的列表，从左到右索引默认 0 开始，从右到左索引默认 -1 开始，下标可以为空表示取到头或尾。加号 + 是列表连接运算符，星号 * 是重复操作。

### Tuple（元组）
元组是另一个数据类型，类似于List（列表）。
元组用"()"标识。内部元素用逗号隔开。但是元组不能二次赋值，相当于只读列表。

### Dictionary（字典）
字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象集合，字典是无序的对象集合。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。字典用"{ }"标识。字典由索引(key)和它对应的值value组成。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

dict = {}
dict['one'] = "This is one"
dict[2] = "This is two"

tinydict = {'name': 'john','code':6734, 'dept': 'sales'}

print dict['one']          # 输出键为'one' 的值
print dict[2]              # 输出键为 2 的值
print tinydict             # 输出完整的字典
print tinydict.keys()      # 输出所有键
print tinydict.values()    # 输出所有值
```

## 类型转换
函数  |说明
--|--
int(x [,base])  |将x转换为一个整数
long(x [,base] )  |将x转换为一个长整数
float(x)  |将x转换到一个浮点数
complex(real [,imag])  |创建一个复数
str(x)  |将对象 x 转换为字符串
repr(x)  |将对象 x 转换为表达式字符串
eval(str)  |用来计算在字符串中的有效Python表达式,并返回一个对象
tuple(s)  |将序列 s 转换为一个元组
list(s)  |将序列 s 转换为一个列表
set(s)  |转换为可变集合
dict(d)  |创建一个字典。d 必须是一个序列 (key,value)元组。
frozenset(s)  |转换为不可变集合
chr(x)  |将一个整数转换为一个字符
unichr(x)  |将一个整数转换为Unicode字符
ord(x)  |将一个字符转换为它的整数值
hex(x)  |将一个整数转换为一个十六进制字符串
oct(x)  |将一个整数转换为一个八进制字符串


## 类型检查
python 的所有数据类型都是类,可以通过 type() 查看该变量的数据类型，还可以用 isinstance 来判断。
isinstance 和 type 的区别在于：
* type()不会认为子类是一种父类类型。
* isinstance()会认为子类是一种父类类型。

# 日期时间
Python 提供了一个 time 和 calendar 模块可以用于格式化日期和时间。时间间隔是以秒为单位的浮点小数。time 模块下有很多函数可以转换常见日期格式
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import time

print time.tzname

localtime = time.time()
print "当前时间戳为 :", localtime

print "CPU时钟 :", time.clock( )

localtime = time.localtime()
print "当前时间戳为 :", localtime
print time.mktime(localtime)

localtime = time.asctime(time.localtime())
print "本地时间字符为 :", localtime

print time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())

a = "Sat Mar 28 22:24:24 2016"
print time.strptime(a, "%a %b %d %H:%M:%S %Y")

print "sleep 2seconds"
time.sleep(2)
```
python中时间日期格式化符号：
* %y 两位数的年份表示（00-99）
* %Y 四位数的年份表示（000-9999）
* %m 月份（01-12）
* %d 月内中的一天（0-31）
* %H 24小时制小时数（0-23）
* %I 12小时制小时数（01-12）
* %M 分钟数（00=59）
* %S 秒（00-59）
* %a 本地简化星期名称
* %A 本地完整星期名称
* %b 本地简化的月份名称
* %B 本地完整的月份名称
* %c 本地相应的日期表示和时间表示
* %j 年内的一天（001-366）
* %p 本地A.M.或P.M.的等价符
* %U 一年中的星期数（00-53）星期天为星期的开始
* %w 星期（0-6），星期天为星期的开始
* %W 一年中的星期数（00-53）星期一为星期的开始
* %x 本地相应的日期表示
* %X 本地相应的时间表示
* %Z 当前时区的名称
* %% %号本身

## 日历
模块的函数都是日历相关的，例如打印某月的字符月历。星期一是默认的每周第一天，星期天是默认的最后一天。更改设置需调用calendar.setfirstweekday()函数。
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import calendar

cal = calendar.month(2016, 1)
print "以下输出2016年1月份的日历:"
print cal;
```

# 语句块
## if-else
语句形式
```python
if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……
```
## for
语句形式
```python
for letter in 'Python':     # 第一个实例
   print '当前字母 :', letter

fruits = ['banana', 'apple',  'mango']
for index in range(len(fruits)):
   print '当前水果 :', fruits[index]

print "Good bye!"
```

### for-else 语句
在 python 中，for … else 表示这样的意思，for 中的语句和普通的没有区别，else 中的语句会在循环正常执行完（**即 for 不是通过 break 跳出而中断的**）的情况下执行，while … else 也是一样。
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

for num in range(10,20):  # 迭代 10 到 20 之间的数字
   for i in range(2,num): # 根据因子迭代
      if num%i == 0:      # 确定第一个因子
         j=num/i          # 计算第二个因子
         print '%d 等于 %d * %d' % (num,i,j)
         break            # 跳出当前循环
   else:                  # 循环的 else 部分
      print num, '是一个质数'
```

## while
语句形式

```python
while 判断条件：
    执行语句……
```
### while-else 语句
while … else 在循环条件为 false 时执行 else 语句块
```python
#!/usr/bin/python

count = 0
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"
```

## 文件IO

### 打开文件
open 函数
你必须先用Python内置的open()函数打开一个文件，创建一个file对象，相关的方法才可以调用它进行读写。
语法：
```python
file object = open(file_name [, access_mode][, buffering])
```

各个参数的细节如下：
1. file_name：file_name变量是一个包含了你要访问的文件名称的字符串值。
2. access_mode：access_mode决定了打开文件的模式：只读，写入，追加等。所有可取值见如下的完全列表。这个参数是非强制的，默认文件访问模式为只读(r)。
3. buffering:如果buffering的值被设为0，就不会有寄存。如果buffering的值取1，访问文件时会寄存行。如果将buffering的值设为大于1的整数，表明了这就是的寄存区的缓冲大小。如果取负值，寄存区的缓冲大小则为系统默认。

模式|描述
--|--
r|以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
rb|以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。
r+|打开一个文件用于读写。文件指针将会放在文件的开头。
rb+|以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。
w|打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
wb|以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
w+|打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
wb+|以二进制格式打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
a|打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
ab|以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
a+|打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
ab+|以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。

### file属性
属性|描述
--|--
file.closed|返回true如果文件已被关闭，否则返回false。
file.mode|返回被打开文件的访问模式。
file.name|返回文件的名称。
file.softspace|如果用print输出后，必须跟一个空格符，则返回false。否则返回true。

方法|描述
--|--
fileno|fileno()返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如os模块的read方法等一些底层操作上。
close|刷新缓冲区里任何还没写入的信息，并关闭该文件，这之后便不能再进行写入
write|任何字符串写入一个打开的文件。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。不会在字符串的结尾添加换行符('\n')：
writelines|writelines(sequence)向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符。
read|read([count])从一个打开的文件中读取一个字符串。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。没有传入count，它会尝试尽可能多地读取更多的内容，很可能是直到文件的末尾。
readline|file.readline([size])读取整行，包括 "\n" 字符。
readlines|file.readlines([sizehint])读取所有行并返回列表，若给定sizeint>0，则是设置一次读多少字节，这是为了减轻读取压力。
next|next()返回文件下一行。
tell|告诉你文件内的当前位置；换句话说，下一次的读写会发生在文件开头这么多字节之后。
seek|seek（offset [,from]）方法改变当前文件的位置。Offset变量表示要移动的字节数。From变量指定开始移动字节的参考位置。from被设为0，这意味着将文件的开头作为移动字节的参考位置。如果设为1，则使用当前的位置作为参考位置。如果它被设为2，那么该文件的末尾将作为参考位置。

### OS模块操作文件方法
OS 对象方法: 提供了处理文件及目录的一系列方法。

序号|方法|描述
--|--|
1|os.access(path, mode)|检验权限模式
2|os.chdir(path)|改变当前工作目录
3|os.chflags(path, flags)|设置路径的标记为数字标记。
4|os.chmod(path, mode)|更改权限
5|os.chown(path, uid, gid)|更改文件所有者
6|os.chroot(path)|改变当前进程的根目录
7|os.close(fd)|关闭文件描述符 fd
8|os.closerange(fd_low, fd_high)|关闭所有文件描述符，从 fd_low (包含) 到 fd_high (不包含), 错误会忽略
9|os.dup(fd)|复制文件描述符 fd
10|os.dup2(fd, fd2)|将一个文件描述符 fd 复制到另一个 fd2
11|os.fchdir(fd)|通过文件描述符改变当前工作目录
12|os.fchmod(fd, mode)|改变一个文件的访问权限，该文件由参数fd指定，参数mode是Unix下的文件访问权限。
13|os.fchown(fd, uid, gid)|修改一个文件的所有权，这个函数修改一个文件的用户ID和用户组ID，该文件由文件描述符fd指定。
14|os.fdatasync(fd)|强制将文件写入磁盘，该文件由文件描述符fd指定，但是不强制更新文件的状态信息。
15|os.fdopen(fd[, mode[, bufsize]])|通过文件描述符 fd 创建一个文件对象，并返回这个文件对象
16|os.fpathconf(fd, name)|返回一个打开的文件的系统配置信息。name为检索的系统配置的值，它也许是一个定义系统值的字符串，这些名字在很多标准中指定（POSIX.1, Unix 95, Unix 98, 和其它）。
17|os.fstat(fd)|返回文件描述符fd的状态，像stat()。
18|os.fstatvfs(fd)|返回包含文件描述符fd的文件的文件系统的信息，像 statvfs()
19|os.fsync(fd)|强制将文件描述符为fd的文件写入硬盘。
20|os.ftruncate(fd, length)|裁剪文件描述符fd对应的文件, 所以它最大不能超过文件大小。
21|os.getcwd()|返回当前工作目录
22|os.getcwdu()|返回一个当前工作目录的Unicode对象
23|os.isatty(fd)|如果文件描述符fd是打开的，同时与tty(-like)设备相连，则返回true, 否则False。
24|os.lchflags(path, flags)|设置路径的标记为数字标记，类似 chflags()，但是没有软链接
25|os.lchmod(path, mode)|修改连接文件权限
26|os.lchown(path, uid, gid)|更改文件所有者，类似 chown，但是不追踪链接。
27|os.link(src, dst)|创建硬链接，名为参数 dst，指向参数 src
28|os.listdir(path)|返回path指定的文件夹包含的文件或文件夹的名字的列表。
29|os.lseek(fd, pos, how)|设置文件描述符 fd当前位置为pos, how方式修改: SEEK_SET 或者 0 设置从文件开始的计算的pos; SEEK_CUR或者 1 则从当前位置计算; os.SEEK_END或者2则从文件尾部开始. 在unix，Windows中有效
30|os.lstat(path)|像stat(),但是没有软链接
31|os.major(device)|从原始的设备号中提取设备major号码 (使用stat中的st_dev或者st_rdev field)。
32|os.makedev(major, minor)|以major和minor设备号组成一个原始设备号
33|os.makedirs(path[, mode])|递归文件夹创建函数。像mkdir(), 但创建的所有intermediate-level文件夹需要包含子文件夹。
34|os.minor(device)|从原始的设备号中提取设备minor号码 (使用stat中的st_dev或者st_rdev field )。
35|os.mkdir(path[, mode])|以数字mode的mode创建一个名为path的文件夹.默认的 mode 是 0777 (八进制)。
36|os.mkfifo(path[, mode])|创建命名管道，mode 为数字，默认为 0666 (八进制)
37|os.mknod(filename[, mode=0600, device])|创建一个名为filename文件系统节点（文件，设备特别文件或者命名pipe）。
38|os.open(file, flags[, mode])|打开一个文件，并且设置需要的打开选项，mode参数是可选的
39|os.openpty()|打开一个新的伪终端对。返回 pty 和 tty的文件描述符。
40|os.pathconf(path, name)|返回相关文件的系统配置信息。
41|os.pipe()|创建一个管道. 返回一对文件描述符(r, w) 分别为读和写
42|os.popen(command[, mode[, bufsize]])|从一个 command 打开一个管道
43|os.read(fd, n)|从文件描述符 fd 中读取最多 n 个字节，返回包含读取字节的字符串，文件描述符 fd对应文件已达到结尾, 返回一个空字符串。
44|os.readlink(path)|返回软链接所指向的文件
45|os.remove(path)|删除路径为path的文件。如果path 是一个文件夹，将抛出OSError; 查看下面的rmdir()删除一个 directory。
46|os.removedirs(path)|递归删除目录。
47|os.rename(src, dst)|重命名文件或目录，从 src 到 dst
48|os.renames(old, new)|递归地对目录进行更名，也可以对文件进行更名。
49|os.rmdir(path)|删除path指定的空目录，如果目录非空，则抛出一个OSError异常。
50|os.stat(path)|获取path指定的路径的信息，功能等同于C API中的stat()系统调用。
51|os.stat_float_times([newvalue])|决定stat_result是否以float对象显示时间戳
52|os.statvfs(path)|获取指定路径的文件系统统计信息
53|os.symlink(src, dst)|创建一个软链接
54|os.tcgetpgrp(fd)|返回与终端fd（一个由os.open()返回的打开的文件描述符）关联的进程组
55|os.tcsetpgrp(fd, pg)|设置与终端fd（一个由os.open()返回的打开的文件描述符）关联的进程组为pg。
56|os.tempnam([dir[, prefix]])|返回唯一的路径名用于创建临时文件。
57|os.tmpfile()|返回一个打开的模式为(w+b)的文件对象 .这文件对象没有文件夹入口，没有文件描述符，将会自动删除。
58|os.tmpnam()|为创建一个临时文件返回一个唯一的路径
59|os.ttyname(fd)|返回一个字符串，它表示与文件描述符fd 关联的终端设备。如果fd 没有与终端设备关联，则引发一个异常。
60|os.unlink(path)|删除文件路径
61|os.utime(path, times)|返回指定的path文件的访问和修改的时间。
62|os.walk(top[, topdown=True[, onerror=None[, followlinks=False]]])|输出在文件夹中的文件名通过在树中游走，向上或者向下。
63|os.write(fd, str)|写入字符串到文件描述符 fd中. 返回实际写入的字符串长度

## 异常
异常即是一个事件，该事件会在程序执行过程中发生，影响了程序的正常执行。一般情况下，在Python无法正常处理程序时就会发生一个异常。异常是Python对象，表示一个错误。当Python脚本发生异常时我们需要捕获处理它，否则程序会终止执行。

捕捉异常可以使用try/except语句。try/except语句用来检测try语句块中的错误，从而让except语句捕获异常信息并处理。如果你不想在异常发生时结束你的程序，只需在try里捕获它。

语法：
```python
以下为简单的try....except...else的语法：
try:
<语句>        #运行别的代码
except <名字>：
<语句>        #如果在try部份引发了'name'异常
except <名字>，<数据>:
<语句>        #如果引发了'name'异常，获得附加的数据
else:
<语句>        #如果没有异常发生
```

try的工作原理是，当开始一个try语句后，python就在当前程序的上下文中作标记，这样当异常出现时就可以回到这里，try子句先执行，接下来会发生什么依赖于执行时是否出现异常。
如果当try后的语句执行时发生异常，python就跳回到try并执行第一个匹配该异常的except子句，异常处理完毕，控制流就通过整个try语句（除非在处理异常时又引发新的异常）。
如果在try后的语句里发生了异常，却没有匹配的except子句，异常将被递交到上层的try，或者到程序的最上层（这样将结束程序，并打印缺省的出错信息）。
如果在try子句执行时没有发生异常，python将执行else语句后的语句（如果有else的话），然后控制流通过整个try语句。

## 模块
模块(Module)，是一个 Python 文件，以 .py 结尾，包含了 Python 对象定义和Python语句。模块让你能够有逻辑地组织你的 Python 代码段。把相关的代码分配到一个模块里能让你的代码更好用，更易懂。模块能定义函数，类和变量，模块里也能包含可执行的代码。

## 引入模块
一个模块只会被导入一次，不管你执行了多少次import。这样可以防止导入模块被一遍又一遍地执行。

```python
import module1[, module2]

# 把一个模块的所有内容全都导入到当前的命名空间
from modname import *

# from 语句让你从模块中导入一个指定的部分到当前命名空间中
from modname import name1[, name2]

```

## 搜索路径
当你导入一个模块，Python 解析器对模块位置的搜索顺序是：
1. 当前目录
2. 如果不在当前目录，Python 则搜索在 shell 变量 PYTHONPATH 下的每个目录。
3. 如果都找不到，Python会察看默认路径。UNIX下，默认路径一般为/usr/local/lib/python/。
模块搜索路径存储在 system 模块的 sys.path 变量中。变量里包含当前目录，PYTHONPATH和由安装过程决定的默认目录。

## 命名空间和作用域
变量是拥有匹配对象的名字（标识符）。命名空间是一个包含了变量名称们（键）和它们各自相应的对象们（值）的字典。
一个 Python 表达式可以访问局部命名空间和全局命名空间里的变量。如果一个局部变量和一个全局变量重名，则局部变量会覆盖全局变量。
每个函数都有自己的命名空间。类的方法的作用域规则和通常函数的一样。
Python 会智能地猜测一个变量是局部的还是全局的，它假设任何在函数内赋值的变量都是局部的。
因此，如果要给函数内的全局变量赋值，必须使用 global 语句。
global VarName 的表达式会告诉 Python， VarName 是一个全局变量，这样 Python 就不会在局部命名空间里寻找这个变量了。

## reload() 函数
当一个模块被导入到一个脚本，模块顶层部分的代码只会被执行一次。
因此，如果你想重新执行模块里顶层部分的代码，可以用 reload() 函数。该函数会重新导入之前导入过的模块。语法如下：
```python
reload(module_name)
```

## 包
包是一个分层次的文件目录结构，它定义了一个由模块及子包，和子包下的子包等组成的 Python 的应用环境。
简单来说，包就是文件夹，但该文件夹下必须存在 __init__.py 文件, 该文件的内容可以为空。__int__.py用于标识当前文件夹是一个包。
```file
test.py
package_runoob
|-- __init__.py
|-- runoob1.py
|-- runoob2.py
```

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# 导入 Phone 包
from package_runoob.runoob1 import runoob1
from package_runoob.runoob2 import runoob2

runoob1()
runoob2()
```

# 面向对象

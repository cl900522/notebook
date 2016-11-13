---
author: "Chenmx"
date: "2016-11-12 23:15"
---
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [引用](#引用)
	- [第一种方式](#第一种方式)
	- [第二种方式](#第二种方式)
- [强调](#强调)
- [列表](#列表)
- [代码](#代码)
- [链接](#链接)
	- [url超链接](#url超链接)
	- [图片链接](#图片链接)
	- [简短的链接](#简短的链接)
- [表格](#表格)
- [脚注](#脚注)
- [复选框](#复选框)
- [流程图](#流程图)

<!-- /TOC -->
Markdown教程
=======================================

Markdown 实例
---------------------------------------

# 引用
## 第一种方式
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
>
>> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
>>id sem consectetuer libero luctus adipiscing.

## 第二种方式
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
id sem consectetuer libero luctus adipiscing.

---------------------------------------

# 强调
*single asterisks*

_single underscores_

**double asterisks**

__double underscores__

\*this text is surrounded by literal asterisks\*

\_\_Not Strong\_\_

---------------------------------------
# 列表
*   Red
+   Green
-   Blue
3.  Black
*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
*  Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.

--------------------------------------
# 代码
`public static void main(String[] args) {`
&nbsp;&nbsp;&nbsp;&nbsp;`System.out.println("HelloWord!");`
`}`

``` java
/**
 * The main function of java class
 */
public static void main(String[] args) {
    System.out.println("HelloWord!");
}
```

A single backtick in a code span: `` ` ``

--------------------------------------
# 链接
## url超链接
This is [an example](http://example.com/ "Title") inline link.
[This link](http://example.net/) has no title attribute.
[ToGoogle][Google] has no title attribute.
See my [Self](./Markdown.md) page for details.

## 图片链接
![an example](./images/m.jpg "Tip")

## 简短的链接
<http://example.com/>

--------------------------------------
# 表格
编码  |名称   |备注
:--:|:---|:--
CODE001  |错误001   |经常发生
CODE002  |错误2   |少
CODE003  |错误3   |很少发生
SUCCESS  |成功    |

--------------------------------------
# 脚注
使用 [^Google] 表示注脚。

--------------------------------------
# 复选框
- [ ] 茄子
- [x] 萝卜

--------------------------------------
# 流程图

```flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

[Google]: http://google.com/ "Google"

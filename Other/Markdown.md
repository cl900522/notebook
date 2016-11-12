---
author: "Chenmx"
date: "2016-11-12 23:15"
---

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

[Google]: http://google.com/ "Google"

--------------------------------------
# 表格
编码  |名称   |备注
--|---|--
CODE001  |错误1   |经常发生
CODE002  |错误2   |
CODE003  |错误3   |很少发生
SUCCESS  |成功    |

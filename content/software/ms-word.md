---
title: "MS-Word使用技巧汇总"
layout: page
date: 2016-05-26
---

# 关于
微软的word文档是一个很好的文字处理工具，但是有时候也需要一些技巧。
比如在撰写大论文的时候，高效的排版技巧是非常必要的，往往能够减少
很多不必要的时间浪费。本文汇总了我在使用word文档时的一些技巧，
大多数是从网上总结别人的经验得到的，也有一些自己摸索出来的技巧。
如果你有其他需求，最好放狗搜，一般都能搜索到。

# 格式篇
## 长文档格式的一致性
  通过对段落添加式样标签，如“标题1”，“正文”等，而不是对每一个段落
  手动添加格式。这样的好处是，当需要更改所有相同段落的时候，比如
  需要将所有正文段落改成四号字体，只需要更改式样标签的式样即可。
  如果你熟悉HTML标签就会知道，这相当于给一个内容块添加了一个标签，
  如果要修改这个标签的式样，只需要为这个标签添加格式即可，而不用
  在每一个段落上申明它的格式。总之一个原则**一定不要手动为一个文本
  单独添加格式！**
  
![格式式样模板](/wiki/static/images/ms-word-style.png)


## 公式排版
### 公式对齐
制表位与对齐。如果你需要排版数学公式，需要将公式居中，而将公式的
编号放在行末并右对齐。显然用空格来控制格式不是一个好主意。可以采用
制表位实现，创建新式样公式，并修改格式，在制表位中添加两个制表位，
20居中和40右对齐，然后在公式前和编号前分别添加制表符即可。效果如下图。
总之一个原则**尽量不要通过空格来调整格式！**

![公式](/wiki/static/images/ms-word-formula.png)


### 公式自动编号
“插入”->“引用”->“题注”插入公式编号；
在需要引用公式的地方用“插入”->“引用”->“交叉引用”引用公式编号，
可以选择“整项题注”引用公式编号行的所有内容。

## 页眉自动根据章节标题自动插入：
可以采用域的方式在文中的页眉中自动插入标题的编号或名称：  
（1）双击页眉位置，页眉处于可编辑状态；   
（2）选择插入-文档部件-域   
（3）选择“类别-链接和引用”下的styleref，选择章节使用的“样式名”（要使用样式哦），“域选项”里不选任何项即插入章节标题；若“域选项”框中选择了“插入段落编号”，则插入章节编号。

## 文档分节
为什么要分节？

1. 分节可以对每一节应用不同的页眉、页脚和页码。
2. 分节文档转换为PDF时，可以自动保证每一节的第一页在奇数页。

## 分节编制页码
首先我们需要把封面页和正文页进行分节，首先将封面页和正文的第一页放在一起，中间不要间隔，然后在正文的第一个字之前点击光标，插入分节符。

然后取消链接到前一条页眉，最后设置页码格式中起始页码为1.

参考 <http://jingyan.baidu.com/article/4f7d57129d9c501a201927ea.html>




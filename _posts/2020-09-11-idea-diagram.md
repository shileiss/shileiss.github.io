---
title: "使用IntelliJ IDEA查看类图"
date: 2020-09-11 15:00:00 +0800
category: IntelliJ IDEA
tags: [IntelliJ IDEA]
excerpt: 本文主要主要说明IntelliJ IDEA查看类图的方法。
---

# 使用 IntelliJ IDEA 查看类图

>最近正好也没什么可忙的，就回过头来鼓捣过去的知识点，到Servlet部分时，以前学习的时候硬是把从上到下的继承关系和接口实现记得乱七八糟。

>这次利用了IDEA的diagram，结果一目了然，也是好用到炸裂，就此分享。


## 查看图形形式的继承链

在你想查看的类的标签页内，点击右键，选择 Diagrams，其中有 show 和 show ... Popup，只是前者新建在标签页内，后者以浮窗的形式展示：



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjuF4bW1MJuehzs9hP4WP1F7qU1NAzX8M5LicK9jNcrwjDvublNrG9bAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

实际上，你也可以从左边的项目目录树中，对你想查看的类点击右键，同样选择Diagrams，效果是一样的：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjTzYNMoicFOLDBdZXo2lqXRAxxq9NQ5RhDjDb3icOLbxt9Ricq0iaTkExdA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后你就会得到如下图所示的继承关系图形，以自定义的Servlet为例：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8Njw89znjO0iaoKG6Y2xBHwMPozubzHNVicVIv6e5JW0I0ebb6BicFzIa7bw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

显而易见的是：

- **蓝色实线箭头**是指继承关系
- **绿色虚线箭头**是指接口实现关系

优化继承链图形，想我所想



## 1 去掉不关心的类

得到的继承关系图形，有些并不是我们想去了解的，比如上图的Object和Serializable，我们只想关心Servlet重要的那几个继承关系，怎么办？

简单，删掉。点击选择你想要删除的类，然后直接使用键盘上的delete键就行了。清理其他类的关系后图形如下：



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8Nj8oxcSCPXjoqBVOHJcNwfNjnUK5sPt28y03MomcZF1BXEvPjwZIA3hQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 2 展示类的详细信息

有人说，诶，这怎么够呢，那继承下来的那些方法我也想看啊？简单，IDEA通通满足你。

在页面点击右键，选择 show categories，根据需要可以展开类中的属性、方法、构造方法等等。当然，第二种方法也可以直接使用上面的工具栏：



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjwgoyDK1yiaibe96otHB7lx1TTLYERqiayKbmicF4HwRQneXuXIZNvCxbQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后你就会得到：



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjdYD9LndCthibaYNbzGBblrXcStXPb6Nvygjl5klHnDjCDa4N52zR5Ww/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

什么，方法里你还想筛选，比如说想看protected权限及以上范围的？简单，右键选择 Change Visibility Level，根据需要调整即可。



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjdYD9LndCthibaYNbzGBblrXcStXPb6Nvygjl5klHnDjCDa4N52zR5Ww/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

什么，你嫌图形太小你看不清楚？IDEA也可以满足你，按住键盘的Alt，竟然出现了放大镜，惊不惊喜，意不意外？



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjibV0d8Fd6JRQA1hbVtzplVh4Ttic41A31MibYpIWBQkUDBhmBzOxJE5MA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 3 加入其他类到关系中来

当我们还需要查看其他类和当前类是否有继承上的关系的时候，我们可以选择加其加入到当前的继承关系图形中来。

在页面点击右键，选择 Add Class to Diagram，然后输入你想加入的类就可以了：



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjOfQ5DKaePR9XU27xQkQvPVEMRFG5Npns7GbPeOdvrkdap5Jk8oEyQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

例如我们添加了一个Student类，如下图所示。好吧，并没有任何箭头，看来它和当前这几个类以及接口并没有发生什么不可描述的关系：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjPke2gzoqLf4Dnic0jQx20tQdNmR24OZGk3BjRibaFBHIuykQdG6yxJeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 4 查看具体代码

如果你想查看某个类中，比如某个方法的具体源码，当然，不可能给你展现在图形上了，不然屏幕还不得撑炸？

但是可以利用图形，或者配合IDEA的structure方便快捷地进入某个类的源码进行查看。

双击某个类后，你就可以在其下的方法列表中游走，对于你想查看的方法，选中后点击右键，选择 Jump to Source：



![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8Nj9XFibLWcnaJSzAoYob1OO5LG548LchOiad0iaTRdFqIXImxXsqV0tnkWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjsLkFI95w4Jmvvficf2tvC8nu3rmhnib4INvibiaLMf9IQ9jg6Am7ibZ8iaPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在进入某个类后，如果还想快速地查看该类的其他方法，还可以利用IDEA提供的structure功能：

![img](https://mmbiz.qpic.cn/mmbiz_png/y5HvXaQmpqklc745lMyNzPr9OvcHP8NjTnibsrv6M6ibmp3JuIxhUickjE5P1KjDNMdR6WB7fVRIf7hsmo8kicEsJA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

选择左侧栏的structure之后，如上图左侧会展示该类中的所有方法，点击哪个方法，页面内容就会跳转到该方法部分去。



最后



用上面提到的的IDEA这些功能，学习和查看类关系，了解诸如主流框架源码之类的东西，可以说是非常舒服了。

来源：www.cnblogs.com/deng-cc/p/6927447.html
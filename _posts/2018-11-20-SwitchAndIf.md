---
layout: post
title:  "C/C++中Switch..case和If..else的定量分析"
date:   2018-11-7
excerpt: "简单讨论了python的Switch..Case以及对C/C++中Switch..case和If..else的定量分析(这个是重点)"
tag:
- C/C++
- article
comments: true
---

写在前面：经konge大佬的指正，这篇文章关于C/C++的所有内容均建立在没有进行优化（-O0）的情况下，且我不保证它的完全正确。

Python没有自带switch…case结构，他们在[官方文档中的faq](https://docs.python.org/2/faq/design.html#why-isn-t-there-a-switch-or-case-statement-in-python/)里谈到：

>Why isn’t there a switch or case statement in Python?

问：为什么python妹有switch case语句哩？

>You can do this easily enough with a sequence of if... elif... elif... else.

答：if… elif挺简单的，您就这么用着吧。

当然，回答并不只有一句，回答的后面也给出了当分支很多的时候的解决方案，即使用字典实现。原文如下：

>For cases where you need to choose from a very large number of possibilities, you can create a dictionary mapping case values to functions to call.

为什么要使用字典呢？

在C中，switch…case在编译过程中会构建一个跳转表，然后通过查找来降低时间复杂度，提高程序的运行效率，而不是像if…else那样一个一个找过去。因此，python给出了用字典实现快速查找的方案。

但显而易见，要使用跳转表，就不可能像if可以做到的那样，在switch…case的条件中加入一大堆表达式、其他变量和函数。比如，有时可能需要判断90<x<100啥的，这就不能用switch…case实现了。

在这个问题下，官方文档同时提到在[PEP275](https://www.python.org/dev/peps/pep-0275/)可以康到详细的内容（Python增强建议书，Python Enhancement Proposals的简写）。

相当遗憾的是，作者的小学生英文水平实在过于丢人，就不给出PEP275的翻译了。PEP275大概讲的是：

你们不是嫌if… elif慢吗？我们这儿有两个方法。

第一个是优化Python对if… elif的编译；

第二个是给python添加switch句法。

>This PEP proposes two different but not necessarily conflicting solutions:
>
>1	Adding an optimization to the Python compiler and VM which detects the above if-elif-else construct and generates special opcodes for it which use a read-only dictionary for storing jump offsets.
>
>2	Adding new syntax to Python which mimics the C style switch statement.

看到没，第一行里那几个大大的but not necessarily（doge脸

PEP275后面的内容，讲的就是两种方法的细节以及例子了。如果对此有兴趣，窝个人还是建议点进原文去康一眼。

----

但我想说的并不是这个，甚至我也不想讨论python。

（我想讨论的是C呀）（小声）

我更想知道的是，if…else结构和通过跳转表实现的switch…case结构，具体的用时差别是多少呢？

从上面的分析我们很容易知道，在判断分支较多时，switch…case要优于if…else。但是，对于较多是多少，switch…case又比if…else快多少，很遗憾的是我没有能在网上找到（特别是C的）if…else和switch…case速度的定量比较。

在查阅资料的过程中，我发现了一些有帮助的资料：

[破乎的这篇回答](https://www.zhihu.com/question/300975864/answer/524449404)里提到了jvm对switch的优化。

这篇回答的最后，又提到了[国外一个论坛中相关的讨论](http://forums.xkcd.com/viewtopic.php?f=11&t=33524)。

呃，实际上这个帖子的讨论已经相当全面了，里面比较详细的讲了java中switch...case的问题。包括楼主在内的不少人统计了java执行switch...case的时间，还给画了图。结论上讲，java中只有switch分支较多时，优化效果才会比较显著（具体来说，是450个分支左右之后）。

下面这张，是原帖中的if...else和switch...case的对比图：

![一张对比图](http://forums.xkcd.com/download/file.php?id=10740&sid=9bd999f1e079cdb68dba9ca65cf1a5dd)

甚至在340个分支以下，if...else还快一点儿哩。

我们把话题回到C上去。

在那个帖子里，有一个叫Karrion的哥们儿，他不但测了java的switch...case，还去测了c的switch…case，并画出了一条相当漂亮的曲线，证明C的优化还是不错的。

![image](http://forums.xkcd.com/download/file.php?id=10727&sid=9bd999f1e079cdb68dba9ca65cf1a5dd)

Karrion只给出了switch…case在1到5000个分支时的时间。但我认为，这个测试只能证明C的优化很不错，但无法对实际编程起到太多的指导作用。毕竟，在平时写程序时几乎用不到500甚至5000个分支。我个人认为，在平时能用到20-30个分支已经算是较多的了。因此我更想知道的是，C中在分支数较小时，if…else和switch…case的差别有多大呢？

我很好奇(千反田脸)。

因此，我打算实际去测一测。

实现的脚本我直接从Karrion老哥那儿抄了一下，当然，在if...else实现时有所改动。考虑到当switch...case与if...else可以相互代替时，if...else应为只判断变量是否相等，因此测试中，if...else括号中的判断条件是两变量值相等。

分支个数1-100结果统计如下：

图：
[![switch和if.png](https://i.loli.net/2018/11/20/5bf3af9ae413c.png)](https://i.loli.net/2018/11/20/5bf3af9ae413c.png)

分支个数|switch|if…else
---|---|---
1|0.784|0.815
2|1.88|0.784
3|2.223|0.858
4|2.706|1.099
5|2.863|0.931
6|2.831|1.049
7|3.073|0.968
8|2.859|1.519
9|2.89|1.078
10|2.916|1.076
11|2.998|1.154
12|3.622|2.17
13|3.38|1.318
14|3.106|1.405
15|3.029|2.095
16|3.495|2.059
17|3.259|1.599
18|3.012|1.848
19|3.06|1.723
20|3.091|3.072
21|3.221|1.951
22|3.038|2.035
23|3.082|2.098
24|3.05|3.017
25|3.145|2.312
26|3.651|2.261
27|3.06|2.327
28|3.509|2.878
29|3.101|2.398
30|3.108|2.488
31|3.641|2.628
32|3.773|3.681
33|3.642|2.716
34|3.164|2.751
35|3.467|3.146
36|3.186|2.823
37|3.543|2.943
38|3.537|3.292
39|4.128|3.249
40|3.085|3.289
41|3.151|3.323
42|3.456|3.592
43|3.397|3.827
44|3.205|3.919
45|3.313|5.402
46|3.652|4.065
47|3.131|4.407
48|3.094|3.721
49|3.183|3.759
50|3.398|5.12
51|3.497|3.875
52|3.098|3.856
53|3.164|3.986
54|3.099|4.047
55|3.241|4.779
56|3.365|6.052
57|3.481|4.31
58|3.545|4.226
59|3.614|4.348
60|3.564|4.271
61|3.268|7.153
62|3.157|5.142
63|3.165|4.411
64|3.362|4.617
65|3.874|5.212
66|3.164|4.55
67|3.626|4.564
68|5.731|4.709
69|3.76|5.472
70|3.146|5.956
71|3.665|4.816
72|3.188|6.279
73|3.953|4.831
74|3.475|5.112
75|3.543|6.316
76|3.947|6.416
77|3.692|5.363
78|4.864|6.565
79|3.657|5.561
80|3.783|6.01
81|3.143|5.431
82|3.655|5.672
83|3.181|6.528
84|3.246|5.375
85|3.186|6.994
86|3.5|5.669
87|3.705|5.888
88|3.235|6.103
89|3.492|6.427
90|3.221|7.737
91|3.793|6.37
92|3.215|8.938
93|3.202|6.837
94|4.535|6.768
95|3.405|6.851
96|3.222|7.1
97|3.72|7.92
98|3.418|8.712
99|3.232|10.021
100|3.508|7.583

从图上可以看出，switch...case用时固定，而if...else用时稳步上升，符合之前的分析。而更为重要的是，在分支多于40个时，switch...case才显现出了速度上的优势。而在分支40个以下时，switch...case甚至要慢于if...else。当然，实际上在40个以下分支时，switch...case结构也不算慢；而如果分支极多，那么if...else的效率就会相当差。

这篇文章并没有讨论switch和if对于代码可读性的影响，只是在性能上进行分析。

但总而言之，从结果来看，如果分支不多，但为了性能选用switch...case的方式显然是不那么妥当的。在if...else与switch...case的选择中，只要分支不太多，最好还是从代码可读性的角度去选择这两种结构，而不是考虑性能。

PS：如果这篇文章哪里出现了错误，还请务必联系一下作者，让他赶紧改正，回头是岸，立地成佛。
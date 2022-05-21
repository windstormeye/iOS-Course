这是我在学习一套完整的学习手册帮助自己准备 Google 的面试的笔记，其想通过这种方式去提升自己姿势水平，链接在此，欢迎大家一块学习。[https://github.com/jwasham/coding-interview-university/blob/master/translations/README-cn.md](https://github.com/jwasham/coding-interview-university/blob/master/translations/README-cn.md)


### 2018-04-25

1. 今天主要的学习内容是关于CPU执行指令、机器码指令转换、编译器工作等内容，算是给自己复习了之前的内容吧。🙄（图较大，再加上国外图床，耐心等待喔）
<img src="https://i.loli.net/2018/04/25/5ae017405bbbc.jpeg" width = "100%" height = "100%" align=center />

2. 分享一个有趣的命令，当我们使用`clang`编译器去编译程序时(C/C++、Objective-C/C++、Swift)，可以使用`clang -S filePath`来查看经过编译器处理完后的汇编代码。如下所示，

这是原C代码：
<img src="https://i.loli.net/2018/04/25/5ae015b5d3dcf.png" width = "100%" height = "100%" align=center />

这是经过编译器转换之后的汇编代码：
<img src="https://i.loli.net/2018/04/25/5ae015b5668c8.png" width = "100%" height = "100%" align=center />

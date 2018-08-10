这篇文章主要是记录我在把主Objective-C的项目迁移到Swift上遇到的问题总结，不得不说，真的很坑。🙄

1. 第一次创建Swift文件时，会提示是否创建桥接文件，如果选了，会在当前路径下创建出桥接文件，当移动该桥接文件时，记得去，Build Setting -> Swift Compiler - General -> Objective-C Bridging Header，修改为调整后的路径即可。

2. Swift中的权限访问控制。http://www.hangge.com/blog/cache/detail_524.html

3. Swift中的指定构造器（designated Initializer）必须有一个，而且对其中初始化的所有属性，必须要在super方法之后。https://www.cnblogs.com/sunshine-anycall/p/4036932.html

4. OC和Swift互相调用。https://www.jianshu.com/p/754396e7e1bd

5. 遇到了一个迷之问题，OC调Swift方法，一直编译不过，看了对应的product Name-Swift.h文件后，发现无用tableView代理方法也暴露出来了，这样不是很好，就给所以相关代理方法加了private，Xcode提示不行，要用internal，改完后，居然神TM编译过了，最后琢磨来琢磨去，还是没搞懂这是为啥，去相关讨论群中咨询时，把internal关键词删掉，又神特么编译过了。。。到现在都不知道是为啥，感觉Swift是不是不太稳定。。

6. Swift中没有NSBundle了，叫Bundle。😑

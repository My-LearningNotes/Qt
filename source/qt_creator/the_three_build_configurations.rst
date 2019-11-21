Debug, Profile和Release三种构建模式之间的区别
=============================================

在Qt Creator中创建一个项目之后，默认情况下，每个构建套件都有三种构建配置：Debug，Profile和Release，这三种构建配置之间的区别是什么呢?

-  Debug通常称为调试版本，它包含了调试信息，并且不作任何优化，便于程序员调试程序；

-  Release称为发布版本，它往往是进行了各种优化，使得程序大小和运行速度上都是最优的，以便用户很好地使用；

-  Profile is *Release-with-debug-symbols*

      This option produce realize build(with all compiler optimization)
      but with debug symbols(pdb files) that are required for testing
      performance of C++.

   Profile版本是一种介于Debug和Release之间的版本，只要用来进行性能测试。

   Profile版本进行了各种优化，但是又包含调试符号。



.. warning::

   需要注意的是，如果程序中用到了其它的函数库，函数库也分为Debug版和Release版，Debug版和Release版是不能混合使用的。

   -  编译Debug版的程序，需要使用Debug版本的函数库，不能使用Release版本的函数库；

   -  编译Release版的程序，需要使用Release版的函数库，不能使用Debug版本的函数库。

      
   最好将Debug版和Release版，分开存放在不同的文件夹内。

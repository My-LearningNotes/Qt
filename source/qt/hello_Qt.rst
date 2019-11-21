Hello, Qt
=========

Qt中一些比较重要的概念包括: 窗口部件，布局，信号槽等。

一个最简单的Qt程序如下：

.. code:: c++

   #include <QApplication>
   #include <QLabel>

   int main(int argc, char *argv[])
   {
       QApplication app(argc, argv);
       QLabel *label = new Label("<h2><font color=red>hello</font>, world!</h2>");
       label->show();
       
       return app.exec();
   }

-  对于每一个Qt类，都有一个与该类同名(且大写)的头文件，在这个头文件中包括了对该类的定义；

-  ``QApplication``\ 对象用来管理整个应用程序所用到的资源，\ ``QApplication``\ 的构造函数需要两个参数，分别是\ ``argc``\ 和\ ``argv``\ ，因为Qt支持它自己的一些命令行参数；

-  在Qt和Unix的术语中，\ **窗口部件**\ 就是用户界面中的一个可视化元素。

   窗口部件在创建时，通常都是隐藏的，这就允许我们先对其进行设置然后再显示它们，从而避免了窗口部件的闪烁现象；

-  执行\ ``app.exec()``\ ，将应用程序的控制权传递给Qt。

   此时，程序会进入事件循环状态，这是一种等待模式，程序会等待用户的操作，例如鼠标单击和按键等操作。

   用户的动作会让可以产生响应的程序生成一些事件（event，也称为"消息")，这里的响应通常就是执行一个或多个函数。

-  Qt窗口部件支持html样式格式。

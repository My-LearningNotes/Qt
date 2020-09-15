Hello, Qt
=========

一个最简单的Qt程序如下:

.. code-block:: cpp

    #include <QApplication>
    #include <QLabel>

    int main(int argc, char *argv[])
    {
        QApplication app(argc, argv);
        QLabel *label = new Label("<h2><font color=red>hello</font>, world!</h2>");
        label->show();
       
        return app.exec();
    }

- 对于每一个Qt类, 都有一个与该类同名(且大写)的头文件, 在这个头文件中包括了对该类的定义;

- ``QApplication``\ 对象用来管理整个应用程序所用到的资源, \ ``QApplication``\ 的构造函数需要两个参数, 分别是\ ``argc``\ 和\ ``argv``\ , 因为Qt支持它自己的一些命令行参数;

- 在Qt和Unix的术语中, \ **窗口部件**\ 就是用户界面中的一个可视化元素;

    * 窗口部件中可以包含其他的窗口部件, 例如, 应用程序的窗口通常就是一个包含了一个\ ``QMenuBar``, 一些\ ``QToolBar``, 一个\ ``QStatusBar``\ 以及一些其他窗口部件的窗口部件;

    * 绝大多数应用程序都会使用一个\ ``QMainWindow``\ 或者一个\ ``QDialog``\ 来作为它的窗口,  但Qt是如此灵活, 以至于任意窗口部件都可以用作窗口;

    * 窗口部件在创建时, 通常都是隐藏的, 这就允许我们先对其进行设置然后再显示它们, 从而避免了窗口部件的闪烁现象;

- 执行\ ``app.exec()``\ , 将应用程序的控制权传递给Qt;

此时, 程序会进入事件循环状态, 这是一种等待模式, 程序会等待用户的操作, 例如鼠标单击和按键等操作.

- Qt窗口部件支持html样式格式.


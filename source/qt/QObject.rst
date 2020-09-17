``QObject``\ 简介
=================

QObject is the heart to the Qt Object Model. 
The central feature in this model is a very powerful mechanism for seamless object communication called signals and slots.

QObject是Qt对象模型的核心. 
Qt对象模型最重要的特点是提供了信号槽机制, 用于对象之间的通信. 

QObjects organize themselves in object trees. 
When you create a QObject with another object as parent, object will automatically add itself to the parent's ``children()`` list. 

Every object has an ``objectName()`` and its class name.

When an object is deleted,  it emits a ``destroyed()`` signal.

QObject can receive events through ``event()`` and filter the events of other objects.

Notice that the ``Q_OBJECT`` macro is mandatory for any object that implements signals, slots or properties. 
You also need to run the **Meta Object Compiler** on the source file. 
We strongly recommended the use of this macro in all subclasses of QObject regardless of whether or not they actually use signals, slots and properties, 
since failure to do so may lead certain functions to exhibit strange behavior.

All QWidgets inherit QObject. 


Thread Affinity
---------------

A QObject instance is said to have a *thread affinity*\ , or that it *lives* in a certain thread. 
When a QObject receives a queued signal or a posted event, the slot or event handler will run in the thread that the object lives in.

.. note::

    If a QObject has no thread affinity(that is, if ``thread()`` returns zero), or if it lives in a thread has no running event loop, then it cannot receive queued signals or posted events.

By default, a QObject lives in the thread in which it is created. 
An object's thread affinity can be queried using ``thread()`` and changed using ``moveToThread()``\ . 

All QObjects must live in the same thread as their parent. Consequently:

* ``setParent()`` will fail if the two QObjects involved live in different threads.
* When a QObject is moved to another thread, all its children will be automatically moved too.
* ``moveToThread()`` will fail if the QObject has a parent.
* If QObjects are created within ``QThread::run()``\ , they cannot become children of the QThread object because the QThread does not live in the thread that calls ``QThread::run()``\ .

.. note::

    A QObject's memeber variables **does not** automatically become its children. 
    The parent-child relationship must be set by either passisng a pointer to the child's constructor, or by calling ``setParent()``\ .
    Without this step, the object's memeber will remain in the old thread when ``moveToThread()`` is called.


## No Copy Constructor or Assignment Operator

QObject has neither a copy constructor nor an assignment operator. This is by design. 
Actually, they are declared, but in a private section with the macro ``Q_DISABLE_COPY()``\ .

In fact, all Qt classes derived from QObject(direct or indirect) use this macro to declare their copy constructor and assignement operator to be private. 

The main consequence is that you should use pointers to QObject (or to your QObject subclass) where you might otherwise be tempted to use your QObject subclass as a value. 
For example, without a copu constructor, you can't use a subclass of QObject as the value to be stored in one of the container classes. You must store pointers.

.. note::

    因为QObject(以及子类)没有复制构造函数和赋值运算符, 所以要注意, 在某些情况下只能通过指针来使用它们.


Auto-Connection
---------------

Qt's meta-object system provides a mechanism to automatically connect signals and slots between QObject subclasses and their children. 
As long as objects are defined with suitable object names, and slot follow a simple naming convention, this connection can be performed at run-time by the ``QMetaObject::connectSlotsByName()`` function.

``uic`` generates code that invokes this function to enable auto-connection to be performed between widgets on forms created with Qt Designer. 

.. note::

    按照一定的约定命名时, 自动建立信号槽的连接.


Dynamic Properties
------------------

From Qt 4.2, dynamic properties can be added to and removed from QObject instances at run-time. 
Dynamic properties do not need to be declared at compile time, yet they provide the same advantages as static properties and are manipulated using the same API - using ``property()`` to read 
and ``setProperty()`` to write them.

From Qt 4.3, dynamic properties are supported by Qt Designer, and both standard Qt widgets and user-created forms can be given dynamic properties.

.. note::

    静态属性, 在编译时就已经定义好的属性.

    动态属性, 可以在运行时动态增删的属性.


Internationalization
--------------------

All QObject subclasses support Qt's translation features, making it possible to translate an application's user interface into different languages.

To make user-visible text translatable, it must be wrapped in calls to the the ``tr()`` function.

.. note::

    国际化, 即可以将界面翻译成多种语言.


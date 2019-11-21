Qt信号槽
========

信号槽机制是Qt的关键部分之一。

通过信号槽，能够使Qt各组件在不知道对方的情形下相互通信，这就将类之间的关系做了最大程度的解耦。

.. note::

   我们在定义一个类时，不需要关心和它通信的其它类是什么样子，只需要关心当前类会发出哪些信号，有哪些槽函数。 
   当对象之间需要通信时，连接相应的信号和槽即可。

因为Qt的信号槽机制是基于元对象系统的，所以使用信号槽的类要继承\ ``QObject``\ 类，并在类声明的开头添加\ ``Q_OBJECT``\ 宏。


信号
----

-  信号的定义放在关键字\ ``signals``\ 之后，并且\ ``signals``\ 后接一个\ ``:``;

-  信号就是一个函数，信号的定义和普通的C++成员函数的定义一样，但是\ **信号只要声明函数原型就可以了，不需要实现**;

-  信号函数的返回值类型通常是没有意义的(因为不会带回返回值)，所以通常定义为\ ``void``;

-  信号函数可能没有参数，表示信号触发槽函数时，没有参数传递；

-  信号函数也可能带有参数，表示信号触发槽函数时，传递相应类型，个数和顺序的参数给槽函数；

-  可以通过\ ``emit()``\ 函数发射信号。

   ``emit()``\ 函数的使用形式为: ``emit(信号名(参数...))``\ ，发射信号时要传递和信号定义时相同数目的参数。


槽函数
------

槽函数和普通的C++成员函数没有很大的区别，可以是\ ``virtual``\ 的，可以被重写，可以是\ ``public``, ``protected``\ 或\ ``private``\ 的；可以由其它的C++函数调用；参数可以是任何类型的。 
如果说区别，那就是槽函数可以和一个信号相连接，当这个信号发生时，它可以被自动调用。

-  槽函数的定义放在关键字\ ``slots``\ 之后，并且\ ``slots``\ 之后有一个\ ``:``;

-  关键字\ ``slots``\ 可以被\ ``public``, ``protected``\ 或\ ``private``\ 修饰，其意义和修饰普通的类成员函数的类似：

   -  ``public slots``\ 表示槽函数可以连接外部的信号

   -  ``protected slots``\ 表示槽函数不可以连接外部的信号，但是可以连接其派生类中的信号

   -  ``private slots`` 表示槽函数只可以连接类内部的信号

-  槽函数的返回值类型通常也没有什么意义(因为没法获取槽函数的返回值)，所以通常定义为\ ``void``.

-  槽函数可能没有参数，表示信号触发槽函数时，不接收参数；

-  槽函数也可能有参数，表示信号触发槽函数时，接收相应类型，个数和顺序的参数；

-  可以像调用普通的类成员函数一样，直接调用槽函数。

.. note::

   在一个类中，可以定义多个\ ``signals``\ 和\ ``slots``\ 部分。

Example:

.. code-block:: c++
   :emphasize-lines: 1, 3, 9, 12

   class Employee : public QObject  // 继承QObject
   {
   	Q_OBJECT  // 类声明中包含Q_OBJECT宏

   public:
   	Employee();
   	~Employee();

   public slots:  // 定义槽函数
   	void SetSalary(int newSalary); 	

   signals:  // 定义信号
   	void SalaryChanged(int newSalary);

   private:
   	int mySalary;
   }


连接信号槽
----------

``connect()``\ 函数是\ ``QObject``\ 类中的一个静态函数，其作用是连接信号槽。

``connect()``\ 函数的原型类似于:

   .. code-block:: c++

      connect(sender, SIGNAL(signal), receiver, SLOT(slot));

-  ``sender``\ 和\ ``receiver``\ 都是\ ``QObject``\ 类型的指针；

-  ``signal``\ 和\ ``slot``\ 是\ **只有参数类型没有参数名称**\ 的函数签名；

-  ``SIGNAL()``\ 和\ ``SLOT()``\ 是两个宏，用来把参数转换成字符串。

-  如果发送者和接收者是同一个对象，可以省略接收者。

   需要注意，在这种旧式的信号槽连接语法中，省略\ ``receiver``\ 的语法，只适用于在类中连接信号槽时，如果是在类的外部连接的信号槽，即使发送者和接收者是同一个对象，也不能省略接收者。

   Example:

      .. code-block:: c++
         :emphasize-lines: 8, 9, 10, 22, 23, 24, 25

         class MyObject : public QObject
         {
            Q_OBJECT

         public:
            MyObject()
            {
               // 在类的内部连接信号槽，发送者和接收者是同一个对象
               // 接收者可以省略
               connect(this, SIGNAL(MySignal()), SLOT(MySlot()));
            }

         public slots:
            void MySlot();

         signals:
            void MySignal();
         };


         MyObject myObject;
         // Error
         // 在类的外部连接信号槽
         // 即使发送者和接收者是同一个对象，也不能省略receiver
         QObject::connect(&myObject, SIGNAL(MySignal()), SLOT(MySlot()));

   Qt 5中新式的信号槽连接语法，不能是在类中还是在类外连接信号槽，只要发送者和接收者是同一个对象，都可以省略接收者。


.. note::

   **string-based SIGNAL and SLOT syntax:**

      ``SIGNAL``\ 和\ ``SLOT``\ 是两个宏，作用是将参数转换为字符串(``const char *``\ 类型)，通过字符串标识信号和槽函数。
      正因为这种机制，所以\ ``SIGNAL``\ 和\ ``SLOT``\ 后的参数写法有严格的要求： 不能包括参数名，参数类型不能是使用\ ``typedef``\ 定义的，不能包含命名空间，
      否则会导致无法根据生成的字符串对应信号和槽函数，从而导致信号槽连接失败。


* **为了正确的连接信号槽，信号和槽的参数个数，类型和顺序都必须相同**

Example:

   .. code-block:: c++

      connect(ftp, SIGNAL(rawCommandReply(int, const QString &)), 
              this, SLOT(processReply(int, const QString &)));

* **这里有一种例外情况，如果信号的参数多于槽的参数是允许，多余的参数都会被忽略掉**

Example:

   .. code-block:: c++

      connect(ftp, SIGNAL(rawCommandReply(int, const QString &)),
            this, SLOT(checkErrorCode(int)));

这里，\ ``const QString &``\ 这个参数就会被槽忽略掉。


* **对于槽函数，可以为其参数定义默认值，从而将槽函数连接到原本参数不匹配的信号上**

   -  定义默认值的参数，要定义在没有默认值的参数的后面

   -  在连接信号槽时，槽函数的函数签名的参数列表，要和信号函数一致(定义默认的参数不要写)

   Example:

      .. code-block:: c++

         class MyObject : public QObject
         {
            Q_OBJECT

         public slots:
            void mySlot(int a, int b = 10);

         signals:
            void mySignal(int);
         }


         int main()
         {
            MyObject myObject;
            QObject::connect(&myObject, SIGNAL(mySignal(int)), &myObject, SLOT(mySlot(int)));

            return 0;
         }


.. note::

   注意，如果信号槽的参数不相容(参数的类型，个数或顺序不同)，或者是信号或槽有一个不存在，或者在信号槽的连接中出现了参数名称，在编译时并不会报错，只有到了运行时才能发现信号槽不能正常工作。


**其它的一些关于信号槽的用法：**

-  **一个信号可以和多个槽相连**

   但是需要注意，如果是这种情况，当信号发出时，这些槽一个接一个的都会被调用，但是它们的调用顺序是不确定的，每次都可能会不同。

-  **多个信号可以连接到一个槽**

   只要任意一个信号发出，这个槽就会被调用。

-  **一个信号可以连接到另一个信号**

   .. code-block:: c++

      connect(sender1, SIGNAL(signal1), sender2, SIGNAL(signal2));

   当第一个信号发出时，会触发第二个信号发出。除此之外，这种信号-信号的形式和信号-槽的形式没有什么区别。


Connection Type
***************

连接信号槽时，\ ``connection()``\ 函数的最后一个参数用来指定信号槽的连接类型，可以指定的连接类型有:

   * ``Qt::AutoConenction``\ (Default)
   * ``Qt::DirectConnection``

      The slot is invoked immediately when the signal is emitted. The slot is executed in the signalling thread.
      槽函数在信号发出后立即执行，并且和发射信号在同一个线程内执行。
      相当于一个函数调用，调用了槽函数。

   * ``Qt::QueuedConnection``

      The slot is invoked when control returns to the event loop of the receiver's thread. The slot is executed in the receiver's thread.
      槽函数在接收者的线程内执行。
      槽函数并不是立即执行，而是讲槽函数的执行添加到接收者所在线程的事件循环，当控制回到接收者所在线程的事件循环时执行槽函数。

   * ``Qt::BlockingQuequedConnetion``
   * ``Qt::UniqueConnection``


断开连接
--------

当信号和槽没有必要继续保持关联时，可以使用\ ``disconnect()``\ 函数来断开连接，和\ ``connect()``\ 一样，\ ``disconnect()``\ 也是\ ``QObject``\ 类的静态函数。

.. note::

   对于\ ``QObject``\ 对象，当一个对象\ ``delete``\ 之后，Qt会自动取消所有连接到这个对象上面的槽，但有些时候，我们还是需要主动断开连接。

``disconnect()``\ 函数的原型和\ ``connect()``\ 函数类似:

   .. code-block:: c++

      disconnect(sender, SIGNAL(signal), receiver, SLOT(slot));

* ``disconnect()``\ 函数可以用来断开指定的信号槽之间的连接；

* 在\ ``disconnect()``\ 函数中，还可以用\ ``0``\ 作为一个通配符，可以表示任何信号，任何接收对象，接收对象中的任何槽函数。

   -  断开与某个对象相关联的任何对象

      .. code-block:: c++

         disconnect(sender, 0, 0, 0);

   -  断开与某个特定信号的任何关联

      .. code-block:: c++

         disconnect(sender, SIGNAL(signal), 0, 0);

   -  断开连个对象之间的关联

      .. code-block:: c++

         disconenct(sender, 0, receiver, 0);


New Signal Slot Syntax
----------------------

`New Signal Slot Syntax <https://wiki.qt.io/New_Signal_Slot_Syntax>`_

`Differences between String-Based and Functor-Based Conenctions <https://doc.qt.io/qt-5/signalsandslots-syntaxes.html>`_

旧式的信号槽连接方法称为: **String-Based Connection**, 新式的信号槽连接方法称为: **Functor-Based Connection**.


Qt 5支持新的信号槽连接语法:

   .. code-block:: c++

      connect(sender, &Sender::valueChanged,
              receiver, &Receiver::updateValue);	

-  ``sender``\ 和\ ``receiver``\ 是\ ``QObject``\ 类型的指针；

-  信号和槽，通过类成员函数指针的形式传入。


**Pros:**

   -  Compile time check of the existence of the signals and slot, of the the types, of if the ``Q_OBJECT`` is missing.

      对于旧式的信号槽连接语法，如果信号和槽有一个不存在，或者参数不匹配，或者类定义时忘记了\ ``Q_OBJECT``\ 宏，信号槽连接会失败，但是在编译时并不会报错，只有到了运行时才能发现信号槽不能正常工作。

      新的信号槽连接语法，在编译时就会对上述的问题进行键查，在编译时就能及时发现这些错误。

   -  Argument can be by typedefs or with different namespace specifier, and it works.

      参数可以是用\ ``typedef``\ 定义的，也可以是定义在不同namespace中的。

      旧式的信号槽语法是\ ``string-based syntax``\ ，不支持这些。

   -  Possibility to automatically cast the types if there is implicit conversions(e.g. from ``QString`` to ``QVariant``)

      如果信号和槽的参数类型不匹配，但是支持隐式类型转换，可以自动进行隐式类型转换。

      在旧式的语法中，信号和槽的参数类型必须完全一致，不支持隐式类型转换，必须显式转换。

   -  It is possible to connect to any member function of ``QObject``, not only slots.

      信号可以连接到\ ``QObject``\ 对象的任何成员函数，并非仅仅是槽函数。

**Cons:**

   -  Complicated syntax in cases of overloads?

      新的信号槽连接语法，只根据名称传入了信号和槽的地址，没有传入和类型相同的部分，这样如果有重载的函数，语法会有点复杂。

      从Qt 5.7开始，引入了\ ``QOverload``\ (can be found in ``qglobal.h``)用来解决信号和槽函数重载的问题。

      -  ``QOverload`` requires the use of C++11

      -  or ``qOverload``, using C++14

      可以使用\ ``QOverload``\ 返回一个指向重载函数的\ *function pointer*\ 作为\ ``connect()``\ 的参数，这样就可以解决重载的问题了。

      ``QOverload``\ 的使用方法: ``QOverload<...>::of(...)``

      -  ``QOverload``\ 后面的尖括号中是要选择的那个重载函数的参数列表(只写参数类型，不要参数名称)；

      -  ``of``\ 后的参数是函数指针

      Example:

         .. code-block:: c++

            QOverload<>::of(&Foo::overloadedFunction);

            QOverload<int, const QString &>::of(&Foo::overloadedFunction);

   -  Default arguments in slot is not supported anymore.

      旧式的信号槽连接语法，可以为槽函数的参数定义默认值，从而将槽函数连接到原本参数不匹配的信号上；

      但是新式的信号槽连接语法中不支持。


**在新的语法中，信号除了可以连接到QObject的成员函数外，还可以连接到普通的函数:**

   .. code-block:: c++

      connect(sender, &Sender::valueChanged, SomeFunction)

**Pros:**

   -  Can be used with ``std:bind``:

      因为信号可以连接到普通的函数，所以可以使用\ ``std::bind``\ 将其它的函数绑定为普通函数，然后和信号连接。

      .. code-block:: c++

         connect(sender, &Sender::valueChanged, 
      	        std::bind(&Receiver::updateValue, receiver, "senderValue", std::placeholders:_1)); 

   -  Can be used with C++11 lambda expressions:

      槽函数可以是匿名函数。

      .. code-block:: c++

         connect(sender, &Sender::valueChanged, 
      	        [=](const QString &newValue) {receiver -> updateValue("senderValue", newValue)};);




Disconnecting in Qt 5
~~~~~~~~~~~~~~~~~~~~~

-  Old way

   如果使用旧式的语法连接的信号槽，那么使用旧式的语法断开连接。

   或者可以使用旧式语法中\ ``disconnect()``\ 函数的通配符功能，断开与某个信号连接的所有槽。

-  如果使用\ *function pointer*\ 连接的信号槽，就用\ *function pointer*\ 的方式断开连接。

   .. code-block:: c++

      disconnect(sender, &Sender::valueChanged, 
      	        receiver, &Receiver::updateValue);

   -  可以使用\ ``0``\ 作为通配符来断开连接，这时对使用旧式语法建立的信号槽也是有效的；

   -  如果没有使用\ ``0``\ 作为通配符，只有当信号槽是通过\ *function pointer*\ 建立的连接时，这种断开连接的语法才有效；

   -  In particular, does not work with **static function**, **functor** or **lambda functions**.

-  New way using ``QMetaObject::Connection``\ :

   .. code-block:: c++

      QMetaObject::Connection m_connection;
      // ...
      m_connection = QObject::connect(/* ... */);
      // ...
      QObject::disconnect(m_connection);

   **Works in all cases, including lambda functions or functors.**


常见问题总结
~~~~~~~~~~~~

-  Missing ``Q_OBJECT`` in class definition

-  Type mismatch

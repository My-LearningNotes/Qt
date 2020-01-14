QThread使用方式
===============

在Qt中，通过\ ``QThread``\ 实现多线程，有两种方式:

方式一
------

子类化\ ``QThread``\ ，重写\ ``QThread::run()``\ 方法，在其中实现要执行的代码，用\ ``QThread::start()``\ 方法启动子线程； 
这样，\ ``run()``\ 中的代码在子线程中运行。

Example:

.. code:: c++

    class WorkerThread : public QThread
    {
        Q_OBJECT
               
       	void run() Q_DECL_OVERRIDE
        {
            QString result;

            /* ... here is the expensive or blocking operation ... */
            emit resultReady(result);
        }
           
        signals:
            void resultReady(const QString &s);
    };

    void MyObject::startWorkInAThread()
    {
        WorkerThread *workerThread = new WorkerThread(this);
        connect(workerThread, &WorkerThread::resultReady, 
                this, &MyObject::handleResults);
        connect(workerThread, &WorkerThread::finished, 
                workerThread, &QObject::deleteLater);
        workerThread->start();
    }

-  By default, ``run()`` starts the event loop by calling ``exec()`` and runs a Qt event loop inside the thread.

    ``QThread::run()``\ 的默认实现是调用\ ``exec()``\ ，在线程中启动一个事件循环。

    在这种方式中重写了\ ``run()``\ 方法，the thread will exit after
    the run function has returned. There will not be any event loop running in the thread unless you call ``exec()``\ .

-  如果希望\ ``run()``\ 中的代码运行完之后线程不退出，而是进入事件循环，可以在\ ``run()``\ 中调用\ ``exec()``

    Example:

    .. code:: c++

        void run()
        {
            /*** something to do ***/
            exec();  // 进入事件循环
        }

  如果\ ``run()``\ 中实现的是一个死循环，但是希望线程能够对事件进行处理

    Example:

    .. code:: c++

        void run()
        {
            QEventLoop eventLoop;
            while (true)
            {
                /*** something to do ***/
                  
                eventLoop.processEvent();
                /* sleep */
            }
        }

-  It is important to remember that a ``QThread`` instance lives in the old thread that instantiated it, not in the new thread that calls ``run()``. 
   This means that all of the QThread's queues slots will execute in the old thread. Thus, a developer who wishes to invoke slots in the new thread must use the *worker-object* approach; 
   new slots should not be implemented directly into a subclass QThread.


方式二
------

子类化\ ``QObject``\ ，在其中定义信号，槽和其他要实现的代码; 
定义一个\ ``QThread``\ 的实例，使用\ ``QObject::movetoThread()``\ 方法将定义的\ ``QObject``\ 实例move到子线程中，用\ ``QThread::start()``\ 方法启动子线程

Example:

.. code:: c++

    class Worker : public QObject
    {
        Q_OBJECT
               
    public slots:
        void doWork(const QString &paramter)
        {
            QString result;
            
            /* ... here is the expensive or blocking operation ... */
            emit resultReady(result);
        }
           
    signals:
        void resultReady(const QString &result);
    };

    class Controller : public QObject
    {
        Q_OBJECT
               
        QThread workerThread;
           
    public:
        Controller()
        {
            Worker *worker = new Worker;
            worker->moveToThread(&workerThread);
            connect(&workerThread, &QThread::finished, worker, &QObject::deleteLater);
            connect(this, &Controller::operate, worker, &Worker::doWork);
            connect(worker, &Worker::resultReady, this, &Controller::handleResults);
            workThread->start();
        }
           
        ~Controller()
        {
            workerThread.quit();
            workerThread.wait();
        }
           
    public slots:
        void handleResults(const QString &);
           
    signals:
        void operate(const QString &);
    };

-  在这种方式中，没有重写\ ``run()``\ 方法，它使用的是默认实现: *starts the event loop by calling exec() and runs a Qt eventloop inside the thread*.

-  通过调用\ ``QObject::moveToThread()``\ 将QObject对象移动到指定线程中执行

-  通过信号槽触发QObject槽函数的执行

    The code inside the *Worker's* slot would then execute in a separate thread.

    However, you are free to connect the *Worker's* slots to any signal, from any object, in any thread. 
    It is safe to connect to signals and slots across different threads, thanks to a mechanism called *queued connections*.

-  通过信号触发槽函数在新线程中执行完之后，新线程并不会退出，而是进入事件循环。


总结
----

一般推荐使用方式二，因为:

-  根据\ ``QThread``\ 的设计初衷，\ ``QThread``\ 是提供一个控制子线程的接口，用来管理子线程；

   而要在子线程中实际运行的代码不应该包含在\ ``QThread``\ 中，这么做违背了\ ``QThread``\ 的设计初衷，也违背了OO(面向对象)的设计原则；

   而且，按照方法1，将单一的实例绑定到一个线程是没有必要的；

   按照正确的设计原则，唯一需要从\ ``QThread``\ 继承的情况应该是我们需要扩展\ ``QThread``\ 的功能，而不是在其中实现要运行的代码，\ **不应该在\ ``QThread``\ 中包含要在子线程中运行的代码**\ 。

-  ``QThread``\ 的正确使用方式应该是方式2

   按照方式2来使用\ ``QThread``\ ，\ **实现了线程控制和线程中运行的代码之间的分离**\ ，这样更加灵活；

   例如，我们可以根据需要将同一个类的不同实例绑定要单一子线程中，或将不同类的实例绑定到单一线程中。

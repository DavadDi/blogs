.. contents:: 本章目录
   :depth: 3


Notification 通知时间派发处理
=================================

通知的作用
---------------------------------
* 如果反应器 Reactor 的所有者线程的时间多路分离器函数已阻塞并等待IO时间的发生，反应器的通知机制赋予了其他线程将所有者唤醒的功能。
   因为调用 ``notify`` 函数会触发notify_handler的读句柄的事件发生。


* 可以将一个时间处理器的指针和ACE_Reactor_Mask值中的一个传给 ``notify`` 方法，这些参数会触发 Reactor 分派对应的处理器的挂钩函数，
   而无需将处理器与IO或者定时器处理事件关联起来。  就是不需要对处理器现行在 Reactor 上进行登记，搞了个特殊化，插队机制。

ACE_Select_Reactor_Notify 类继承图
------------------------------------

``ACE_Select_Reactor_T`` 类中定义的 ``notify_handler``, 见 :ref:`mem_def`,  默认类型为 *ACE_Select_Reactor_Notify*。

.. image:: _static/a03597.png
   :scale: 100%
   :align: center
   :alt: ACE_Select_Reactor_Notify 

.. note:: 

	在 ``ACE_Select_Reactor_T`` 的 ``open`` 函数的参数中一个参数用于控制 notify 是否采用 pipe 的参数 *disable_notify_pipe*，
        默认值是  *ACE_DISABLE_NOTIFY_PIPE_DEFAULT*，宏定义是根据不同平台特性决定该值为0还是1 （需要进一步确认TODO）。

.. _notify_reactor:

通知与Select_Reactor 的分发集成
---------------------------------

.. rubric:: ace/Select_Reactor_T.cpp dispatch_notification_handlers 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 1160-1191
    :emphasize-lines: 13-15

dispatch_notifications 函数
+++++++++++++++++++++++++++++

.. rubric:: ace/Select_Reactor_Base.cpp dispatch_notifications 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_Base.cpp
    :linenos:
    :lines: 731-749
    :emphasize-lines: 15
..


handle_input 函数
+++++++++++++++++++++++++++++

.. rubric:: ace/Select_Reactor_Base.cpp handle_input 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_Base.cpp
    :linenos:
    :lines: 921-961
    :emphasize-lines: 15, 19,40-43
..


dispatch_notify 函数
+++++++++++++++++++++++++++++

.. rubric:: ace/Select_Reactor_Base.cpp dispatch_notify 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_Base.cpp
    :linenos:
    :lines: 785-863
    :emphasize-lines: 14-16, 37,51

open函数 
+++++++++++++++++++++++++++++

.. rubric:: ace/Select_Reactor_Base.cpp open 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_Base.cpp
    :linenos:
    :lines: 594-642
    :emphasize-lines: 8,18,38-41
..

其中 ``notification_pipe_`` 类型为 ``ACE_Pipe``,  38-41行 则注册了 pipe 的 handler 的 READ_MASK 至 Reactor 中，及时接受相关通知消息。


通知的调用接口
---------------------------

notify 函数 
+++++++++++++++++++++++++++++

用于进行通知的调用方法。

.. rubric:: ace/Select_Reactor_Base.cpp notify 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_Base.cpp
    :linenos:
    :lines: 674-726
    :emphasize-lines: 20,40-43
..


通知的注意事项
---------------------------

**避免死锁**

#. 缺省机制下，反应器的通知机制是通过有界缓存来实现的，而 notify() 使用柱塞式发送调用将通知插入队列中。因此如果缓存区已经满，某个时间处理器的 handle_*() 函数又调用了 notify() ，就可能发生死锁。 避免方式可采用如下：

  * 使用 notify() 函数时候指定超时时间

  * 对应用进行设计，保证生成 notify() 调用的速度不会快于处理器处理的速度，这是最好的保护方式。

#. 扩大 ACE_Select_Reactor 的通知机制

  * 将通知机制的有界缓存替换成任意扩大的用户队列，通过 $ACE_ROOT/ace/config.h 文件中增加 ``#define ACE_HAS_REACTOR_NOTIFICATION_QUEUE`` 宏定义，然后重新编译，缺省情况下该特性没有被启用，因为高性能系统或者嵌入式系统中难以接受。

  * 使用该用户空间可扩大队列，允许使用 ``ACE_Reactor::purge_pending_notifications()`` 方法扫描队列，并移除符合要求的时间处理器，避免已经销毁的时间处理器仍然挂在 notify 的队列中。
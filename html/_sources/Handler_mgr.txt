.. contents:: 本章目录
   :depth: 3

.. highlight:: c++

I/O Handler的管理
------------------


.. _handler_reactor:

IO句柄与Select_Reactor的分发集成
+++++++++++++++++++++++++++++++++++++++++++++


dispatch_io_handlers 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. rubric:: ace/Select_Reactor_T.cpp dispatch_io_handlers 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 1234-1283
    :emphasize-lines: 13-18, 25, 37

dispatch_io_set 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. rubric:: ace/Select_Reactor_T.cpp dispatch_io_set 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 1193-1232
    :emphasize-lines: 19-24
..


notify_handle 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_T.cpp notify_handle 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 805-838
    :emphasize-lines: 24-29
..

如果回调函数 ``status < 0``, 则调用移除该 handle 的对应 mask：``this->remove_handler_i (handle, mask)`` 。

如果回调函数 ``status > 0``， 这表明该 handle 需要继续被调度处理，则需要将该 handle 设置到 ``ready_set_`` 中再次被派发。

handler注册
+++++++++++++++

register_handler 函数流程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



.. rubric:: ace/Select_Reactor_T.cpp register_handler 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 279-290
    :emphasize-lines: 9
..

register_handler_i 函数流程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_T.cpp register_handler 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 997-1010
    :emphasize-lines: 11
..

.. _event_handlers_def:

bind 函数
~~~~~~~~~~~~~

.. rubric:: 映射数据结构定义

行26实现了真正的关联操作，``this->event_handlers_.bind (handle, event_handler, entry);`` 其中变量 ``event_handlers_`` 
在程序中定义为*map_type*, 在Unix下具体定义如下 ``typedef ACE_Array_Base<value_type> map_type;``，也即使用handle
句柄值作为数组的索引，实现了map的功能。在windows下使用了ACE_Hash_Map_Manager_Ex作为map实现的类， 
``typedef ACE_Hash_Map_Manager_Ex<key_type,value_type, ACE_Hash<key_type>,std::equal_to<key_type>,ACE_Null_Mutex> 
map_type;``,其中 key_type的定义为：``typedef ACE_HANDLE key_type;``，另外value_type定义为 ``typedef ACE_Event_Handler * value_type;``。

.. rubric:: ace/Select_Reactor_Base.cpp bind 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_Base.cpp
    :linenos:
    :lines: 186-282
    :emphasize-lines: 26,50-51,71,75,82
..
 
handler移除
+++++++++++++++

remove_handler 函数流程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_T.cpp remove_handler 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 342-350
    :emphasize-lines: 8

remove_handler_i 函数流程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_T.cpp remove_handler_i 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 1011-1020
    :emphasize-lines: 9

unbind 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_Base.cpp unbind 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_Base.cpp
    :linenos:
    :lines: 284-402
    :emphasize-lines: 20,26,107-108
..

.. WARNING::

	``remove_handler`` 函数在后续调用 ``unbind`` 函数中，会触发对 ``handle_close`` 行107-108 的调用，需要注意，否则在 ``handle_close`` 函数中 使用 ``remove_handler`` 如果没有设置 ``DONT_CALL`` 标志，会导致 ``handle_close`` 的循环调用。
	

handler暂停
+++++++++++++++

suspend_handler 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_T.cpp suspend_handler 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 226-232
    :emphasize-lines: 6

suspend_i 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. rubric:: ace/Select_Reactor_T.cpp suspend_i 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 951-979
    :emphasize-lines: 10-11,15-16,20-21,27

handler恢复
+++++++++++++++

resume_handler 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_T.cpp resume_handler 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 218-224
    :emphasize-lines: 6

resume_handler_i 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. rubric:: ace/Select_Reactor_T.cpp resume_handler_i 函数
.. literalinclude:: F:\ACE-6.2.4\ACE_wrappers\ace\Select_Reactor_T.cpp
    :linenos:
    :lines: 924-947
    :emphasize-lines: 10-11,15-16,20-21 

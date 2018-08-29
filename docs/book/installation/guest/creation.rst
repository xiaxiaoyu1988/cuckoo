===============================
创建虚拟机
===============================

在 :doc:`安装好<../host/requirements>` 虚拟机软件后,
就可以开始创建虚拟机了.

虚拟机软件的使用和配置不在本文的范围内，可以参考您选择的虚拟机软件官网文档.

.. note::

    You can find some hints and considerations on how to design and create
    your virtualized environment in the :doc:`../../introduction/sandboxing`
    chapter.

.. note::

    我们推荐64位的Win7或者WinXP虚拟机， 如果是Win7系统，需要关闭UAC。

    .. versionchanged:: 2.0-rc2
       We used to suggest Windows XP as a guest VM but nowadays a 64-bit
       Windows 7 machine yields much better results.

.. note::

    KVM 用户 - 要选择一种支持快照的虚拟机镜像格式.
    可以查阅 :doc:`saving` 获取更多信息

创建的虚拟机， Cuckoo并不要求特殊的硬件配置信息， 你可以选择最适合需要的配置。
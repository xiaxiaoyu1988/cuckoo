.. Installation chapter frontpage

安装
============

本章节主要说明如何安装Cuckoo。推荐Linux系统（Debian 或者 Ubuntu 等）。

.. note::
    【译者注】 测试过程中选择了Debian 8, Debian下大部分的软件安装方式与Ubuntu是类似的

Although the recommended setup is *GNU/Linux* (Debian or Ubuntu preferably),
Cuckoo has proved to work smoothly on *Mac OS X* and *Microsoft Windows 7* as
host as well. The recommended and tested setup for guests are *Windows XP* and
*64-bit Windows 7* for Windows analysis, *Mac OS X Yosemite* for Mac OS X
analysis, and Debian for Linux Analysis, although Cuckoo should work with
other releases of guest Operating Systems as well.

.. note::

    This documentation refers to *Host* as the underlying operating systems on
    which you are running Cuckoo (generally being a GNU/Linux distribution) and
    to *Guest* as the Windows virtual machine used to run the isolated analysis.

.. toctree::

    host/index
    guest/index
    guest_physical/index
    upgrade

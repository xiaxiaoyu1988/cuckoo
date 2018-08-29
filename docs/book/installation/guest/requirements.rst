============
依赖
============

为了Windows虚拟机可以与Cuckoo工作正常，需要安装一些必须的软件和库。

安装 Python
==============

Python 是 Cuckoo 客户端（*分析器*） 正常工作的必须软件。

可以直接从官网下载安装，要求 Python2.7 版本。

Cuckoo 客户端组件依赖于部分额外的Python 库， 包括:

    * `Python Pillow`_: 截图组件需要用到.

这些组件不是必须要安装的， 但是不安装的话，分析组件的部分功能就无法正常使用。

.. _`official website`: http://www.python.org/getit/
.. _`Python Pillow`: https://python-pillow.org/

其他软件
===================

至此，Cuckoo 正常工作所需的软件的已经安装完成了。

不过根据你需要分析的文件类型， 也同时需要安装相应的软件， 例如浏览器，PDF阅读器，Office软件等。
记得要关闭这些软件的检查更新和自动更新。

这些额外的软件是否需要安装，完全取决于你是否所需。 
可以阅读 :doc:`../../introduction/sandboxing` 章节了解更多的信息.


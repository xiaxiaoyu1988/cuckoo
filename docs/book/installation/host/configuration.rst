=============
配置
=============

Cuckoo 中有几个核心的配置文件:

* :ref:`cuckoo_conf`: 用于配置通用选项和分析参数.
* :ref:`auxiliary_conf`: 用于开启或者分配辅助模块.
* :ref:`machinery_conf`: 用于配置和填入虚拟机相关参数（使用何种虚拟机，则选择哪种虚拟机配置文件，例如选择kvm， 则配置kvm.conf）.
* :ref:`memory_conf`: Volatility 配置选项.
* :ref:`processing_conf`: 用户开启或者配置数据处理模块.
* :ref:`reporting_conf`: 用于开关报表模块.

Cuckoo正常工作至少需要配置两个文件 :ref:`cuckoo_conf` 和 
:ref:`machinery_conf`.

.. _cuckoo_conf:

cuckoo.conf
===========

文件路径 ``$CWD/conf/cuckoo.conf``. 
注意下下 ``$CWD`` 目录指的Cuckoo工作目录，具体可以参考  :doc:`cwd` .
The ``cuckoo.conf`` 包含了通用的选项，修改前要熟知其含义.

配置文件中已经对相关选项做了详细的注释，如下几个选项我们做一下特别的说明:

*  ``[cuckoo]`` 中的 ``machinery`` :
    该选项指定使用何种虚拟机引擎 (e.g., ``virtualbox`` or ``vmware``).

* ``[resultserver]`` 中的 ``ip`` 和 ``port`` :
    这个IP和端口是Cuckoo的结果服务需要监听的，要确保虚拟机的网络对该IP和端口是可达的，
    否则可能造成没有分析结果.

* ``[database]`` 中的 ``connection`` :
    这个配置用于定义数据库链接URL。可以使用任何 `SQLAlchemy`_
    支持的 `Database Urls`_ 格式.

.. _`SQLAlchemy`: http://www.sqlalchemy.org/
.. _`Database Urls`: http://docs.sqlalchemy.org/en/latest/core/engines.html#database-urls

.. warning:: Check your interface for resultserver IP! Some virtualization software (for example Virtualbox)
    don't bring up the virtual networking interfaces until a virtual machine is started.
    Cuckoo needs to have the interface where you bind the resultserver up before the start, so please
    check your network setup. If you are not sure about how to get the interface up, a good trick is to manually start
    and stop an analysis virtual machine, this will bring virtual networking up.
    If you are using NAT/PAT in your network, you can set up the resultserver IP
    to 0.0.0.0 to listen on all interfaces, then use the specific options `resultserver_ip` and `resultserver_port`
    in *<machinery>.conf* to specify the address and port as every machine sees them. Note that if you set
    resultserver IP to 0.0.0.0 in cuckoo.conf you have to set `resultserver_ip` for all your virtual machines.

.. _auxiliary_conf:

auxiliary.conf
==============

辅助模块在恶意软件运行的同时运行, 该配置文件中可以修改相关选项.

以下是 ``$CWD/conf/auxiliary.conf`` 的文件内容.
.. note::
    【译者注】 文件内容就不翻译了，选项含义都较为明确

.. literalinclude:: ../../_files/conf/auxiliary.conf
    :language: ini

.. _machinery_conf:

<machinery>.conf
================

虚拟机模块定义了Cuckoo与选择的虚拟机引擎之间是如何交互的.

每种虚拟机引擎都有独立的配置文件，例如KVM引擎就是kvm.conf.

Cuckoo 默认使用的是 Virtualbox.

以下即是 ``$CWD/conf/Virtualbox.conf`` 的文件内容.

.. literalinclude:: ../../_files/conf/Virtualbox.conf
    :language: ini

不同虚拟机的配置文件看起来类似, 只是稍有不同. 例如., ``XenServer`` 通过API操作，所以需要填写URL和认证信息.

配置文件中对选项含义也有详细备注.

以下是 ``$CWD/conf/kvm.conf`` 的文件内容.

.. literalinclude:: ../../_files/conf/kvm.conf
    :language: ini

.. _memory_conf:

memory.conf
===========

Volatility 工具提供的内存分析的大量插件， 其中一部分插件运行很慢。
``$CWD/conf/volatility.conf`` 配置文件可以让你配置开关哪些插件。
如果需要运行内存分析，需要打开两个开关:

 * 启用 ``$CWD/conf/processing.conf`` 中的  ``volatility``
 * 启用 ``$CWD/conf/cuckoo.conf`` 中的 ``memory_dump``

``$CWD/conf/memory.conf`` 文件的基础配置一节中， 可以配置是否在内存分析完成后，删除转储文件。
可以节省大量的磁盘空间， 配置内存如下::

    # Basic settings
    [basic]
    # Profile to avoid wasting time identifying it
    guest_profile = WinXPSP2x86
    # Delete memory dump after volatility processing.
    delete_memdump = no

在此之下，每个插件都有相应的配置::

    # Scans for hidden/injected code and dlls
    # http://code.google.com/p/volatility/wiki/CommandReference#malfind
    [malfind]
    enabled = on
    filter = on

    # Lists hooked api in user mode and kernel space
    # Expect it to be very slow when enabled
    # http://code.google.com/p/volatility/wiki/CommandReference#apihooks
    [apihooks]
    enabled = off
    filter = on

每个插件都可以单独是否开启白名单filter.
[mask] 中的 pid_generic 可以配置进程id 白名单， 在白名单中的进程不做内存分析::

    # Masks. Data that should not be logged
    # Just get this information from your plain VM Snapshot (without running malware)
    # This will filter out unwanted information in the logs
    [mask]
    # pid_generic: a list of process ids that already existed on the machine before the malware was started.
    pid_generic = 4, 680, 752, 776, 828, 840, 1000, 1052, 1168, 1364, 1428, 1476, 1808, 452, 580, 652, 248, 1992, 1696, 1260, 1656, 1156

.. _processing_conf:

processing.conf
===============

该配置文件用于开关以及配置结果分析模块.
结果分析模块属于 ``cuckoo.processing`` 模块，主要用于对原始数据进行分析 .

``$CWD/conf/processing.conf`` 中每一个分析模块都有相应的配置section.

.. literalinclude:: ../../_files/conf/processing.conf
    :language: ini

如果你有私有的 `VirusTotal`_ key， 可以将它修改为自己的key.

.. _`VirusTotal`: http://www.virustotal.com

.. _reporting_conf:

reporting.conf
==============

``$CWD/conf/reporting.conf`` 主要用于配置报告生成.

主要包含以下内容.

.. literalinclude:: ../../_files/conf/reporting.conf
    :language: ini

通过将选项值修改为 ``on`` 或者 ``off`` 来开关相应的报告生成

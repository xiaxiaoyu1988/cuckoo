==============================
Cuckoo 工作目录使用说明
==============================

.. note:: 本章阅读前，可以先看下Cuckoo安装和 :doc:`../installation/host/cwd`.

在文章开始之前，我们首先说明下，``CWD`` 的引入到底改进了很多地方，
这么说吧， 能够直接提升我们的生活质量:)

改进点:

* 如 :doc:`../installation/host/installation` 一节中所述
  Cuckoo 现在的安装和升级只需要 执行一条命令 ``pip install -U cuckoo``.
* 由于现在Cuckoo已经录入Python官方版本库，所以我们对版本升级控制的更加严格了，
  升级过程中会尽可能减少对用户已有数据的影响.
* 也因为升级更加方便，我们的版本发布将会更加频繁。比如，BUG修复的速度将会更快。
* **Cuckoo的配置文件不在归档到GIt版本库中**. 用户如果是从旧版本的Cuckoo升级过来的.
  需要手动将配置文件更新到新的配置文件中。
* 新的Cuckoo将所有的配置文件都集中放到 ``CWD`` 目录.
* 新版本的Cuckoo支持一次安装运行多个实例指向不同的 ``CWD`` 目录。
* 新版Cuckoo中，整合了之前的多个脚本，
  以 ``cuckoo`` 脚本命令行参数的方式来运行 :ref:`cuckoo_apps` 。 

Usage
=====

Cuckoo 安装(:ref:`installing`)和配置(:doc:`../installation/host/cwd`)完成后，
就可以开始使用了。
如果安装过程种有问题，可以参考 :ref:`pip_install_issue`
如果使用的是virtualenv安装，可以参考如下命令

.. code-block:: bash

    $ virtualenv venv
    $ . venv/bin/activate
    (venv)$ pip install -U pip setuptools
    (venv)$ pip install -U cuckoo
    (venv)$ cuckoo --cwd ~/.cuckoo

开始使用之前，如果需要修改Cuckoo的默认配置， 配置目录在 ``$CWD/conf/`` 。
如果添加虚拟机或者修改数据库， 可以参考 :doc:`../installation/guest/index`。
如果需要WEB界面上看到样本分析报告， ``$CWD/conf/reporting.conf`` 中的 ``mongodb`` 一定要启用。
参考 :doc:`web`

接下来我们需要下载 Cuckoo Community，
其中包含了300多个恶意软件行为签名，可用于简化我们对结果的分析。
下载命令如下::

    (venv)$ cuckoo community

或者如果有下载好的 community 压缩包， (例如 ``wget https://github.com/cuckoosandbox/community/archive/master.tar.gz``)
可以通过如下命令直接导入::

    (venv)$ cuckoo community --file master.tar.gz

至此，我们就可以开始提交样本了， 可以参考 :ref:`submitpy` 。
多个样本可以在一次命令中提交，例如::

    (venv)$ cuckoo submit /tmp/sample1.exe /tmp/sample2.exe /tmp/sample3.exe
    Success: File "/tmp/sample1.exe" added as task with ID #1
    Success: File "/tmp/sample2.exe" added as task with ID #2
    Success: File "/tmp/sample3.exe" added as task with ID #3
    (venv)$ cuckoo submit --url google.com bing.com
    Success: URL "google.com" added as task with ID #4
    Success: URL "bing.com" added as task with ID #5

样本的分析，依赖 cuckoo 的守护进程。 默认情况下， 直接运行守护进程，
不限制同时分析的样本数量（可通过 ``-m`` 参数指定)

.. code-block:: bash

    # This command is equal to what used to be "./cuckoo.py -d".
    (venv)$ cuckoo -d

如果需要从WEB界面查看界面分析结果， 则需要运行cuckoo WEB进程。
对于测试环境或者并发数较小的环境， 可以通过内置的 Django WEB server 来运行，
实际环境下，我们更推荐基于高性能的WEB服务器来部署， 可以参考 :ref:`web_deployment`


.. code-block:: bash

    (venv)$ cuckoo web
    Performing system checks...

    System check identified no issues (0 silenced).
    March 31, 2017 - 12:10:46
    Django version 1.8.4, using settings 'cuckoo.web.web.settings'
    Starting development server at http://localhost:8000/
    Quit the server with CONTROL-C.

另外，cuckoo 还包含了一些其他的领命， 例如 
``cuckoo clean`` (:ref:`cuckoo-clean`),  :ref:`rooter`
以及 :ref:`cuckoo_apps` 列出的一些实用工具， 除此之外就没别的了。
so, happy analyzing.
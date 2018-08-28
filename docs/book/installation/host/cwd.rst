.. _CWD:

========================
Cuckoo 工作目录
========================

.. versionadded:: 2.0.0

新版本中多了一个 ``Cuckoo 工作目录`` 的概念， 用来存储之前的所有配置文件，
生成的数据以及分析结果。
具体包括但不限于以下几种文件:

* 配置文件
* Cuckoo 签名规则
* Cuckoo 分析器
* Cuckoo 客户端
* Yara 规则集
* Cuckoo 分析数据存储
* 其他文件..

``Cuckoo 工作目录`` 相比较之前的方式，有了更多的优点.

.. note:: This document merely shows the installation part of the ``CWD``, for
    its actual usage, please refer to the :doc:`../../usage/cwd` document.

配置
=============

If you have ever updated your Cuckoo setup to a later version, you have run
into the issue where you had to make a backup of your configuration, update
your Cuckoo instance, and either restore your configuration or re-apply it
completely.

With the introduction of the ``CWD`` we have gotten rid of this update
nightmare.

``Cuckoo`` 首次运行的时候 ``CWD`` 目录会自动创建，输出如下::

    $ cuckoo -d

            _       _                   _             _              _            _
            /\ \     /\_\               /\ \           /\_\           /\ \         /\ \
            /  \ \   / / /         _    /  \ \         / / /  _       /  \ \       /  \ \
            / /\ \ \  \ \ \__      /\_\ / /\ \ \       / / /  /\_\    / /\ \ \     / /\ \ \
        / / /\ \ \  \ \___\    / / // / /\ \ \     / / /__/ / /   / / /\ \ \   / / /\ \ \
        / / /  \ \_\  \__  /   / / // / /  \ \_\   / /\_____/ /   / / /  \ \_\ / / /  \ \_\
        / / /    \/_/  / / /   / / // / /    \/_/  / /\_______/   / / /   / / // / /   / / /
        / / /          / / /   / / // / /          / / /\ \ \     / / /   / / // / /   / / /
    / / /________  / / /___/ / // / /________  / / /  \ \ \   / / /___/ / // / /___/ / /
    / / /_________\/ / /____\/ // / /_________\/ / /    \ \ \ / / /____\/ // / /____\/ /
    \/____________/\/_________/ \/____________/\/_/      \_\_\\/_________/ \/_________/

    Cuckoo Sandbox 2.0.0
    www.cuckoosandbox.org
    Copyright (c) 2010-2017

    =======================================================================
        Welcome to Cuckoo Sandbox, this appears to be your first run!
        We will now set you up with our default configuration.
        You will be able to modify the configuration to your likings
        by exploring the /home/cuckoo/.cuckoo directory.

        Among other configurable things of most interest is the
        new location for your Cuckoo configuration:
                /home/cuckoo/.cuckoo/conf
    =======================================================================

    Cuckoo has finished setting up the default configuration.
    Please modify the default settings where required and
    start Cuckoo again (by running `cuckoo` or `cuckoo -d`).

从输出消息中可以看到 ``CWD`` 的具体路径。默认是在当前用户目录下  ``~/.cuckoo`` .
配置文件在 ``$CWD/conf`` 目录下.

由于现在有了 ``CWD`` 目录， 配置与Cuckoo的引擎分离， 所以以后的版本更新维护会更方便。
两边都可以独立升级。

CWD 路径
========

默认情况下 ``CWD`` 默认目录是 ``~/.cuckoo`` 。 但是这个路径也是可以通过以下几种方式修改的，
优先级从高到低

* 通过命令行参数 ``--cwd``  (e.g., ``--cwd ~/.cuckoo``).
* 通过配置环境变量 ``CUCKOO``  (e.g., ``export CUCKOO=~/.cuckoo``).
* 通过配置环境变量 ``CUCKOO_CWD`` .
* 当前目录名为 .cuckoo   (e.g., ``cd ~/.cuckoo`` 则会将当前目录作为 ``CWD``).
* 默认路径 ``~/.cuckoo``.

由于 ``CWD`` 目录的可配， 理论上可以并行Cuckoo进程， 例如可以同时运行Windows 和 Android 分析。

下面有一些修改 ``CWD`` 路径的命令样例供参考.

.. code-block:: bash

    # Places the CWD in /opt/cuckoo. Note that Cuckoo will normally create the
    # CWD itself, but in order to create a directory in /opt root capabilities
    # are usually required.
    $ sudo mkdir /opt/cuckoo
    $ sudo chown cuckoo:cuckoo /opt/cuckoo
    $ cuckoo --cwd /opt/cuckoo

    # You could place this line in your .bashrc, for example.
    $ export CUCKOO=/opt/cuckoo
    $ cuckoo

Experimenting with multiple Cuckoo setups is now as simple as creating
multiple ``CWD``'s and configuring them accordingly.

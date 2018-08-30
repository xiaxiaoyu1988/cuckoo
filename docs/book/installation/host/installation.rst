.. _installing:

=================
Cuckoo 安装
=================

创建用户
=============

Cuckoo可以运行在已有用户下面，也可以新建一个用户来跑Cuckoo。
但是要保证虚拟机和Cuckoo运行在相同的用户下。

创建新用户::

    $ sudo adduser cuckoo

If you're using VirtualBox, make sure the new user belongs to the "vboxusers"
group (or the group you used to run VirtualBox)::

    $ sudo usermod -a -G vboxusers cuckoo

如果使用KVM，要将用户加入到Libvirtd用户组::

    $ sudo usermod -a -G libvirtd cuckoo

增加打开文件数限制
===================

在 :doc:`../../faq/index` 文档里的问题 :ref:`openfiles24` 由于操作系统的打开文件
数限制，会导致报表生成失败。

.. _install_cuckoo:

安装 Cuckoo
==============

安装最新版本的Cuckoo比较简单.
我们推荐使用 ``pip`` 和 ``setuptools``来安装最新版本的Cuckoo。 (一些可能存在的问题 :ref:`pip_install_issue`).

.. warning::
   缺少依赖的时候会导致各种问题.建议安装前仔细阅读
   :doc:`requirements` 章节.

.. code-block:: bash

    $ sudo pip install -U pip setuptools
    $ sudo pip install -U cuckoo

*全局* 安装Cuckoo是没有问题的，但是我们 **强力推荐** 用 ``virtualenv`` 来安装 ::

    $ virtualenv venv
    $ . venv/bin/activate
    (venv)$ pip install -U pip setuptools
    (venv)$ pip install -U cuckoo

为什么我们推荐使用 ``virtualenv`` 呢:

* Cuckoo的依赖并不是用的最新版本，可能会与系统已有的版本冲突.
* 系统中其他软件的安装，可能会导致Cuckoo的依赖产生问题.
* 使用virtualenv，可以让非root用户也可以安装相关软件.
* 简单来说virtualenv是最佳实践.

Please refer to :doc:`cwd` and :doc:`../../usage/cwd` to learn more about the
``Cuckoo Working Directory`` and how to operate it.

Install Cuckoo from file
========================

By downloading a hard copy of the Cuckoo Package and installing it *offline*,
one may set up Cuckoo using a cached copy and/or have a backup copy of current
Cuckoo versions in the future. We also feature the option to download such a
tarball on our website.

Obtaining the tarball of Cuckoo and all of its dependencies manually may be
done as follows::

    $ pip download cuckoo

You will end up with a file ``Cuckoo-2.0.0.tar.gz`` (or a higher number,
depending on the latest released stable version) as well as all of its
dependencies (e.g., ``alembic-0.8.8.tar.gz``).

Installing that exact version of Cuckoo may be done as you're familiar with
from installing it using ``pip`` directly, except now using the filename of
the tarball::

    $ pip install Cuckoo-2.0.0.tar.gz

On systems where no internet connection is available, the ``$ pip download
cuckoo`` command may be used to fetch all of the required dependencies and as
such one should be able to - in theory - install Cuckoo completely offline
using those files, i.e., by executing something like the following::

    $ pip install *.tar.gz

Build/Install Cuckoo from source
================================

By cloning Cuckoo Sandbox from our `official repository`_, you can install it from source.
After cloning, follow the steps mentioned in :doc:`../../development/package` to start the installation.

.. _`official repository`: https://github.com/cuckoosandbox/cuckoo

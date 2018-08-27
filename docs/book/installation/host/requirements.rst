============
依赖
============

在安装和配置Cuckoo之前，需要先安装依赖的一些软件和库。

.. note::
    【译者注】 Debian下Apt软件安装，可以去掉命令前面的sudo

安装 Python 库 (Ubuntu/Debian-based)
==================================================================

Cuckoo的管理组件完全由Python脚本编写，所以就需要适合的Python版本。
当前，我们完全兼容的Python版本是 **2.7**。 

老版本的Python和Python 3（未来可能会支持） 目前都是不支持的。

以下一些通过Apt安装的软件都是必须的::

    $ sudo apt-get install python python-pip python-dev libffi-dev libssl-dev
    $ sudo apt-get install python-virtualenv python-setuptools
    $ sudo apt-get install libjpeg-dev zlib1g-dev swig

如果要使用我们基于Django开发的Web界面, 则MongoDB是必须要安装的::

    $ sudo apt-get install mongodb

如果要使用PostgreSQL数据库(推荐), PostgreSQL也必须安装::

    $ sudo apt-get install postgresql libpq-dev

`Yara`_ 和 `Pydeep`_ 是 *可选* 的插件。如果选择安装的话，具体安装步骤可以参考他们的官网.

如果使用KVM的话，则需要安装KVM相关依赖::

    $ sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils python-libvirt

If you want to use XenServer you'll have to install the *XenAPI* Python package::

    $ sudo pip install XenAPI

如果要使用*mitm*辅助模块 ( SSL/TLS 中间人攻击), 
需要安装 `mitmproxy`_. 可以参考官网的相关安装说明.

.. _Yara: https://github.com/plusvic/yara
.. _Pydeep: https://github.com/kbandla/pydeep
.. _mitmproxy: https://mitmproxy.org/

Installing Python libraries (on Mac OS X)
=========================================

This is mostly the same as the installation on Ubuntu/Debian, except that
we'll be using the ``brew`` package manager. Install all the required
dependencies as follows (this list is WIP)::

    $ brew install libmagic cairo pango openssl

In addition to that you'll also want to expose the openssl header files in the
standard GCC/Clang include directory, so that ``yara-python`` may compile
successfully. This can be done `as follows`_::

    $ cd /usr/local/include
    $ ln -s ../opt/openssl/include/openssl .

.. _as follows: https://www.anintegratedworld.com/mac-osx-fatal-error-opensslsha-h-file-not-found/

Installing Python libraries (on Windows 7)
==========================================

To be documented.

虚拟化软件
=======================

Cuckoo沙箱支持大部分的虚拟化软件，可以很方便的添加和使用各种虚拟化支持。

本文档以VirtualBox为例。选择哪种虚拟机软件并不影响后续的分析， 
但是如果你选择了相应的虚拟机，应该按照我们相应的文档和FAQ去配置。

.. note::
    【译者注】 测试过程中选择了KVM

Assuming you decide to go for VirtualBox, you can get the proper package for
your distribution at the `official download page`_. Please find following the
commands to install the latest version of VirtualBox on your Ubuntu LTS
machine. Note that Cuckoo supports VirtualBox 4.3, 5.0, and 5.1::

    $ echo deb http://download.virtualbox.org/virtualbox/debian xenial contrib | sudo tee -a /etc/apt/sources.list.d/virtualbox.list
    $ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
    $ sudo apt-get update
    $ sudo apt-get install virtualbox-5.1

For more information on VirtualBox, please refer to the
`official documentation`_.

.. _VirtualBox: http://www.virtualbox.org
.. _official download page: https://www.virtualbox.org/wiki/Linux_Downloads
.. _official documentation: https://www.virtualbox.org/wiki/Documentation

安装 tcpdump
==================

Tcpdump用于抓取恶意软件运行过程中产生的所有流量。

安装命令::

    $ sudo apt-get install tcpdump apparmor-utils
    $ sudo aa-disable /usr/sbin/tcpdump

``AppArmor`` 只有当PCAP文件生成没有权限的时候才需要，可以参考 :ref:`tcpdump_permission_denied`

禁用了AppArmor 的Linux的平台下， 比如Debian， 仅需要安装 `tcpdump`_::

    $ sudo apt-get install tcpdump

Tcpdump需要root权限，如果不想运行在root用户下，需要做以下设置::

    $ sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

可以用以下命令验证是否配置正确::

    $ getcap /usr/sbin/tcpdump
    /usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip

如果没有`setcap`命令， 则需要安装下面的包::

    $ sudo apt-get install libcap2-bin

或者 (**不推荐**) ::

    $ sudo chmod +s /usr/sbin/tcpdump

需要注意的是 `setcap` 命令不安全，有可能造成提权漏洞，我们建议将Cuckoo安装在专有的环境里。

.. _tcpdump: http://www.tcpdump.org

安装 Volatility
=====================

Volatility 用于分析内存转储文件的可选工具.
Cuckoo与Volatility配合，可以更深度和全面的分析，可以防止恶意软件利用rookit技术逃逸沙箱的监控。

为了能够工作正常，Cuckoo要求Volatility版本不低于 2.3， 推荐最新版本2.5。
可以从官网下载 `official repository`_.

可以查阅Volatility官方文档的安装说明.

.. _official repository: https://github.com/volatilityfoundation

安装 M2Crypto
===================

当前 ``M2Crypto`` 库需要 `SWIG`_ 支持.  Ubuntu/Debian-like 系统下可以通过以下命令安装::

    $ sudo apt-get install swig

``SWIG`` 安装好之后，通过以下命令安装 ``M2Crypto``::

    $ sudo pip install m2crypto==0.24.0

.. _SWIG: http://www.swig.org/

安装 guacd
================

``guacd`` 是RDP，SSH，VNC等远程控制的代理层， 是Cuckoo的Web界面的远程终端中使用，可选。

没有它，远程控制功能就无法使用，版本要求0.9.9及以上。我们推荐安装最新版本
使用如下命令安装::

    $ sudo apt install libguac-client-rdp0 libguac-client-vnc0 libguac-client-ssh0 guacd

如果只需要远程桌面功能，则可以跳过
``libguac-client-vnc0`` 和 ``libguac-client-ssh0`` 两个包.

如果你使用了较老的Linux发行版，又想使用最新的guacd，那只能自己动手编译，就不做过多说明了::

    $ sudo apt -y install libcairo2-dev libjpeg-turbo8-dev libpng-dev libossp-uuid-dev libfreerdp-dev
    $ mkdir /tmp/guac-build && cd /tmp/guac-build
    $ wget https://www.apache.org/dist/guacamole/0.9.14/source/guacamole-server-0.9.14.tar.gz
    $ tar xvf guacamole-server-0.9.14.tar.gz && cd guacamole-server-0.9.14
    $ ./configure --with-init-dir=/etc/init.d
    $ make && sudo make install && cd ..
    $ sudo ldconfig
    $ sudo /etc/init.d/guacd start

When installing from source, make sure you don't have another version of any
of the ``libguac-`` libraries installed from your package manager or you might
experience issues due to incompatibilities which can crash guacd.

Note that the `VirtualBox Extension Pack`_ must also be installed to take
advantage of the Cuckoo Control functionality exposed by Guacamole.

.. _VirtualBox Extension Pack: https://www.virtualbox.org/wiki/Downloads

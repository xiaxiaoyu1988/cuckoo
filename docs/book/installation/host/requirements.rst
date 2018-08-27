============
依赖
============

在安装和配置Cuckoo之前，需要先安装依赖的一些软件和库。

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

Installing tcpdump
==================

In order to dump the network activity performed by the malware during
execution, you'll need a network sniffer properly configured to capture
the traffic and dump it to a file.

By default Cuckoo adopts `tcpdump`_, the prominent open source solution.

Install it on Ubuntu::

    $ sudo apt-get install tcpdump apparmor-utils
    $ sudo aa-disable /usr/sbin/tcpdump

Note that the ``AppArmor`` profile disabling (the ``aa-disable`` command) is
only required when using the default ``CWD`` directory as AppArmor would
otherwise prevent the creation of the actual PCAP files (see also
:ref:`tcpdump_permission_denied`).

For Linux platforms with AppArmor disabled (e.g., Debian) the following
command will suffice to install `tcpdump`_::

    $ sudo apt-get install tcpdump

Tcpdump requires root privileges, but since you don't want Cuckoo to run as
root you'll have to set specific Linux capabilities to the binary::

    $ sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

You can verify the results of the last command with::

    $ getcap /usr/sbin/tcpdump
    /usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip

If you don't have `setcap` installed you can get it with::

    $ sudo apt-get install libcap2-bin

Or otherwise (**not recommended**) do::

    $ sudo chmod +s /usr/sbin/tcpdump

Please keep in mind that even the `setcap` method is not perfectly safe (due
to potential security vulnerabilities) if the system has other users which are
potentially untrusted. We recommend to run Cuckoo on a dedicated system or a
trusted environment where the privileged tcpdump execution is contained
otherwise.

.. _tcpdump: http://www.tcpdump.org

Installing Volatility
=====================

Volatility is an optional tool to do forensic analysis on memory dumps. In
combination with Cuckoo, it can automatically provide additional visibility
into deep modifications in the operating system as well as detect the presence
of rootkit technology that escaped the monitoring domain of Cuckoo's analyzer.

In order to function properly, Cuckoo requires at least version 2.3 of
Volatility, but recommends the latest version, Volatility 2.5. You can
download it from their `official repository`_.

See the volatility documentation for detailed instructions on how to install it.

.. _official repository: https://github.com/volatilityfoundation

Installing M2Crypto
===================

Currently the ``M2Crypto`` library is only supported when `SWIG`_ has been
installed. On Ubuntu/Debian-like systems this may be done as follows::

    $ sudo apt-get install swig

If ``SWIG`` is present on the system one may install ``M2Crypto`` as follows::

    $ sudo pip install m2crypto==0.24.0

.. _SWIG: http://www.swig.org/

Installing guacd
================

``guacd`` is an optional service that provides the translation layer for RDP,
VNC, and SSH for the remote control functionality in the Cuckoo web interface.

Without it, remote control won't work. Versions 0.9.9 and up will work, but we
recommend installing the latest version. On an Ubuntu 17.04 machine the
following command will install version ``0.9.9-2``::

    $ sudo apt install libguac-client-rdp0 libguac-client-vnc0 libguac-client-ssh0 guacd

If you only want RDP support you can skip the installation of the
``libguac-client-vnc0`` and ``libguac-client-ssh0`` packages.

If you are using an older distribution or you just want to use the latest
version (our recommendation), the following will build the latest version
(``0.9.14``) from source::

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

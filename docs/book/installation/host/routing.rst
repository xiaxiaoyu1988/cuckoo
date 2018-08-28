.. _routing:

============================
单次分析路由
============================

从 Cuckoo ``2.0-rc1`` 版本起， 每个文件分析都可以有单独的网络路由。
换句话说，如果要去分析三个文件， 第一个可以不允许访问网络，第二个可以通过VPN访问网络，
第三个可以通过Tor路由访问网络。

然后，除了单次分析路由， 之前的默认路由方式更为常用。

我们的样例里以 ``VirtualBox`` 为例.

.. _simple_global_routing:

全局路由
=====================

在深入功能更丰富更复杂的单次路由之前，我们首先看之前的全局路由方式，
基于 ``iptables`` 规则， 一次设置， 永久有效。

在以下的配置中，我们假设分配给我们 VirtualBox 虚拟机的网络是 ``vboxnet0`` ，
虚拟机的网络是 ``192.168.56.101`` 子网 ``/24`` ， 出口网卡是 ``eth0`` 。
下面的 ``iptables`` 规则设置，将会允许虚拟机访问 Cuckoo  的宿主机以及互联网。

.. code-block:: bash

    $ sudo iptables -t nat -A POSTROUTING -o eth0 -s 192.168.56.0/24 -j MASQUERADE

    # Default drop.
    $ sudo iptables -P FORWARD DROP

    # Existing connections.
    $ sudo iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT

    # Accept connections from vboxnet to the whole internet.
    $ sudo iptables -A FORWARD -s 192.168.56.0/24 -j ACCEPT

    # Internal traffic.
    $ sudo iptables -A FORWARD -s 192.168.56.0/24 -d 192.168.56.0/24 -j ACCEPT

    # Log stuff that reaches this point (could be noisy).
    $ sudo iptables -A FORWARD -j LOG

以上的配置已经差不多了，我们还需要配置一个内核参数，打开IP转发。
不过这个配置是临时生效的，如果需要一直生效，需要在开机启动的时候自动执行下面两条命令::

    $ echo 1 | sudo tee -a /proc/sys/net/ipv4/ip_forward
    $ sudo sysctl -w net.ipv4.ip_forward=1

Iptables 规则也是临时的，重启后失效，如果需要一直生效可以安装 ``iptables-persistent``
或者使用自启动脚本。

有些Linux新的发行版中， 对于网卡的命名规则已经变了， 配置的时候尤其要注意。

单次分析路由配置
====================================

上面已经分析过老的路由方式了， 接下来我们分析更细粒度管理的动态网络路由组件。

如本章引言中所述， 从 ``2.0-rc1`` 版本起, 我们引入了 :ref:`rooter`,
增加了更多的路由方式.

下面是可选的几种路由方式.

+-------------------------+--------------------------------------------------+
| 路由选项                | 描述                                             |
+=========================+==================================================+
| :ref:`routing_none`     | 无路由方式，这是唯一一种不需要Rooter运行的方式,  |
|                         | 默认路由生效。                                   |
+-------------------------+--------------------------------------------------+
| :ref:`routing_drop`     | 完全丢弃所有的非Cuckoo流量,                      |
|                         | 包括虚拟机子网内的流量.                          |
+-------------------------+--------------------------------------------------+
| :ref:`routing_internet` | 完整的互联网访问                                 |
+-------------------------+--------------------------------------------------+
| :ref:`routing_inetsim`  | 将所有流量路由至宿主机上的虚拟网络服务           |
+-------------------------+--------------------------------------------------+
| :ref:`routing_tor`      | 将所有流量路由至Tor网络                          |
+-------------------------+--------------------------------------------------+
| :ref:`routing_vpn`      | 将所有流量路由值预先配置号的多个VPN网络          |
+-------------------------+--------------------------------------------------+

使用单次网络分析路由
==================================

通过上面的描述，我们已经了解了基础知识了，接下来我们开始实践。
Cuckoo 配置好之后， 启动Cuckoo Rooter，选择一种网络模式去分析就简单了。

``Cuckoo Rooter`` 具体说明可以参考
:ref:`cuckoo_rooter_usage` .

.. _routing_iproute2:

配置 iproute2
====================

由于Linux 内核对于 TCP/IP 源路由需要注册所有的网卡信息， 所以我们使用 ``iproute2``。

下面我们将以配置 :ref:`routing_internet` 为例。

假设出口网卡是 ``eth0``。

配置 ``iproute2`` 需要打开 ``/etc/iproute2/rt_tables`` 文件，内容可能如下显示::

    #
    # reserved values
    #
    255     local
    254     main
    253     default
    0       unspec
    #
    # local
    #

选个文件中没有的数字， 在文件末尾新建一行，填入数字 加 网卡名称。
例如::

    #
    # reserved values
    #
    255     local
    254     main
    253     default
    0       unspec
    #
    # local
    #

    400     eth0

如果需要配置多个网卡，每个网卡都需要在文件中配置.

.. _routing_none:

None Routing
^^^^^^^^^^^^

什么都不做，使用 :ref:`simple_global_routing`.

.. _routing_drop:

Drop Routing
^^^^^^^^^^^^

``drop routing`` 跟默认的 :ref:`routing_none`
很像 (如果没有配置全局的 iptables 规则), 不过它不允许虚拟机对互联网的访问。

使用 ``drop routing`` 只允许Cuckoo的内部流量， 任何对外的 ``DNS`` 或者 ``TCP/IP``
都会被阻断。

.. _routing_internet:

Internet Routing
^^^^^^^^^^^^^^^^

该路由模式允许虚拟机有完整的互联网路由， 不过正因为如此， 它允许恶意软件通过上行链路
连接到网络，我们称之为 ``dirty line`` 。

.. note:: ``dirty line`` 的出口网卡需要在 :ref:`routing_iproute2` 中配置.

.. _routing_inetsim:

InetSim Routing
^^^^^^^^^^^^^^^

`InetSim`_ 是一个提供模拟网络服务的开源软件。 如果需要使用 InetSim Routing 
我们需要部署 `InetSim`_ 并且配置相关信息，让Cuckoo 可以使用。

``$CWD/conf/routing.conf`` 中的配置::

    [inetsim]
    enabled = yes
    server = 192.168.56.1

为了尽快的使用InetSim， 可以下载最新的 `REMnux`_ 发布版本，它包含了最新版本的InetSim。

.. note::
    【译者注】 REMnux 是个包含了很多分析工具的虚拟机。

.. _InetSim: http://www.inetsim.org/
.. _REMnux: https://remnux.org/

.. _routing_tor:

Tor Routing
^^^^^^^^^^^

.. note::
    【译者注】 Tor 网络国内基本无法使用，就不翻译了。

.. note:: Although we **highly discourage** the use of Tor for malware analysis
    - the maintainers of ``Tor exit nodes`` already have a hard enough time
    keeping up their servers - it is in fact a well-supported feature.

First of all Tor will have to be installed. Please find instructions on
installing the `latest stable version of Tor here`_.

We'll then have to modify the ``Tor`` configuration file (not talking about
Cuckoo's configuration for Tor yet!) In order to do so, we will have to
provide Tor with the listening address and port for TCP/IP connections and UDP
requests. For a default ``VirtualBox`` setup, where the host machine has IP
address ``192.168.56.1``, the following lines will have to be configured in
the ``/etc/tor/torrc`` file::

    TransPort 192.168.56.1:9040
    DNSPort 192.168.56.1:5353

Don't forget to restart Tor (``/etc/init.d/tor restart``). That leaves us with
the Tor configuration for Cuckoo, which may be found in the
``$CWD/conf/routing.conf`` file. The configuration is pretty self-explanatory
so we'll leave filling it out as an exercise to the reader (in fact, toggling
the ``enabled`` field goes a long way)::

    [tor]
    enabled = yes
    dnsport = 5353
    proxyport = 9040

Note that the port numbers in the ``/etc/tor/torrc`` and
``$CWD/conf/routing.conf`` files must match in order for the two to interact
correctly.

.. _`latest stable version of Tor here`: https://www.torproject.org/docs/debian.html.en

.. _routing_vpn:

VPN Routing
^^^^^^^^^^^

VPN 路由允许通过多个VPN节点进行分析。 通过在配置文件中设置不同国家的VPN信息， 
我们可以模拟在不同的国家和IP地址下， 恶意软件是否有不同的行为。

VPN的配置和虚拟机信息的配置很类似， 配置在 ``$CWD/conf/routing.conf`` 文件中。

一个配置样例如下 ::

    [vpn]
    # Are VPNs enabled?
    enabled = yes

    # Comma-separated list of the available VPNs.
    vpns = vpn0

    [vpn0]
    # Name of this VPN. The name is represented by the filepath to the
    # configuration file, e.g., cuckoo would represent /etc/openvpn/cuckoo.conf
    # Note that you can't assign the names "none" and "internet" as those would
    # conflict with the routing section in cuckoo.conf.
    name = vpn0

    # The description of this VPN which will be displayed in the web interface.
    # Can be used to for example describe the country where this VPN ends up.
    description = Spain, Europe

    # The tun device hardcoded for this VPN. Each VPN *must* be configured to use
    # a hardcoded/persistent tun device by explicitly adding the line "dev tunX"
    # to its configuration (e.g., /etc/openvpn/vpn1.conf) where X in tunX is a
    # unique number between 0 and your lucky number of choice.
    interface = tun0

    # Routing table name/id for this VPN. If table name is used it *must* be
    # added to /etc/iproute2/rt_tables as "<id> <name>" line (e.g., "201 tun0").
    # ID and name must be unique across the system (refer /etc/iproute2/rt_tables
    # for existing names and IDs).
    rt_table = tun0

.. note:: 每个VPN网卡需要在 :ref:`routing_iproute2` 中配置.

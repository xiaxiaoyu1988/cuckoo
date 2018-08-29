===============
启动 Cuckoo
===============

使用如下命令可以启动Cuckoo::

    $ cuckoo

启动后，可以看到如下的日志输出::

      eeee e   e eeee e   e  eeeee eeeee
      8  8 8   8 8  8 8   8  8  88 8  88
      8e   8e  8 8e   8eee8e 8   8 8   8
      88   88  8 88   88   8 8   8 8   8
      88e8 88ee8 88e8 88   8 8eee8 8eee8

     Cuckoo Sandbox 2.0.0
     www.cuckoosandbox.org
     Copyright (c) 2010-2017

     Checking for updates...
     Good! You have the latest version available.

    2017-03-31 17:08:53,527 [cuckoo.core.scheduler] INFO: Using "virtualbox" as machine manager
    2017-03-31 17:08:53,935 [cuckoo.core.scheduler] INFO: Loaded 1 machine/s
    2017-03-31 17:08:53,964 [cuckoo.core.scheduler] INFO: Waiting for analysis tasks.


Cuckoo 会在开始的时候，请求 ``api.cuckoosandbox.org`` 检查更新。
不过可以在配置文件中修改 ``version_check`` 来关闭更新检查。

启动完成后，Cuckoo 就等着提交文件来分析了。

``cuckoo`` 有多个命令行参数，通过 --help 参数可以看到所有的参数::

    $ cuckoo --help
    Usage: cuckoo [OPTIONS] COMMAND [ARGS]...

    Invokes the Cuckoo daemon or one of its subcommands.

    To be able to use different Cuckoo configurations on the same
    machine with the same Cuckoo installation, we use the so-called
    Cuckoo Working Directory (aka "CWD"). A default CWD is
    available, but may be overridden through the following options -
    listed in order of precedence.

    * Command-line option (--cwd)
    * Environment option ("CUCKOO")
    * Environment option ("CUCKOO_CWD")
    * Current directory (if the ".cwd" file exists)
    * Default value ("~/.cuckoo")

    Options:
      -d, --debug             Enable verbose logging
      -q, --quiet             Only log warnings and critical messages
      -m, --maxcount INTEGER  Maximum number of analyses to process
      --user TEXT             Drop privileges to this user
      --cwd TEXT              Cuckoo Working Directory
      --help                  Show this message and exit.

    Commands:
      api          Operate the Cuckoo REST API.
      clean        Clean the CWD and associated databases.
      community    Fetch supplies from the Cuckoo Community.
      distributed  Distributed Cuckoo helper utilities.
      dnsserve     Custom DNS server.
      import       Imports an older Cuckoo setup into a new CWD.
      init         Initializes Cuckoo and its configuration.
      machine      Dynamically add/remove machines.
      migrate      Perform database migrations.
      process      Process raw task data into reports.
      rooter       Instantiates the Cuckoo Rooter.
      submit       Submit one or more files or URLs to Cuckoo.
      web          Operate the Cuckoo Web Interface.

``--debug`` 和 ``--quiet`` 用来控制Cuckoo的日志级别。

.. _cuckoo_background:

后台运行 Cuckoo
========================

刚开始用的时候，手动起几次Cuckoo没什么感觉， 
但是如果有很多台机器去管理的话， 自动化的运行Cuckoo就比较有必要了。

幸运的是，Cuckoo在 ``CWD`` 目录提供了一个 supervisord 的配置文件  ``supervisord.conf`` 。

运行  ``supervisord`` 指定配置文件路径::

    $ supervisord -c $CWD/supervisord.conf

.. note::
    【译者注】 supervisord 类似 Watchdog， 如果Cuckoo进程不存在，就会自动拉起。

需要注意的是， 默认情况下 ``supervisord`` 会启动4个 :ref:`cuckoo_process` 实例。
要把 ``$CWD/conf/cuckoo.conf`` 配置中的 ``process_results`` 选项关闭。

配置好之后， 通过 supervisord 就可以管理cuckoo 进程了， 例如::

    # Stop the Cuckoo daemon and the processing utilities.
    $ supervisorctl stop cuckoo:

    # Start the Cuckoo daemon and the processing utilities.
    $ supervisorctl start cuckoo:

注意下， 命令中 cuckoo 后面 需要有个 冒号 :
表示一组cuckoo进程，包含process 和 daemon进程。
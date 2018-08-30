==================
提交分析
==================

* :ref:`submitpy`
* :ref:`apipy`
* :ref:`distpy`
* :ref:`python`

.. _submitpy:

提交工具
==================

最简单的提交样本分析的方式是通过 ``cuckoo submit`` 命令， 例如::

    $ cuckoo submit --help
    Usage: cuckoo submit [OPTIONS] [TARGET]...

      Submit one or more files or URLs to Cuckoo.

    Options:
      -u, --url           Submitting URLs instead of samples
      -o, --options TEXT  Options for these tasks
      --package TEXT      Analysis package to use
      --custom TEXT       Custom information to pass along this task
      --owner TEXT        Owner of this task
      --timeout INTEGER   Analysis time in seconds
      --priority INTEGER  Priority of this task
      --machine TEXT      Machine to analyze these tasks on
      --platform TEXT     Analysis platform
      --memory            Enable memory dumping
      --enforce-timeout   Don't terminate the analysis early
      --clock TEXT        Set the system clock
      --tags TEXT         Analysis tags
      --baseline          Create baseline task
      --remote TEXT       Submit to a remote Cuckoo instance
      --shuffle           Shuffle the submitted tasks
      --pattern TEXT      Provide a glob-pattern when submitting a
                          directory
      --max INTEGER       Submit up to X tasks at once
      --unique            Only submit samples that have not been
                          analyzed before
      -d, --debug         Enable verbose logging
      -q, --quiet         Only log warnings and critical messages
      --help              Show this message and exit.

通过 ``cuckoo submit`` 可以指定文件或者目录， 如果是目录的话， 会遍历并提交里面的文件。

对于提交为样本类型会在后续的代码中自动分析， 可以参考 :doc:`packages` 

*Example*: 提交一个本地的二进制文件::

    $ cuckoo submit /path/to/binary

*Example*: 提交一个 URL::

    $ cuckoo submit --url http://www.example.com

*Example*: 提交一个本地的二进制文件并且指定了较高的优先级::

    $ cuckoo submit --priority 5 /path/to/binary

*Example*: 提交一个本地的二进制文件并且设置最长分析时间是60秒::

    $ cuckoo submit --timeout 60 /path/to/binary

*Example*: 提交一个本地的二进制文件并且指定文件类型::

    $ cuckoo submit --package <name of package> /path/to/binary

*Example*: 提交一个本地的二进制文件并且指定网络路由方式是tor::

    $ cuckoo submit -o route=tor /path/to/binary

*Example*: 提交一个本地的二进制文件并且指定文件类型，以及指定二进制文件运行时携带的参数::

    $ cuckoo submit --package exe --options arguments=--dosomething /path/to/binary.exe

*Example*: 提交一个本地的二进制文件并且指定运行的虚拟机是 *cuckoo1*::

    $ cuckoo submit --machine cuckoo1 /path/to/binary

*Example*: 提交一个本地的二进制文件并且指定虚拟机平台是windows::

    $ cuckoo submit --platform windows /path/to/binary

*Example*: 提交一个本地的二进制文件并且要求完整内存dumps::

    $ cuckoo submit --memory /path/to/binary

*Example*: 提交一个本地的二进制文件并且强制使用最大的单个样本分析时长::

    $ cuckoo submit --enforce-timeout /path/to/binary

*Example*: 提交一个本地的二进制文件并且指定设置虚拟机的系统日期时间::

    $ cuckoo submit --clock "01-24-2001 14:41:20" /path/to/binary

*Example*: 提交一个本地的二进制文件并且要求内存分析， 且设置内存分析的参数 ::

    $ cuckoo submit --memory --options free=yes /path/to/binary

.. _apipy:

API
===

REST API 的使用方法参考 :doc:`api`.

.. _distpy:

分布式 Cuckoo
==================

分布式的Cuckoo 可以参考
:doc:`dist`.

.. _python:

Python 函数库
================

为了数据库的兼容性，我们使用了一个流行的Python ORM 库 `SQLAlchemy`_，
可以支持多种数据库类型，包括但不限于 SQLite, MySQL or MariaDB, PostgreSQL 。

Cuckoo 被设计成可以方便集成到大的系统中。 我们推荐使用 REST API 接口， 参考 :doc:`api` 。
如果想实现自己的提交脚本，也可以使用  ``add_path()`` 和 ``add_url()`` 函数。

函数接口如下.

.. function:: add_path(file_path[, timeout=0[, package=None[, options=None[, priority=1[, custom=None[, owner=""[, machine=None[, platform=None[, tags=None[, memory=False[, enforce_timeout=False], clock=None[]]]]]]]]]]]]])

    Add a local file to the list of pending analysis tasks. Returns the ID of the newly generated task.

    :param file_path: path to the file to submit
    :type file_path: string
    :param timeout: maximum amount of seconds to run the analysis for
    :type timeout: integer
    :param package: analysis package you want to use for the specified file
    :type package: string or None
    :param options: list of options to be passed to the analysis package (in the format ``key=value,key=value``)
    :type options: string or None
    :param priority: numeric representation of the priority to assign to the specified file (1 being low, 2 medium, 3 high)
    :type priority: integer
    :param custom: custom value to be passed over and possibly reused at processing or reporting
    :type custom: string or None
    :param owner: task owner
    :type owner: string or None
    :param machine: Cuckoo identifier of the virtual machine you want to use, if none is specified one will be selected automatically
    :type machine: string or None
    :param platform: operating system platform you want to run the analysis one (currently only Windows)
    :type platform: string or None
    :param tags: tags for machine selection
    :type tags: string or None
    :param memory: set to ``True`` to generate a full memory dump of the analysis machine
    :type memory: True or False
    :param enforce_timeout: set to ``True`` to force the execution for the full timeout
    :type enforce_timeout: True or False
    :param clock: provide a custom clock time to set in the analysis machine
    :type clock: string or None
    :rtype: integer

    Example usage:

    .. code-block:: python
        :linenos:

        >>> from cuckoo.core.database import Database
        >>> db = Database()
        >>> db.add_path("/tmp/malware.exe")
        1
        >>>

.. function:: add_url(url[, timeout=0[, package=None[, options=None[, priority=1[, custom=None[, owner=""[, machine=None[, platform=None[, tags=None[, memory=False[, enforce_timeout=False], clock=None[]]]]]]]]]]]]])

    Add a local file to the list of pending analysis tasks. Returns the ID of the newly generated task.

    :param url: URL to analyze
    :type url: string
    :param timeout: maximum amount of seconds to run the analysis for
    :type timeout: integer
    :param package: analysis package you want to use for the specified URL
    :type package: string or None
    :param options: list of options to be passed to the analysis package (in the format ``key=value,key=value``)
    :type options: string or None
    :param priority: numeric representation of the priority to assign to the specified URL (1 being low, 2 medium, 3 high)
    :type priority: integer
    :param custom: custom value to be passed over and possibly reused at processing or reporting
    :type custom: string or None
    :param owner: task owner
    :type owner: string or None
    :param machine: Cuckoo identifier of the virtual machine you want to use, if none is specified one will be selected automatically
    :type machine: string or None
    :param platform: operating system platform you want to run the analysis one (currently only Windows)
    :type platform: string or None
    :param tags: tags for machine selection
    :type tags: string or None
    :param memory: set to ``True`` to generate a full memory dump of the analysis machine
    :type memory: True or False
    :param enforce_timeout: set to ``True`` to force the execution for the full timeout
    :type enforce_timeout: True or False
    :param clock: provide a custom clock time to set in the analysis machine
    :type clock: string or None
    :rtype: integer

Example Usage:

.. code-block:: python
    :linenos:

    >>> from cuckoo.core.database import Database
    >>> db = Database()
    >>> db.connect()
    >>> db.add_url("http://www.cuckoosandbox.org")
    2
    >>>

.. _`SQLAlchemy`: http://www.sqlalchemy.org

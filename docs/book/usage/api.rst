========
REST API
========

正在 :doc:`submit` 章节提到的， Cuckoo 提供一个基于 `Flask`_ 的轻量级 RESET APT 服务。

.. _`Flask`: http://flask.pocoo.org/

启动 API 服务
=======================

API 启动命令如下::

    $ cuckoo api

默认情况下绑定的是 **localhost:8090**.
如果需要修改监听，命令如下::

    $ cuckoo api --host 0.0.0.0 --port 1337
    $ cuckoo api -H 0.0.0.0 -p 1337

Web 部署
--------------

默认的方式已经可以处理大部分场景。
如果需要更高的性能和稳定性，可以使用 `uWSGI`_ 和 `nginx`_ 来部署API。

uWSGI 部署需要安装相关依赖::

    $ sudo apt-get install uwsgi uwsgi-plugin-python nginx

uWSGI 设置
^^^^^^^^^^^

First, use uWSGI to run the API server as an application.

首先通过 ``cuckoo api --uwsgi`` 来生成 uWSGI 的配置文件内容， 配置文件存储在
``/etc/uwsgi/apps-available/cuckoo-api.ini`` ，内容如下::

    $ cuckoo api --uwsgi
    [uwsgi]
    plugins = python
    virtualenv = /home/cuckoo/cuckoo
    module = cuckoo.apps.api
    callable = app
    uid = cuckoo
    gid = cuckoo
    env = CUCKOO_APP=api
    env = CUCKOO_CWD=/home/..somepath..

配置文件中大部分内容是继承自 uWSGI的默认配置， 以及导入了 ``cuckoo.apps.api``。
由于示例中 Cuckoo 是通过 virtualenv  来安装的，所以配置中含有了相关信息，
如果不是 virtualenv 安装，则没有类似的配置信息

连接配置文件，启动 uwsgi 应用.

.. code-block:: bash

    $ sudo ln -s /etc/uwsgi/apps-available/cuckoo-api.ini /etc/uwsgi/apps-enabled/
    $ sudo service uwsgi start cuckoo-api    # or reload, if already running

.. note::

   uwsgi 的日志文件路径 ``/var/log/uwsgi/app/cuckoo-api.log``.
   UNIX socket 文件路径
   ``/run/uwsgi/app/cuckoo-api/socket``.

nginx 设置
^^^^^^^^^^^

uWSGI的应用已经跑起来了，接下来把NGINX配置成反向代理模式，转发HTTP请求到uWSGI应用。

通过 ``cuckoo api --nginx`` 命令生成配置文件内容， 
配置文件存储到 ``/etc/nginx/sites-available/cuckoo-api`` 目录::

    $ cuckoo api --nginx
    upstream _uwsgi_cuckoo_api {
        server unix:/run/uwsgi/app/cuckoo-api/socket;
    }

    server {
        listen localhost:8090;

        # REST API app
        location / {
            client_max_body_size 1G;
            uwsgi_pass  _uwsgi_cuckoo_api;
            include     uwsgi_params;
        }
    }

确保 Nginx 有权限连接到uWSGI 应用。
如果 cuckoo 以 **cuckoo** 用户组运行， 则需要将www-data 用户加入到用户组::

    $ sudo adduser www-data cuckoo

链接配置，并启动nginx

.. code-block:: bash

    $ sudo ln -s /etc/nginx/sites-available/cuckoo-api /etc/nginx/sites-enabled/
    $ sudo service nginx start    # or reload, if already running

至此 web 界面就跑起来了， 监听端口是 **8090**。
接下来可以继续调整配置，例如调整nginx的性能参数，或者使用https 服务， 
这些本文档就不做详细说明了， 各位如果有兴趣，可以自己去研究。

.. _`uWSGI`: http://uwsgi-docs.readthedocs.org/en/latest/
.. _`nginx`: http://nginx.org/

接口
=========

下表是当前可用的接口和简单描述， 欲知详情，可以点击接口名称


+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| 接口名称                            | 接口描述                                                                                                         |
+=====================================+==================================================================================================================+
| ``POST`` :ref:`tasks_create_file`   | 提交一个样本并创建分析任务.                                                                                      |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``POST`` :ref:`tasks_create_url`    | 提交一个URL并创建分析任务.                                                                                       |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``POST`` :ref:`tasks_create_submit` | 提交一个或多个样本并创建分析任务.                                                                                |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_list`           | 返回数据中存储的分析任务列表.                                                                                    |
|                                     | 可通过参数控制返回的任务数量.                                                                                    |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_sample`         | 根据样本ID返回任务列表.                                                                                          |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_view`           | 根据任务ID返回任务详情.                                                                                          |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_reschedule`     | 根据任务ID重新开始任务.                                                                                          |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_delete`         | 根据任务ID删除任务和任务报表.                                                                                    |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_report`         | 根据任务ID返回报表内容.                                                                                          |
|                                     | 默认为JSON格式报表，可选其他格式                                                                                 |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_shots`          | 根据任务ID和截图ID返回截图内容.                                                                                  |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_rereport`       | 根据任务ID重新生成报表.                                                                                          |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`tasks_reboot`         | 根据任务ID重启任务.                                                                                              |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`memory_list`          | 根据任务ID 返回memory dump 列表.                                                                                 |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`memory_get`           | 根据任务ID和memory dump id返回Memory dump内容.                                                                   |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`files_view`           | 根据 MD5 hash, SHA256 hash 或者任务ID返回样本信息.                                                               |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`files_get`            | 根据SHA256 hash值获取样本内容.                                                                                   |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`pcap_get`             | 根据任务ID返回PCAP文件内容.                                                                                      |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`machines_list`        | 返回当前可用的虚拟机列表.                                                                                        |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`machines_view`        | 根据虚拟机名称返回虚拟机相信信息.                                                                                |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`cuckoo_status`        | 返回Cuckoo当前状态，包括版本信息和任务概览.                                                                      |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`vpn_status`           | 返回VPN状态.                                                                                                     |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+
| ``GET`` :ref:`exit`                 | 关闭API服务.                                                                                                     |
+-------------------------------------+------------------------------------------------------------------------------------------------------------------+

.. _tasks_create_file:

/tasks/create/file
------------------

**POST /tasks/create/file**

提交一个样本并创建分析任务. 返回创建的任务ID.

**Example request**::

    curl -F file=@/path/to/file http://localhost:8090/tasks/create/file

**Example request using Python**..

.. code-block:: python

    import requests

    REST_URL = "http://localhost:8090/tasks/create/file"
    SAMPLE_FILE = "/path/to/malwr.exe"

    with open(SAMPLE_FILE, "rb") as sample:
        files = {"file": ("temp_file_name", sample)}
        r = requests.post(REST_URL, files=files)

    # Add your code to error checking for r.status_code.

    task_id = r.json()["task_id"]

    # Add your code for error checking if task_id is None.

**Example response**.

.. code-block:: json

    {
        "task_id" : 1
    }

**Form parameters**:

* ``file`` *(required)* - 样本内容 (multipart encoded file content)
* ``package`` *(optional)* - 样本文件类型
* ``timeout`` *(optional)* *(int)* - 分析超时时长 (in seconds)
* ``priority`` *(optional)* *(int)* - 任务优先级 (1-3)
* ``options`` *(optional)* - 样本执行参数
* ``machine`` *(optional)* - 指定运行的虚拟机名称
* ``platform`` *(optional)* - 指定运行的虚拟机平台 (e.g. "windows")
* ``tags`` *(optional)* - 指定虚拟机启动的tags，以逗号分割， 该选项生效的前提是 platform 参数必须设置
* ``custom`` *(optional)* - 自定义字符串，用于分析和报表模块
* ``owner`` *(optional)* - 指定任务所属责任人
* ``clock`` *(optional)* - 设置虚拟机系统时间 (format %m-%d-%Y %H:%M:%S)
* ``memory`` *(optional)* - 开启完整的虚拟机内存dump
* ``unique`` *(optional)* - 只提交样本，不分析
* ``enforce_timeout`` *(optional)* - 开启强制使用最大分析超时时长

**Status codes**:

* ``200`` - 接口提交成功
* ``400`` - 样本已存在 (unique 参数开启的情况下)

.. _tasks_create_url:

/tasks/create/url
-----------------

**POST /tasks/create/url**

提交一个URL并创建分析任务. 返回创建的任务ID.

**Example request**.

.. code-block:: bash

    curl -F url="http://www.malicious.site" http://localhost:8090/tasks/create/url

**Example request using Python**.

.. code-block:: python

    import requests

    REST_URL = "http://localhost:8090/tasks/create/url"
    SAMPLE_URL = "http://example.org/malwr.exe"

    data = {"url": SAMPLE_URL}
    r = requests.post(REST_URL, data=data)

    # Add your code to error checking for r.status_code.

    task_id = r.json()["task_id"]

    # Add your code to error checking if task_id is None.

**Example response**.

.. code-block:: json

    {
        "task_id" : 1
    }

**Form parameters**:

* ``url`` *(required)* - 待分析的URL (multipart encoded content)
* ``package`` *(optional)* - 样本文件类型
* ``timeout`` *(optional)* *(int)* - 分析超时时长 (in seconds)
* ``priority`` *(optional)* *(int)* - 任务优先级 (1-3)
* ``options`` *(optional)* - 样本执行参数
* ``machine`` *(optional)* - 指定运行的虚拟机名称
* ``platform`` *(optional)* - 指定运行的虚拟机平台 (e.g. "windows")
* ``tags`` *(optional)* - 指定虚拟机启动的tags，以逗号分割， 该选项生效的前提是 platform 参数必须设置
* ``custom`` *(optional)* - 自定义字符串，用于分析和报表模块
* ``owner`` *(optional)* - 指定任务所属责任人
* ``memory`` *(optional)* - 开启完整的虚拟机内存dump
* ``enforce_timeout`` *(optional)* - 开启强制使用最大分析超时时长
* ``clock`` *(optional)* - 设置虚拟机系统时间 (format %m-%d-%Y %H:%M:%S)

**Status codes**:

* ``200`` - 提交成功

.. _tasks_create_submit:

/tasks/create/submit
--------------------

**POST /tasks/create/submit**

提交一个或多个样本 或者 多个URL或hash值 并创建分析任务。
返回创建的任务ID列表。

**Example request**.

.. code-block:: bash

    # Submit two executables.
    curl http://localhost:8090/tasks/create/submit -F files=@1.exe -F files=@2.exe

    # Submit http://google.com
    curl http://localhost:8090/tasks/create/submit -F strings=google.com

    # Submit http://google.com & http://facebook.com
    curl http://localhost:8090/tasks/create/submit -F strings=$'google.com\nfacebook.com'

**Example request using Python**.

.. code-block:: python

    import requests

    # Submit one or more files.
    r = requests.post("http://localhost:8090/tasks/create/submit", files=[
        ("files", open("1.exe", "rb")),
        ("files", open("2.exe", "rb")),
    ])

    # Add your code to error checking for r.status_code.

    submit_id = r.json()["submit_id"]
    task_ids = r.json()["task_ids"]
    errors = r.json()["errors"]

    # Add your code to error checking on "errors".

    # Submit one or more URLs or hashes.
    urls = [
        "google.com", "facebook.com", "cuckoosandbox.org",
    ]
    r = requests.post(
        "http://localhost:8090/tasks/create/submit",
        data={"strings": "\n".join(urls)}
    )

**Example response** from the executable submission.

.. code-block:: json

    {
        "submit_id": 1,
        "task_ids": [1, 2],
        "errors": []
    }

**Form parameters**:

* ``file`` *(optional)* - 兼容 :ref:`tasks_create_file` 接口的样本
* ``files`` *(optional)* - 提交分析队列的多个样本
* ``strings`` *(optional)* - 按行分割的多个URL或HASH值列表 (to be obtained using your VirusTotal API key)
* ``timeout`` *(optional)* *(int)* - 分析超时时长 (in seconds)
* ``priority`` *(optional)* *(int)* - 任务优先级 (1-3)
* ``options`` *(optional)* - 样本执行参数
* ``tags`` *(optional)* - 指定虚拟机启动的tags，以逗号分割， 该选项生效的前提是 platform 参数必须设置
* ``custom`` *(optional)* - 自定义字符串，用于分析和报表模块
* ``owner`` *(optional)* - 指定任务所属责任人
* ``memory`` *(optional)* - 开启完整的虚拟机内存dump
* ``enforce_timeout`` *(optional)* - 开启强制使用最大分析超时时长
* ``clock`` *(optional)* - 设置虚拟机系统时间 (format %m-%d-%Y %H:%M:%S)

**Status codes**:

* ``200`` - 提交成功

.. _tasks_list:

/tasks/list
-----------

**GET /tasks/list/** *(int: limit)* **/** *(int: offset)*

返回任务列表.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/list

**Example response**.

.. code-block:: json

    {
        "tasks": [
            {
                "category": "url",
                "machine": null,
                "errors": [],
                "target": "http://www.malicious.site",
                "package": null,
                "sample_id": null,
                "guest": {},
                "custom": null,
                "owner": "",
                "priority": 1,
                "platform": null,
                "options": null,
                "status": "pending",
                "enforce_timeout": false,
                "timeout": 0,
                "memory": false,
                "tags": []
                "id": 1,
                "added_on": "2012-12-19 14:18:25",
                "completed_on": null
            },
            {
                "category": "file",
                "machine": null,
                "errors": [],
                "target": "/tmp/malware.exe",
                "package": null,
                "sample_id": 1,
                "guest": {},
                "custom": null,
                "owner": "",
                "priority": 1,
                "platform": null,
                "options": null,
                "status": "pending",
                "enforce_timeout": false,
                "timeout": 0,
                "memory": false,
                "tags": [
                            "32bit",
                            "acrobat_6",
                        ],
                "id": 2,
                "added_on": "2012-12-19 14:18:25",
                "completed_on": null
            }
        ]
    }

**Parameters**:

* ``limit`` *(optional)* *(int)* - 返回的最大任务数量
* ``offset`` *(optional)* *(int)* - 任务列表开始位置

**Status codes**:

* ``200`` - 成功

.. _tasks_sample:

/tasks/sample
-------------

**GET /tasks/sample/** *(int: sample_id)*

根据样本ID返回任务列表.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/sample/1

**Example response**.

.. code-block:: json

    {
        "tasks": [
            {
                "category": "file",
                "machine": null,
                "errors": [],
                "target": "/tmp/malware.exe",
                "package": null,
                "sample_id": 1,
                "guest": {},
                "custom": null,
                "owner": "",
                "priority": 1,
                "platform": null,
                "options": null,
                "status": "pending",
                "enforce_timeout": false,
                "timeout": 0,
                "memory": false,
                "tags": [
                            "32bit",
                            "acrobat_6",
                        ],
                "id": 2,
                "added_on": "2012-12-19 14:18:25",
                "completed_on": null
            }
        ]
    }

**Parameters**:

* ``sample_id`` *(required)* *(int)* - 样本ID

**Status codes**:

* ``200`` - 成功

.. _tasks_view:

/tasks/view
-----------

**GET /tasks/view/** *(int: id)*

根据任务ID返回任务详情.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/view/1

**Example response**.

.. code-block:: json

    {
        "task": {
            "category": "url",
            "machine": null,
            "errors": [],
            "target": "http://www.malicious.site",
            "package": null,
            "sample_id": null,
            "guest": {},
            "custom": null,
            "owner": "",
            "priority": 1,
            "platform": null,
            "options": null,
            "status": "pending",
            "enforce_timeout": false,
            "timeout": 0,
            "memory": false,
            "tags": [
                        "32bit",
                        "acrobat_6",
                    ],
            "id": 1,
            "added_on": "2012-12-19 14:18:25",
            "completed_on": null
        }
    }

Note: ``status`` 参数包含以下几种状态:

* ``pending``
* ``running``
* ``completed``
* ``reported``

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID

**Status codes**:

* ``200`` - 成功
* ``404`` - 未找到任务

.. _tasks_reschedule:

/tasks/reschedule
-----------------

**GET /tasks/reschedule/** *(int: id)* **/** *(int: priority)*

根据任务ID重新设置任务分析计划，设置任务优先级，默认为 1

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/reschedule/1

**Example response**.

.. code-block:: json

    {
        "status": "OK"
    }

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID
* ``priority`` *(optional)* *(int)* - 任务优先级

**Status codes**:

* ``200`` - 成功
* ``404`` - 未找到任务

.. _tasks_delete:

/tasks/delete
-------------

**GET /tasks/delete/** *(int: id)*

根据任务ID删除任务和任务报表.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/delete/1

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID

**Status codes**:

* ``200`` - 成功
* ``404`` - 未找到任务
* ``500`` - 无法删除任务

.. _tasks_report:

/tasks/report
-------------

**GET /tasks/report/** *(int: id)* **/** *(str: format)*

根据任务ID返回报表内容.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/report/1

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID
* ``format`` *(optional)* - 报表格式 [json/html/all/dropped/package_files]. 默认为JSON格式. ``all`` 返回 tar.bz2 格式的压缩包，包含所有报告文件, ``dropped`` 返回 tar.bz2 格式压缩包， 包含所有样本产生的文件, ``package_files`` 返回分析模块传到宿主机上的所有文件.

**Status codes**:

* ``200`` - 成功
* ``400`` - 报告格式参数错误
* ``404`` - 未找到相应的报告

.. _tasks_shots:

/tasks/screenshots
------------------

**GET /tasks/screenshots/** *(int: id)* **/** *(str: number)*

根据任务ID返回截图内容.

**Example request**.

.. code-block:: bash

    wget http://localhost:8090/tasks/screenshots/1

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID
* ``screenshot`` *(optional)* - 截图的序号 (e.g. 0001, 0002)

**Status codes**:

* ``404`` - 文件或者文件夹未找到

.. _tasks_rereport:

/tasks/rereport
---------------

**GET /tasks/rereport/** *(int: id)*

根据任务ID重新生成报表.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/rereport/1

**Example response**.

.. code-block:: json

    {
        "success": true
    }

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID

**Status codes**:

* ``200`` - 成功
* ``404`` - 未找到任务

.. _tasks_reboot:

/tasks/reboot
-------------

**GET /tasks/reboot/** *(int: id)* **

根据已有的任务分析ID添加重新分析任务.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/tasks/reboot/1

**Example response**.

.. code-block:: json

    {
        "task_id": 1,
        "reboot_id": 3
    }

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID

**Status codes**:

* ``200`` - 成功
* ``404`` - 创建任务失败

.. _memory_list:

/memory/list
------------

**GET /memory/list/** *(int: id)*

根据任务ID返回一个或者多个Memory dump的内容


**Example request**.

.. code-block:: bash

    wget http://localhost:8090/memory/list/1

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID

**Status codes**:

* ``404`` - 未找到文件

.. _memory_get:

/memory/get
-----------

**GET /memory/get/** *(int: id)* **/** *(str: number)*

根据任务ID和Memory dump的序号，返回内容.

**Example request**.

.. code-block:: bash

    wget http://localhost:8090/memory/get/1/1908

**Parameters**:

* ``id`` *(required)* *(int)* - 任务ID
* ``pid`` *(required)* - memory dump 文件序号 (e.g. 205, 1908)

**Status codes**:

* ``404`` - 文件未找到

.. _files_view:

/files/view
-----------

**GET /files/view/md5/** *(str: md5)*

**GET /files/view/sha256/** *(str: sha256)*

**GET /files/view/id/** *(int: id)*

根据指定的 MD5 hash, SHA256 hash 或者样本ID号返回样本信息.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/files/view/id/1

**Example response**.

.. code-block:: json

    {
        "sample": {
            "sha1": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
            "file_type": "empty",
            "file_size": 0,
            "crc32": "00000000",
            "ssdeep": "3::",
            "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
            "sha512": "cf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3e",
            "id": 1,
            "md5": "d41d8cd98f00b204e9800998ecf8427e"
        }
    }

**Parameters**:

* ``md5`` *(optional)* - MD5 值
* ``sha256`` *(optional)* - SHA256 hash 值
* ``id`` *(optional)* *(int)* - 样本 ID

**Status codes**:

* ``200`` - 成功
* ``400`` - 无效的查找项
* ``404`` - 文件未找到

.. _files_get:

/files/get
----------

**GET /files/get/** *(str: sha256)*

 根据 SHA256 hash值返回样本内容.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/files/get/e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855 > sample.exe

**Status codes**:

* ``200`` - 成功
* ``404`` - 文件未找到

.. _pcap_get:

/pcap/get
---------

**GET /pcap/get/** *(int: task)*

根据任务ID返回PCAP文件内容.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/pcap/get/1 > dump.pcap

**Status codes**:

* ``200`` - 成功
* ``404`` - 文件未找到

.. _machines_list:

/machines/list
--------------

**GET /machines/list**

返回可用的虚拟机详情列表.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/machines/list

**Example response**.

.. code-block:: json

    {
        "machines": [
            {
                "status": null,
                "locked": false,
                "name": "cuckoo1",
                "resultserver_ip": "192.168.56.1",
                "ip": "192.168.56.101",
                "tags": [
                            "32bit",
                            "acrobat_6",
                        ],
                "label": "cuckoo1",
                "locked_changed_on": null,
                "platform": "windows",
                "snapshot": null,
                "interface": null,
                "status_changed_on": null,
                "id": 1,
                "resultserver_port": "2042"
            }
        ]
    }

**Status codes**:

* ``200`` - 成功

.. _machines_view:

/machines/view
--------------

**GET /machines/view/** *(str: name)*

根据虚拟机名称返回虚拟机详情.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/machines/view/cuckoo1

**Example response**.

.. code-block:: json

    {
        "machine": {
            "status": null,
            "locked": false,
            "name": "cuckoo1",
            "resultserver_ip": "192.168.56.1",
            "ip": "192.168.56.101",
            "tags": [
                        "32bit",
                        "acrobat_6",
                    ],
            "label": "cuckoo1",
            "locked_changed_on": null,
            "platform": "windows",
            "snapshot": null,
            "interface": null,
            "status_changed_on": null,
            "id": 1,
            "resultserver_port": "2042"
        }
    }

**Status codes**:

* ``200`` - 成功
* ``404`` - 未找到虚拟机

.. _cuckoo_status:

/cuckoo/status
--------------

**GET /cuckoo/status/**

返回cuckoo 当前状态。 
在 1.3 版本中，增加了磁盘状态 包含磁盘已使用，未使用以及总磁盘（仅在类Unix系统中有效）。
同时增加了CPU负载情况，包含CPU过去的1分钟，5分钟和15分钟的负载（仅在类Unix系统中有效）。

**Diskspace directories**:

* ``analyses`` - $CUCKOO/storage/analyses/
* ``binaries`` - $CUCKOO/storage/binaries/
* ``temporary`` - ``tmppath`` as specified in ``conf/cuckoo.conf``

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/cuckoo/status

**Example response**.

.. code-block:: json

    {
        "tasks": {
            "reported": 165,
            "running": 2,
            "total": 167,
            "completed": 0,
            "pending": 0
        },
        "diskspace": {
            "analyses": {
                "total": 491271233536,
                "free": 71403470848,
                "used": 419867762688
            },
            "binaries": {
                "total": 491271233536,
                "free": 71403470848,
                "used": 419867762688
            },
            "temporary": {
                "total": 491271233536,
                "free": 71403470848,
                "used": 419867762688
            }
        },
        "version": "1.0",
        "protocol_version": 1,
        "hostname": "Patient0",
        "machines": {
            "available": 4,
            "total": 5
        }
    }

**Status codes**:

* ``200`` - 成功
* ``404`` - 虚拟机未找到

.. _vpn_status:

/vpn/status
-----------

**GET /vpn/status**

返回VPN状态.

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/vpn/status

**Status codes**:

* ``200`` - 查询成功
* ``500`` - 不可用

.. _exit:

/exit
-----

**GET /exit**

如果在调试模式以及使用的werkzeug服务，可以关闭当前的API 服务

**Example request**.

.. code-block:: bash

    curl http://localhost:8090/exit

**Status codes**:

* ``200`` - 成功
* ``403`` - 该接口仅有debug模式有效
* ``500`` - 报错

=============
Web 界面
=============

Cuckoo 提供一个较为完成的web界面，提供样本提交，报告查看， 分析结果搜索功能。

配置
=============

Web 界面依赖 Mongodb， 如果没有安装或者 ``reporting.conf`` 没有打开开关，运行就会报错。

``$CWD/web/local_settings.py`` 文件中包含了web 界面的配置信息.

.. literalinclude:: ../../../cuckoo/data/web/local_settings.py
    :language: python

生产环境下，我们建议 关闭 ``DEBUG`` 开关， 以及至少配置一个 ``ADMIN`` 信息
用于发送告警的通知邮件。

.. versionchanged:: 2.0.0
   The default maximum upload size has been bumped from 25 MB to 10 GB so that
   virtually any file should be accepted.

启动 Web 界面
==========================

通过如下命令即可启动 Web 界面::

    $ cuckoo web runserver

如果需要指定监听的IP和端口，可以参考如下命令::

    $ cuckoo web runserver 0.0.0.0:PORT

或者::

    $ cuckoo web -H 0

.. _web_deployment:

Web 界面部署
--------------

默认的 Web 界面部署方式基本上没有什么大问题。
但是如果需要更好的性能和稳定性，我们推荐  WSGI 方式部署。
本章简单介绍了， 如何通过 `uWSGI`_ 和 `nginx`_ 来部署。
以下都是以 Ubuntu环境下为例， 但是其他操作系统下，配置也是类似的

首先需要安装相关依赖包::

    $ sudo apt-get install uwsgi uwsgi-plugin-python nginx

uWSGI 设置
^^^^^^^^^^^

首先通过 ``cuckoo web --uwsgi`` 来生成 uWSGI 的配置文件内容， 配置文件存储在
``/etc/uwsgi/apps-available/cuckoo-web.ini`` ，内容如下::

    $ cuckoo web --uwsgi
    [uwsgi]
    plugins = python
    virtualenv = /home/cuckoo/cuckoo
    module = cuckoo.web.web.wsgi
    uid = cuckoo
    gid = cuckoo
    static-map = /static=/home/..somepath..
    # If you're getting errors about the PYTHON_EGG_CACHE, then
    # uncomment the following line and add some path that is
    # writable from the defined user.
    # env = PYTHON_EGG_CACHE=
    env = CUCKOO_APP=web
    env = CUCKOO_CWD=/home/..somepath..

配置文件中大部分内容是继承自 uWSGI的默认配置， 以及导入了 ``cuckoo.web.web.wsgi``。
由于示例中 Cuckoo 是通过 virtualenv  来安装的，所以配置中含有了相关信息，
如果不是 virtualenv 安装，则没有类似的配置信息。

连接配置文件，启动 uwsgi 应用.

.. code-block:: bash

    $ sudo ln -s /etc/uwsgi/apps-available/cuckoo-web.ini /etc/uwsgi/apps-enabled/
    $ sudo service uwsgi start cuckoo-web    # or reload, if already running

.. note::

   uwsgi 的日志文件路径 ``/var/log/uwsgi/app/cuckoo-web.log``.
   UNIX socket 文件路径
   ``/run/uwsgi/app/cuckoo-web/socket``.

nginx 设置
^^^^^^^^^^^

uWSGI的应用已经跑起来了，接下来把NGINX配置成反向代理模式，转发HTTP请求到uWSGI应用。

通过 ``cuckoo web --nginx`` 命令生成配置文件内容， 配置文件存储到 ``/etc/nginx/sites-available/cuckoo-web`` 目录
::

    $ cuckoo web --nginx
    upstream _uwsgi_cuckoo_web {
        server unix:/run/uwsgi/app/cuckoo-web/socket;
    }

    server {
        listen localhost:8000;

        # Cuckoo Web Interface
        location / {
            client_max_body_size 1G;
            uwsgi_pass  _uwsgi_cuckoo_web;
            include     uwsgi_params;
        }
    }

确保 Nginx 有权限连接到uWSGI 应用。
如果 cuckoo 以 **cuckoo** 用户组运行， 则需要将www-data 用户加入到用户组::

    $ sudo adduser www-data cuckoo

链接配置，并启动nginx

.. code-block:: bash

    $ sudo ln -s /etc/nginx/sites-available/cuckoo-web /etc/nginx/sites-enabled/
    $ sudo service nginx start    # or reload, if already running

至此 web 界面就跑起来了， 监听端口是 **8000**。
接下来可以继续调整配置，例如调整nginx的性能参数，或者使用https 服务， 
这些本文档就不做详细说明了， 各位如果有兴趣，可以自己去研究。

.. _`uWSGI`: http://uwsgi-docs.readthedocs.org/en/latest/
.. _`nginx`: http://nginx.org/

===============
Cuckoo 是什么?
===============

Cuckoo是一个开源的恶意软件自动分析系统。
通常被用于在隔离的环境中运行和收集恶意软件的信息以便分析。

它可以分析出以下几种类型的结果:

    * 跟踪恶意软件产生函数调用.
    * 恶意软件执行期间的文件操作，包括新建，删除以及文件下载.
    * 恶意软件的内存转储.
    * PCAP格式的网络流量捕获.
    * 恶意软件运行时的截屏.
    * 虚拟机的完整内存转储文件.

Cuckoo的历史
============

Cuckoo 沙箱最初开始于2010年的谷歌编程之夏中的蜜网项目。它由*Claudio “nex” Guarnieri* 设计和开发，并且
他们现在仍然时该项目的核心开发和领导者。

在2010年夏天的初步开发后， Cuckoo的首个公开测试版本在2011年2月5日发布。


In March 2011, Cuckoo has been selected again as a supported project during
Google Summer of Code 2011 with The Honeynet Project, during which
*Dario Fernandes* joined the project and extended its functionality.

On November 2nd 2011 Cuckoo the release of its 0.2 version to the public as the
first real stable release.
On late November 2011 *Alessandro "jekil" Tanasi* joined the team expanding
Cuckoo's processing and reporting functionality.

On December 2011 Cuckoo v0.3 gets released and quickly hits release 0.3.2 in
early February.

In late January 2012 we opened `Malwr.com`_, a free and public running Cuckoo
Sandbox instance provided with a full fledged interface through which people
can submit files to be analysed and get results back.

In March 2012 Cuckoo Sandbox wins the first round of the `Magnificent7`_ program
organized by `Rapid7`_.

During the Summer of 2012 *Jurriaan "skier" Bremer* joined the development team,
refactoring the Windows analysis component sensibly improving the analysis'
quality.

On 24th July 2012, Cuckoo Sandbox 0.4 is released.

On 20th December 2012, Cuckoo Sandbox 0.5 "To The End Of The World" is released.

On 15th April 2013 we released Cuckoo Sandbox 0.6, shortly after having launched
the second version of `Malwr.com`_.

On 1st August 2013 *Claudio “nex” Guarnieri*, *Jurriaan "skier" Bremer* and
*Mark "rep" Schloesser* presented `Mo' Malware Mo' Problems - Cuckoo Sandbox to the rescue`_
at Black Hat Las Vegas.

On 9th January 2014, Cuckoo Sandbox 1.0 is released.

In March 2014 `Cuckoo Foundation`_ born as non-profit organization dedicated to growth of Cuckoo Sandbox and the
surrounding projects and initiatives.

On 7th April 2014, Cuckoo Sandbox 1.1 is released.

On the 7th of October 2014, Cuckoo Sandbox 1.1.1 is released after a
`Critical Vulnerability`_ had been disclosed by Robert Michel.

On the 4th of March 2015, Cuckoo Sandbox 1.2 has been released featuring a
wide array of improvements regarding the usability of Cuckoo.

During summer 2015 Cuckoo Sandbox started the development of Mac OS X malware
analysis as a `Google Summer of Code`_ project within `The Honeynet Project`_.
*Dmitry Rodionov* qualified for the project and developed a working analyzer
for Mac OS X.

On the 21st of February 2016 `version 2.0 Release Candidate 1`_ is released.
This version ships with almost two years of combined effort into making Cuckoo
Sandbox a better project for daily usage.

.. _`Google Summer of Code`: http://www.google-melange.com
.. _`The Honeynet Project`: http://www.honeynet.org
.. _`Malwr.com`: http://malwr.com
.. _`Magnificent7`: http://community.rapid7.com/community/open_source/magnificent7
.. _`Mo' Malware Mo' Problems - Cuckoo Sandbox to the rescue`: https://media.blackhat.com/us-13/US-13-Bremer-Mo-Malware-Mo-Problems-Cuckoo-Sandbox-Slides.pdf
.. _`Rapid7`: http://www.rapid7.com
.. _`Cuckoo Foundation`: http://cuckoofoundation.org/
.. _`Critical Vulnerability`: https://cuckoosandbox.org/2014-10-07-cuckoo-sandbox-111.html
.. _`version 2.0 Release Candidate 1`: https://cuckoosandbox.org/2016-01-21-cuckoo-sandbox-20-rc1.html

Use Cases
=========

Cuckoo 由于其模块化的设计，既可以作为独立的应用程序，亦可嵌入到大的系统中。

它可以用于分析:

    * 通用的Windows可执行文件
    * DLL文件
    * PDF文档
    * Microsoft Office文档
    * URLs 和 HTML 文件
    * PHP 脚本
    * CPL 文件
    * Visual Basic (VB) 脚本
    * ZIP 文件
    * Java JAR 文件
    * Python 脚本
    * *大部分的其他文件类型*

由于它的模块化以及脚本化， Cuckoo不限制你用它来实现任何系统。

更多信息请参考 :doc:`../customization/index`
章节。

架构
============

Cuckoo沙箱包含了一个核心管理组件，用于管理样本的执行和分析。
每次分析都是在隔离的虚拟或者物理机环境上执行。

Cuckoo 由一个宿主机（管理组件）加上多个沙箱（物理机或者虚拟机）组成。
宿主机上的管理组件负责了一个样本分析的全部过程，样本的执行过程都是在沙箱中进行。

如下图片说明了Cuckoo的主要架构:

    .. image:: ../_images/schemas/architecture-main.png
        :align: center

Obtaining Cuckoo
================

.. deprecated:: 2.0-rc2
    Although Cuckoo can still be downloaded from the website we discourage
    from doing so, given that simply installing it through pip is the
    preferred way to get Cuckoo. Please refer to
    :doc:`../installation/host/installation`.

Cuckoo can be downloaded from the `official website`_, where the stable and
packaged releases are distributed, or can be cloned from our `official git
repository`_.

    .. warning::

        While being more updated, including new features and bugfixes, the
        version available in the git repository should be considered an
        *under development* stage. Therefore its stability is not guaranteed
        and it most likely lacks updated documentation.

.. _`official website`: http://www.cuckoosandbox.org
.. _`official git repository`: http://github.com/cuckoosandbox/cuckoo


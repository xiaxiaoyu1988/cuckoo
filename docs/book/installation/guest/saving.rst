==========================
保存虚拟机
==========================

在快照之前，**一定要确保 Windows系统完全启动了，并且客户端在运行**。

虚拟机准备好之后，做一个快照，保存准备好的状态， 每种虚拟机软件，快照的方式都略有不同。

下面介绍了几种不同的虚拟机下快照的方法。

VirtualBox
==========

VirtualBox可以在图形界面上直接创建快照或者通过命令行创建::

    $ VBoxManage snapshot "<Name of VM>" take "<Name of snapshot>" --pause

快照创建好之后，可以通过如下命令关机和还原快照::

    $ VBoxManage controlvm "<Name of VM>" poweroff
    $ VBoxManage snapshot "<Name of VM>" restorecurrent

KVM
===

如果决定使用KVM的话，首先必须选择一种支持快照的磁盘镜像格式。
我们推荐使用QCOW2格式， 创建快照较为便捷。

使用libvirt来操作KVM虚拟机是最方便的， 它提供了 ``virsh`` 和 ``virt-manager``
的命令行或者界面方式来管理虚拟机。

如果之前创建的虚拟机不是QCOW2的格式，也可以通过命令来转化格式，例如::

    $ cd /your/disk/image/path
    $ qemu-img convert -O qcow2 your_disk.raw your_disk.qcow2

然后修改虚拟机定义文件::

    $ virsh edit "<Name of VM>"

找到磁盘相关的部分::

    <disk type='file' device='disk'>
        <driver name='qemu' type='raw'/>
        <source file='/your/disk/image/path/your_disk.raw'/>
        <target dev='hda' bus='ide'/>
        <address type='drive' controller='0' bus='0' unit='0'/>
    </disk>

修改磁盘的文件后缀::

    <disk type='file' device='disk'>
        <driver name='qemu' type='qcow2'/>
        <source file='/your/disk/image/path/your_disk.qcow2'/>
        <target dev='hda' bus='ide'/>
        <address type='drive' controller='0' bus='0' unit='0'/>
    </disk>

重新开机测试虚拟机是否正常。

快照创建命令如下::

    $ virsh snapshot-create "<Name of VM>"

有多个快照的情况下，有可能导致如下的错误::

    ERROR: No snapshot found for virtual machine VM-Name

虚拟机快照的查看和删除，可以通过以下命令::

    $ virsh snapshot-list "VM-Name"
    $ virsh snapshot-delete "VM-Name" 1234567890

VMware Workstation
==================

VMware也可以通过界面或者命令行的方式来创建快照::

    $ vmrun snapshot "/your/disk/image/path/wmware_image_name.vmx" your_snapshot_name

your_snapshot_name 是快照的名称。
创建完成之后关闭虚拟机，可以通过界面或者命令行方式::

    $ vmrun stop "/your/disk/image/path/wmware_image_name.vmx" hard

XenServer
=========

.. note::
    【译者注】 XenServer没有用到，就不翻译了。

If you decided to adopt XenServer, the XenServer machinery supports starting
virtual machines from either disk or a memory snapshot. Creating and reverting
memory snapshots require that the Xen guest tools be installed in the
virtual machine. The recommended method of booting XenServer virtual machines is
through memory snapshots because they can greatly reduce the boot time of
virtual machines during analysis. If, however, the option of installing the
guest tools is not available, the virtual machine can be configured to have its
disks reset on boot. Resetting the disk ensures that malware samples cannot
permanently modify the virtual machine.

Memory Snapshots
----------------

The Xen guest tools can be installed from the XenCenter application that ships
with XenServer. Once installed, restart the virtual machine and ensure that the
Cuckoo agent is running.

Snapshots can be taken through the XenCenter application and the command line
interface on the control domain (Dom0). When creating the snapshot from
XenCenter, ensure that the "Snapshot disk and memory" is checked. Once created,
right-click on the snapshot and note the snapshot UUID.

To snapshot from the command line interface, run the following command::

    $ xe vm-checkpoint vm="vm_uuid_or_name" new-name-label="Snapshot Name/Description"

The snapshot UUID is printed to the screen once the command completes.

Regardless of how the snapshot was created, save the UUID in the virtual
machine's configuration section. Once the snapshot has been created, you can
shutdown the virtual machine.

Booting from Disk
-----------------

If you can't install the Xen guest tools or if you don't need to use memory
snapshots, you will need to ensure that the virtual machine's disks are reset on
boot and that the Cuckoo agent is set to run at boot time.

Running the agent at boot time can be configured in Windows by adding a startup
item for the agent.

The following commands must be run while the virtual machine is powered off.

To set the virtual machine's disks to reset on boot, you'll first need to list
all the attached disks for the virtual machine. To list all attached disks, run
the following command::

    $ xe vm-disk-list vm="vm_name_or_uuid"

Ignoring all CD-ROM and read-only disks, run the following command for each
remaining disk to change it's behavior to reset on boot::

    $ xe vdi-param-set uuid="vdi_uuid" on-boot=reset

After the disk is set to reset on boot, no permanent changes can be made to the
virtual machine's disk. Modifications that occur while a virtual machine is
running will not persist past shutdown.

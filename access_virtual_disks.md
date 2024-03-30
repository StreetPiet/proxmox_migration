# How to access certain virtual Disk types in Proxmox:

## The Goal:
How to provide Access to certain virtual Disk formats so that we can use standard Tools
like fdisk, cfdisk, dd or qemu-img to access or manipulate virtual Disks.

Be warned! You can easily destroy your Data if you do not know what you are doing!
Never tamper witha virtual Disk while the virtual Machine is running!

## File Based:
### qcow2
If you want to access a virtual Disk, maybe to extract or add some Files you may need the "qemu-nbd" Tool. Here's an example:

 ```qemu-nbd -r -t -c /dev/nbd0 -f vhdx /mnt/nas/Windows10_Backup.vhdx```

This command will create the Device "nbd0" and attaches the virtual Disk to it. After the command you can mount the virtual Disk just like a real Disk or use tools like cfdisk or dd.
You can read or write file based virtual Disks not only on your local Proxmox Server but also over CIFS/SMB or NFS mounts. You can use this command with all formats that Proxmox supports:

### raw

 ```qemu-nbd -r -t -c /dev/nbd0 -f raw /mnt/nas/hddbackup.raw```

### vmdk

```qemu-nbd -r -t -c /dev/nbd0 -f vmdk /mnt/nas/vmwareMachine.vmdk```

## LVM
Proxmox presents LVM-Disks in a convenient way. Look to the ```/dev/mapper/```Directory on your Proxmox Server
and you will find the "raw" Disks of your VMs nicely presented:

```
root@nas02:/dev/mapper# ls -la
total 0
drwxr-xr-x  2 root root     280 Mar 30 12:47 .
drwxr-xr-x 20 root root    4840 Mar 30 14:18 ..
crw-------  1 root root 10, 236 Mar 29 19:35 control
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-data -> ../dm-5
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-data_tdata -> ../dm-3
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-data_tmeta -> ../dm-2
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-data-tpool -> ../dm-4
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-root -> ../dm-1
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-swap -> ../dm-0
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-vm--100--disk--0 -> ../dm-6
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-vm--101--disk--0 -> ../dm-7
lrwxrwxrwx  1 root root       7 Mar 29 19:35 pve-vm--102--disk--0 -> ../dm-8
lrwxrwxrwx  1 root root       7 Mar 30 12:47 pve-vm--106--disk--0 -> ../dm-9
lrwxrwxrwx  1 root root       8 Mar 30 12:47 pve-vm--106--disk--1 -> ../dm-10
root@nas02:/dev/mapper#
```

## ZFS
Almost the same like LVM. The "raw" Disks of your VMs are stored in ```/dev/zvol/<ZFS-Pool-Name>```:

```
root@nas02:/dev/zvol/ZFS-Fast# ls -la
total 0
drwxr-xr-x 2 root root 320 Mar 30 14:18 .
drwxr-xr-x 3 root root  60 Mar 29 19:35 ..
lrwxrwxrwx 1 root root  10 Mar 29 19:35 vm-103-disk-0 -> ../../zd48
lrwxrwxrwx 1 root root  12 Mar 29 19:35 vm-103-disk-0-part1 -> ../../zd48p1
lrwxrwxrwx 1 root root   9 Mar 29 19:35 vm-103-disk-1 -> ../../zd0
lrwxrwxrwx 1 root root  11 Mar 29 19:35 vm-103-disk-1-part1 -> ../../zd0p1
lrwxrwxrwx 1 root root  10 Mar 30 14:18 vm-104-disk-0 -> ../../zd64
lrwxrwxrwx 1 root root  12 Mar 30 14:18 vm-104-disk-0-part1 -> ../../zd64p1
lrwxrwxrwx 1 root root  12 Mar 30 14:18 vm-104-disk-0-part2 -> ../../zd64p2
lrwxrwxrwx 1 root root  12 Mar 30 14:18 vm-104-disk-0-part5 -> ../../zd64p5
lrwxrwxrwx 1 root root  10 Mar 29 19:35 vm-105-disk-0 -> ../../zd32
lrwxrwxrwx 1 root root  10 Mar 30 12:47 vm-106-disk-0 -> ../../zd16
lrwxrwxrwx 1 root root  12 Mar 30 12:47 vm-106-disk-0-part1 -> ../../zd16p1
lrwxrwxrwx 1 root root  12 Mar 30 12:47 vm-106-disk-0-part2 -> ../../zd16p2
lrwxrwxrwx 1 root root  12 Mar 30 12:47 vm-106-disk-0-part3 -> ../../zd16p3
lrwxrwxrwx 1 root root  12 Mar 30 12:47 vm-106-disk-0-part4 -> ../../zd16p4
root@nas02:/dev/zvol/ZFS-Fast#
```

## Ceph
Accessing Ceph is a little bit harder because Proxmox does not provide symbolic Links like it does with LVM or ZFS.

First we need to know which Ceph Pool we have:
```
root@pve:~# ceph osd lspools
1 device_health_metrics
2 vm_storage
```

Then we can list the stored virtual Disks:
```
root@pve:~# rbd -p vm_storage list
vm-100-disk-0
vm-101-disk-0
vm-109-disk-0
vm-109-state-eins
vm-110-disk-0
vm-110-disk-1
```

Now we can map the desired virtual Disk to a local Device. The rbd command tells us which local Device is picked:
```
root@pve:~# rbd -p vm_storage map vm-100-disk-0
/dev/rbd0
```

Now we can use /dev/rbd0 like a normal Disk Drive. If you're done, you can unmap /dev/rbd0:
```
root@pve:~# rbd unmap /dev/rbd0
```


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
You can use this command with all formats that Proxmox supports:

### raw

 ```qemu-nbd -r -t -c /dev/nbd0 -f raw /mnt/nas/hddbackup.raw```

### vmdk

```qemu-nbd -r -t -c /dev/nbd0 -f vmdk /mnt/nas/vmwareMachine.vmdk```

## LVM

## ZFS

## Ceph


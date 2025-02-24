---
title: "How to resize a KVM Virtual Machine partition"
description: "Before the advent of containers, resource encapsulation was primarily achieved using virtualization technology. Consequently, achieving high performance for encapsulated programs posed significant challenges. The development of Kernel-based Virtual Machines (KVM) marked a pivotal advancement, as it introduced the capability to bypass virtualization limitations and minimize performance losses associated with encapsulation. Due to its effectiveness, KVM has become one of the most widely used virtualization solutions, supported by a robust community."
date: 2025-02-24
image: "/images/posts/kvm/kvm.png"
categories: ["tutorials"]
authors: ["Diogo Monteiro"]
tags: ["deploy", "KVM"]
draft: false
---

Before the advent of containers, resource encapsulation was primarily achieved using virtualization technology. Consequently, achieving high performance for encapsulated programs posed significant challenges. The development of Kernel-based Virtual Machines (KVM) marked a pivotal advancement, as it introduced the capability to bypass virtualization limitations and minimize performance losses associated with encapsulation. Due to its effectiveness, KVM has become one of the most widely used virtualization solutions, supported by a robust community.

This tutorial has been developed using Ubuntu 22.04.

### Resizing Virtual Disk

Firstly, identify the virtual disk's path (commonly `/var/lib/libvirt/images/<vm_name>.qcow2`):
```bash
virsh domblklist <vm_name>
```

Resize the image, changing the size according to your preferences:
```bash
qemu-img resize <vm_disk_path> +20G
```

Check if there are any VM snapshots:
```bash
virsh snapshot-list <vm_name>
```

Delete the remaining ones:
```bash
virsh snapshot-delete --doman <vm_name> --snapshotname <snapshot_name>
```

### Resizing partition

Inside your VM, identify the device your partition owns:
```bash
lsblk
```

Then, use the interactive command to edit the partition:
```bash
fdisk <device_path>
```
- p: list partitions
- d: delete partition
- n: new partition
- w: write changes on partition table

After that, you need to reload the partition table by simply rebooting:
```bash
reboot
```

Then, finally, resize the filesystem:
```bash
resixe2fs <partition_path>
```

### Resizing Logical Partition

If you have logical partitions (LVM) that were not resized, you can change them using the following command:
```bash
lvextend --resizefs -L +20G <filesystem_name>
```

If you want to allocate all available space, simply run:
```bash
lvextend --resizefs -l +100%FREE <filesystem_name>
```

You can find out the filesystem name by running:
```bash
lsblk
```

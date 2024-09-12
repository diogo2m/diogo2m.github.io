---
title: "How to create a Virtual Machine using KVM hypervisor"
description: "Before the advent of containers, resource encapsulation was primarily achieved using virtualization technology. Consequently, achieving high performance for encapsulated programs posed significant challenges. The development of Kernel-based Virtual Machines (KVM) marked a pivotal advancement, as it introduced the capability to bypass virtualization limitations and minimize performance losses associated with encapsulation. Due to its effectiveness, KVM has become one of the most widely used virtualization solutions, supported by a robust community."
date: 2024-06-29
image: "/images/posts/kvm/kvm.png"
categories: ["tutorials"]
authors: ["Diogo Monteiro"]
tags: ["deploy", "KVM"]
draft: false
---

Before the advent of containers, resource encapsulation was primarily achieved using virtualization technology. Consequently, achieving high performance for encapsulated programs posed significant challenges. The development of Kernel-based Virtual Machines (KVM) marked a pivotal advancement, as it introduced the capability to bypass virtualization limitations and minimize performance losses associated with encapsulation. Due to its effectiveness, KVM has become one of the most widely used virtualization solutions, supported by a robust community.

This tutorial has been developed using Ubuntu 22.04.

### Installing KVM

Firstly, update apt-get:
```bash
sudo apt-get update && sudo apt-get upgrade
```

Then, install the KVM packages:
```bash
sudo apt -y install bridge-utils cpu-checker libvirt-clients libvirt-daemon libvirt-daemon-system virtinst qemu qemu-kvm
```

### Checking virtualization availability

First of all, you need to verify if your CPU supports virtualization. In this case, we are checking if your processor (Intel or AMD) indicates that it have virtualization technology (i.e. if its more than zero, it supports).
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

To make sure that KVM VM can run in the system:
```bash
kvm-ok
# Expected output: INFO: /dev/kvm exists \ KVM acceleration can be used
```
### Adjusting KVM to work

Add your user to the `libvirt`and `kvm`groups:
```bash
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```

Start `libvirtd`service:
```bash
sudo systemctl enable --now libvirtd
```

### Creating a Ubuntu 22.04 VM

Create the repository where the VM's images will be stored:
```bash
sudo mkdir -p /var/kvm/images
```

Create the VM, changing specifications as demanded:
```bash
sudo virt-install \
--name vm_name \
--ram 2048 \
--disk path=/var/kvm/images/vm_name.img,size=30 \
--vcpus 2 \
--os-variant ubuntu22.04 \
--network network=default \
--graphics none \
--console pty,target_type=serial \
--location /home/ubuntu-22.04-live-server-amd64.iso,kernel=casper/vmlinuz,initrd=casper/initrd \
--extra-args 'console=ttyS0,115200n8' 
```

> [NOTE:] To other values to `os-variant` install `libosinfo-bin` and execute:
> ```bash
> osinfo-query os
> ```

Now, if all is correct you should be able to see VM here.
```bash
sudo virsh list --all
```

### Useful commands
<details>

```bash
sudo virsh <command> <vm_name>
```

Available commands:
- start: Starts VM
- shutdown: Turns off VM
- destroy: Force VM to shutoff
- undefine: deletes VM's domain
- console: Access VM's console (just one session allowed)
- edit: Allows to modify VM's yaml file

#### Virtual network commands
<details>

The same commands work for the virtual network, adding "net-" before command, just like the follow:
```bash
sudo virsh net-<command> <net_name>
```

You can also use `net-list` to list all available networks.
```bash
sudo virsh net-list
```
</details>

#### Creating a new disk
<details>

Creates a new disk:
```bash
qemu-img create -f qcow2 /path/to/new/disk.qcow2 10G
```

Attributes the disk to the virtual machine:
```bash
virsh attach-disk <vm_name> /path/to/new/disk.qcow2 vdb --cache none --subdriver qcow2 --persistent
```
</details>

#### Creating a new interface
<details>

To list VM network interfaces:
```bash
sudo virsh domiflist <vm_name>
```

To create a new interface:
```bash
sudo virsh attach-interface --domain <vm_name> --type bridge --source br0 --model virtio --config --live
```

Here is a an example of a interface in the VM file:
```xml
    <interface type='bridge'>
      <mac address='52:54:00:3f:b9:0a'/>
      <source bridge='br-mgmt'/>
      <target dev='vnet0'/>
      <model type='virtio'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </interface>
``
</details>

</details>

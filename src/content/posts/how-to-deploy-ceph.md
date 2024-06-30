---
title: "How to deploy Ceph in Ubuntu 22.04"
description: "Given the technological advancements, data traffic has become more abundant and valuable. Considering this, along with the numerous benefits of Distributed Systems (DS), Ceph has emerged as a reliable and scalable tool to enhance data redundancy and storage within your local infrastructure."
date: 2024-06-29
image: "/images/posts/computer-components/hd/how-to-deploy-ceph.jpg"
categories: ["tutorials"]
authors: ["Diogo Monteiro"]
tags: ["deploy", "DS", "Ceph"]
draft: false
---

Given the technological advancements, data traffic has become more abundant and valuable. Considering this, along with the numerous benefits of Distributed Systems (DS), Ceph has emerged as a reliable and scalable tool to enhance data redundancy and storage within your local infrastructure.

Before continue, you need to consider some characteristics of our deployment. We will use 3 machines with one disk without formatation (in our environment in /dev/vdb).

### Basic configuration (all nodes)

Create user `cluster-admin`.
```bash
sudo adduser cluster-admin
```

Add user to `sudo` group.
```bash
sudo usermod -aG sudo cluster-admin
```

To allow cluster-admin to use privilege commands without the necessity to use password, change the sudoers file adding `cluster-admin ALL = (root) NOPASSWD:ALL` one line before `@includedir /etc/sudoers.d`.
```bash
sudo vim /etc/sudoers
```

In `/etc/hosts/` add the follow lines, adapting each IP for your case.
```bash
192.168.123.110          ceph-mon
192.168.123.111          ceph-osd1
192.168.123.112          ceph-osd2
```

Install docker.io.
```bash
sudo apt update && sudo apt upgrade
sudo apt install -y docker.io
```

To generate a public ssh key (without pass-key):
```bash
ssh-keygen -t rsa -b 4096 -C "ceph-mon"
```

To copy ssh keys to the ceph-osds:
```bash
ssh-copy-id ceph-osd1
ssh-copy-id ceph-osd2
```

### Configuring Ceph in the master (ceph-mon)

Firstly, download `cephadm`
```bash
mkdir bin
cd bin
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm
chmod u+x cephadm
cd ..
```

Now, we need to choose which version of `cephadm` we will use, at this tutorial we will use Pacific 16.2.10. For more verions consult [ceph documentation](https://docs.ceph.com/en/latest/releases/index.html)
```bash
sudo ./bin/cephadm add-repo --release 16.2.10
```

Install `ceph-common`.
```bash
sudo apt install -y ceph-common
```

Add your first monitor. At `bootstrap_output.txt` there will be some metrics about Ceph Dashboard.
```bash
sudo ./bin/cephadm bootstrap --mon-ip 192.168.123.110  &> bootstrap_output.txt
```

To verify Ceph status:
```bash
sudo ceph status
```

When cluster status is `HEALTH_WARN` in some cases, it is because the current number of OSDs is lesser than the default replicas pool size. Considering that we will have just two, we need to change replicas size to two.
```bash
sudo ceph config set global osd_pool_default_size 2
```

To verify if it was correctly changed:
```bash
sudo ceph config get osd osd_pool_default_size
```

To maintain the number of monitor and manager fixed to one:
```bash
sudo ceph orch apply mon --placement="1 experiment-ceph-mon"
sudo ceph orch apply mgr --placement="1 experiment-ceph-mon"
```

### Configuring Ceph in the clients (ceph-osds)

Install `ceph-common`.
```bash
sudo apt install -y ceph-common
```

After properly installing Ceph, we need to copy some config files from master node.
```bash
sudo scp cluster-admin@ceph-mon:/etc/ceph/ceph.conf /etc/ceph/ceph.conf
sudo scp cluster-admin@ceph-mon:/etc/ceph/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring 
```

Now, follow with [additional commands](#additional-commands) to a complete configuration.

### Additional commands

#### Add nodes

Execute the follow commands at `ceph-mon`.
```bash
sudo ceph orch host add cluster-admin ceph-osd1 --labels=osd
sudo ceph orch host add cluster-admin ceph-osd2 --labels=osd
```

To verify if they were properly added to the cluster:
```bash
sudo ceph orch host ls
```

#### Add OSD units

Execute the follow commands at `ceph-mon`.
```bash
sudo ceph orch device ls
sudo ceph orch daemon add osd ceph-osd1:/dev/vdb
sudo ceph orch daemon add osd ceph-osd2:/dev/vdb
```

#### Mount RBD images at ceph-mon

Create a pool called `main-block-devices`.
```bash
sudo ceph osd pool create main-block-devices
```

Verify if pool was properly created.
```bash
sudo ceph osd pool ls
```

Start the pool.
```bash
sudo rbd pool init main-block-devices
```

Create a rbd called `foo`.
```bash
sudo rbd create main-block-devices/foo --size 1024
```

Verify if rbd was properly created.
```bash
sudo rbd -p  main-block-devices ls
```

### Mount RBD images at ceph-osds

Execute the follow commands at your nodes.
```bash
sudo rbd map foo --name client.admin -m ceph-mon -k /etc/ceph/ceph.client.admin.keyring -p main-block-devices
sudo mkfs.ext4 /dev/rbd/main-block-devices/foo
sudo mkdir /mnt/ceph-rbd
sudo mount /dev/rbd/main-block-devices/foo /mnt/ceph-rbd
```

To verify if disk is mounted:
```bash
lsblk
```
# Kubernetes the Hard Way

To become a better Kubernetes practioner I know I have to move deeper down the layers of abstraction. To do so I came across 
[Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way). Using this guide I am going to leverage my homelab setup to bootstrap a kubebernetes cluster in my home network. The end goal being a some Ansible code that allows me to automate this process on any set of Ubuntu server machines.

## Table of Contents

- [Kubernetes the Hard Way](#kubernetes-the-hard-way)
  - [Table of Contents](#table-of-contents)
  - [What you Will Need:](#what-you-will-need)
    - [Homelab](#homelab)
  - [1. Setting up the virtual machines](#1-setting-up-the-virtual-machines)
    - [IP Address](#ip-address)
    - [SSH](#ssh)
    - [Update the VM and install required software](#update-the-vm-and-install-required-software)
    - [Overview](#overview)


## What you Will Need:
### Homelab

First of all, if you follow along with the [original guide](https://github.com/kelseyhightower/kubernetes-the-hard-way) then you will be using Google Cloud Platform.  But any cloud platform that allows you to create a Virtual Private Cloud and some Virtual Machines would work.

So what am I working with?  My VPC is my home network and for the machines I have 3 8gb Raspberry Pi's. Due to my limit of 3 machines I won't be deploying a highly available cluster. But the goal is if I did have 6 Pi's or some other variation of ARM machines, I could run the Ansible code to provision 3 workers + 3 control plane nodes.

**NOTE** - If you are aware of the Raspberry Pi world, it can be hard to get your hands on one. I recommend following [Rpilocator](https://rpilocator.com/) to get live updates of when they are back in stock, wherever you live.

## 1. Setting up the virtual machines

### IP Address

In my environment, the machines are already given an IP address within my defined subnet via my DHCP server. To make sure the addresses do not change on me, I have set some DHCP static reservations for those machines. The other method would be to ssh into each VM and reconfigure each network interface with a static IP. It would look something like:

```
# Fresh Ubuntu server default username/password after installing
$ ssh ubuntu@192.168.x.x
Enter password: ubuntu

$ ubuntu@192.168.x.x: sudo nano /etc/network/interfaces

# interface file auto-generated by buildroot

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
  address 192.168.1.100
  netmask 255.255.255.0
  gateway 192.168.1.1
  pre-up /etc/network/nfs_check
  wait-delay 15
  hostname $(hostname)
```

The above commands have given the VM the IP `192.168.1.100`

### SSH 

By default, a freshly installed Ubuntu server has the ssh credentials of username `ubuntu` and password `ubuntu`. I spent a bunch of time trying to figure out how to automate the copying of my ssh public key over to my 3 machines using an Ansible Task.  It can be done with something called [sshpass](https://ports.macports.org/port/sshpass/) but from what I read it's bad practice as ssh passwords should be typed in interactively using a physical keyboard.  

Since I only have 3 machines, I ran the command:

```bash
  ssh-copy-id -i ~/.ssh/<your-private-key-file> ubuntu@ip.ad.ddr.ess
```

for each machine.

### Update the VM and install required software

The code for this can be found in [/roles/1_configure_machines](https://github.com/jonparkdev/ansible-kubernetes/tree/main/roles/1_configure_machines) directory.

Summary:
- Ensure virtual machine have standard security configuration using [Jeff Geerling](https://www.youtube.com/@JeffGeerling)'s [Ansible Security Role](https://github.com/geerlingguy/ansible-role-security)
- Install Kubectl using apt so machines can interface with the cluster
- Install cfssl and cfssljson for provision required certificates (See next section)

### Overview

If you are following to [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way), this subsection maps one to one with the following sections:

- 01-prequesites
- 02-client-tools
- 03-compute-resources

---
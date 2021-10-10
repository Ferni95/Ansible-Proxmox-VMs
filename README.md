Ansible Role: Create virtual Machines with proxmox and install or clone with cloud-init. Support multiple Linux Distos.
=========

The villafer.promox_VMs, create or clone all VMs, that are passed via vars file, and install cloud-init Linux distros with differents custom parameters for each VM. I designed this in a similar way that Vagrant will do to create VMs but with Proxmox. Also I set the option to Power On the proxmox server, for the people like me that don't have the server power on 24x7.
The role auto detect the new valid VMID in Proxmox and set a new VM, also one of the best features it integrate is that you don't need to have a DHCP server or setr the IP manually, the IP for cloud-init VMs is set based on the VMID, in a similar way the cloud providers, like AWS or Azure do.
The main.yml of the 'tasks' directory, is the main file in the role, here you can see the diffents tags that could be use. Power-on proxmox host, deploy/clone VMs, update config of the VMs, start/stop VMs, delete VMs and shutdown proxmox host.

Requirements
------------

Proxmox server, with promoxer
Ansible lastest version, tested on ansible core 2.11.5 with Python 3.8
Ansible comunity.general collection version 3.7.0
Proxmox API user and recommended ssh key for root in proxmox server

Dependencies
------------

Proxmoxer in proxmox server
Ansible comunity.general collection version 3.7.0

Role Variables
--------------
All the vars shown here could be set in one file or in multiple one, I recommend using one file for Proxmox variables, VMs default variables and images URLs. One file for VMs configuration using this you can easily update, and change the VMs you want to create/start/delete or the different tasks that could be set in this role.

Vars related to proxmox server, under "pve" 

| Name | Description |
| --- | --- |
| `prox_user` | Proxmox user (ex: `root@pam`) |
| `prox_password` | Proxmox user password |
| `prox_node` | node name (ex: `node1`) |
| `host_ip` | Proxmox user (ex: `root@pam`) |
| `clone_timeout` | Timeout for cloning each VM  |
| `mac` | MAC address of the proxmox server for WOL|
| `broadcast` | Broadcast address for WOL |
| `img_dir` | Directory to download cloud-init images |
| `test_port` | Proxmox web port, most of the tasks are done via API|
| `test_delay` | Delay after WOL to start checking for connectiviy |
| `test_timeout` | timeout after WOL |

Variables related to VMs, could be set under "vms" dictionary (using separate files) and "vms_def" (for defaults) in vars/main/main.yml. Please set this file with your preferences. Currently the role will fail if the variables is not set at least in "vms_def" or "vms" dictionary, I need to investigate how to set us optional and with default variable at the same time.
| Name | Description |
| --- | --- |
| 'name'| Name of the VM you want to create *Only compulsory variable in vms dictionary*|
| 'template'| Name of a VM or template to clone, used with clone tag |
| 'user' | Name of the user for the VM (ex: 'fernando') (*cloud-init required*)|
| 'password' | Password for the user in the VM *optional, and not recommended use ssh-key* |
| 'ssh_key' | Public SSH key (OpenSSH format - *cloud-init required*) |
| 'dns1' | Ip of the primary DNS for the VM (*cloud-init required*) |
| 'dns2' | Ip of the secondary DNS for the VM (*cloud-init required*) |
| 'hdd_size' | Desired size of the primary HDD, accept both full size (ex: '16G') or increment size (ex: '+8G') |
| 'cores' | Number of cores per socket |
| 'ram' | Memory size in MB for VM (ex:'2048') |
| 'network_device' | Dictionary of network interfaces for the VM (ex: 'virtio,bridge=vmbr0') |
| 'clone_full' | yes or no *Currently this is not a VM var this role only accept full clones, sync clone will be in future updates*|
| 'storage' | proxmox storage for VMs (ex: 'lvm-thin') |
| 'distro' | Name of the dictionary, for download and install cloud-init images |
Proxmox recommened config for cloud init
| 'boot_order' | VM boot order *Proxmox recommened config for cloud init "c"*
| 'boot_disk' | VM boot disk *Proxmox recommened config for cloud init "scsi0"*
| *This values are for all the VMs, only in main.yml pending to add as VM variable in future updates* |
| 'network' | Network WITHOUT the last digit (ex: "192.168.33.")(*cloud-init required*) |
| 'netmask' | Network mask (ex: "24") (*cloud-init required*) |
| 'gateway' | Network gateway (ex: "192.168.33.1") (*cloud-init required*) |

Variables for img
| Name | Description |
| --- | --- |
| 'url'| URL of the cloud init image you want to install, this is set in a dictionary manner |

Please take into account that "disto" variable in VM, has to be a valid entry in the img_templates dictionary

This is OK
```yaml
# File vars/main/vms.yml defining the vm especific config
vms:
  Docker2:
    name: Docker20
    distro: ubuntu

# File vars/main/main.yml
img_templates:
  centos7:
    url: https://cloud.centos.org/altarch/7/images/CentOS-7-x86_64-GenericCloud.qcow2
  ubuntu:
    url: https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```
This is not OK, the value under distro is not available in img_templates
```yaml
# File vars/main/vms.yml defining the vm especific config
vms:
  Docker2:
    name: Docker20
    distro: centos8

# File vars/main/main.yml
img_templates:
  centos7:
    url: https://cloud.centos.org/altarch/7/images/CentOS-7-x86_64-GenericCloud.qcow2
  ubuntu:
    url: https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```

Roles tags that could be selected
----------------

| Name | Description |
| --- | --- |
| `up` | WOL promox server, start VMs *Best way to start VMs*|
| `deploy` | WOL promox server, create VMs, update VMs config and start VMs |
| `clone` | WOL promox server, clone VMs, update VMs config and start VMs |
| `start` | Start VMs |
| `stop` | Stop VMs  |
| `delete` | Delete VMs |
| `halt` | Stop VMs in vars file and shutdown proxmox server  |
| `update` | Update the VMs config *Be careful if using hdd_size increments* |

Example Playbook
----------------

```
ansible-playbook playbook.yml -i inventory -u root --tags "deploy"
```
```
- hosts: proxmox1
  gather_facts: false
  
  roles:
    - proxmox
```
My inventory is:
```
[proxmox1]
pve1 ansible_ssh_host=192.168.35.6

[localhost]
localhost
```
License
-------

MIT

Author Information
------------------

Villafer, 
Fernando.

Please if you have any new feauture do you want implemented, please let me know, feel free to improve this role, if you do this keep me informed, I want to learn other ways.

Future features
------------------

* Make network VM variable
* Enable sync clones
* Make sure the ansible.comunity.general 4.0.0 defaults changes do not affect this role
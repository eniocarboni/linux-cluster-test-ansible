## Linux-cluster-test-ansible

[![GPL License](https://img.shields.io/badge/license-GPL-blue.svg)](https://www.gnu.org/licenses/)
[![Release v 0.1](https://img.shields.io/badge/release-v.0.1-green.svg)](https://github.com/eniocarboni/Linux-cluster-test-ansible)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.me/EnioCarboni/)

**Linux-cluster-test-ansible** is a Vagrant environment test for Linux cluster based on Pacemaker and Corousync using pcs CLI interface.
It use Ubuntu 1804 lts+, Centos 7 or AmlaLiux 8+ vm.

It is based on the **[Linux-cluster-test project](https://github.com/eniocarboni/linux-cluster-test)** which, however, did not use ansible. 

## Installation

```
mkdir linux-cluster-test-ansible
cd linux-cluster-test-ansible
git clone https://github.com/eniocarboni/linux-cluster-test-ansible .
cp provision/vars/cluster_password.TMPL.yml provision/vars/cluster_password.yml
ansible-galaxy collection install -r provision/requirements.yml

# edit provision/vars/cluster_password.yml and change the 3 password to secure installation
```

Based on your needs you may want to create an encrypted ansible-vault containing the passwords from the provision/vars/cluster_password.yml file.
This way **ansible** will ask you to enter your password each time to open the vault:

```
ansible-vault encrypt provision/vars/cluster_password.yml
sed -i 's/#ansible.ask_vault_pass/ansible.ask_vault_pas/' Vagrantfile
```
Check configuration variables in _provision/vars/cluster.yml_

## Use

First, check the Vagrant file to see $BOX variable.
by default use **ubuntu/focal64** (Ubuntu 20.04 LTS) and 3 cluster node and a simple fence ssh agent (see https://github.com/nannafudge/fence_ssh).

```
vagrant up --provision
```

### Add new nodes to cluster

You can add a node by changing the variable **$NUM_CLUSTER_NODES** in Vagrant file, for example from 3 to 4.
At this point you just need to relaunch "vagrant up && vagrant provision" or

```
vagrant up --provision
```

### Remove a node from cluster

Suppose we start with the default cluster with 3 nodes.
To decrease the cluster by one node you must:
* launch "vagrant destroy cl-node-3"
* update the Vagrantfile by decreasing the $NUM_CLUSTER_NODES variable by one (put it at 2)
* launch "vagrant up --provision"

### Restoring a node for problems

Suppose we want to restore a problematic node, for example node 2:
* launch "vagrant destroy cl-node-2"
* launch "vagrant up --provision"

### Provision with ansible ###

We could directly use **ansible** for the "provision" instead of vagrant:

```
ansible-playbook --inventory-file=.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory provision/playbook.yml
```

In this way we could take advantage of tags with ansible:

```
ansible-playbook --inventory-file=.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory --tags="fence" provision/playbook.yml
# or 
ansible-playbook --inventory-file=.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory --tags="controller" provision/playbook.yml
```

## Notes on operation

### Note about the cluster resources created

The provision scripts create 4 cluster resources groupped in 3 resources:
* group apachegroup:
  * resource first_test_ip (`ocf::heartbeat:IPaddr2`)
  * resource Web1 (`ocf::heartbeat:apache`)
* group group_second_test_ip
  * resource second_test_ip (`ocf::heartbeat:IPaddr2`)
* group group_last_test_ip
  * resource last_test_ip (`ocf::heartbeat:IPaddr2`)

View **resources** in _provision/vars/cluster.yml_

If **fence_agent** is true the provision scripts create one fence resource for each node:
* stonith-ssh-1 (for node 1)
* stonith-ssh-2 (for node 2)
* stonith-ssh-3 (for node 3)
and a constraint resource for each fence resource so that to not start in the same node relative to the fence:
* stonith-ssh-1 not start on node 1
* stonith-ssh-2 not start on node 2
* stonith-ssh-3 not start on node 3

View **fence_agent** in _provision/vars/cluster.yml_

You may set **fence_agent: false** to remove all stonith resources and remove 3rd network interface and all refering to **fence**

### Note on firewall

On RedHat family box use **firewalld** while on Ubuntu/Debian use **ufw**

## COPYRIGHT

Copyright (c) 2022 Enio Carboni - Italy

This file is part of **Linux-cluster-test-ansible**.

**Linux-cluster-test-ansible** is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

**Linux-cluster-test-ansible** is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with **Linux-cluster-test**. If not, see <http://www.gnu.org/licenses/>.

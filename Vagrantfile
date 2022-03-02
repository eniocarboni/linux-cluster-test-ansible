# **
# Linux-cluster-test-ansible
# Copyright (c) 2022 Enio Carboni (enio.carboni __at__ gmail.com)
# Distributed under the GNU GPL v3. For full terms see the file LICENSE.
# **
# Description: Simulate a linux N node cluster with cps, Pacemaker, Corosync and Ansible in Centos 7+, AlmaLinux 8+ or Ubuntu 1804+ lts vms
# **

# NUM_CLUSTER_NODES: Number of Cluster Nodes VM we must create
$NUM_CLUSTER_NODES=3
# I found problem on lxd in parallel mode (lock problem)
ENV["VAGRANT_NO_PARALLEL"]="1"

#$BOX="almalinux/8"
#$BOX="generic/centos7"
#$BOX="ubuntu/bionic64"
$BOX="ubuntu/focal64"

$PRE_NODE_NAME="cl-node-"

node_names=[]
Vagrant.configure("2") do |config|
  (1..$NUM_CLUSTER_NODES).each do |i|
    node_names.push("#{$PRE_NODE_NAME}#{i}")
    config.vm.define "#{$PRE_NODE_NAME}#{i}" do |node|
      node.vm.box = $BOX
      # Cluster private net for Carousync
      node.vm.network "private_network", ip: "192.168.50.#{i +1}", auto_config: false
      # private net for fencing (pcs stonith)
      node.vm.network "private_network", ip: "192.168.51.#{i +1}", auto_config: false
      node.vm.synced_folder ".", "/vagrant", disabled: true
      node.vm.provider "virtualbox" do |vb, override|
        vb.name = "#{$PRE_NODE_NAME}#{i}"
        vb.memory = "1024"
        vb.customize ["modifyvm", :id, "--groups", "/cluster"]
        vb.linked_clone = true if Gem::Version.new(Vagrant::VERSION) >= Gem::Version.new('1.8.0')
      end
      if i == $NUM_CLUSTER_NODES
        node.vm.provision :ansible do |ansible|
          # Disable default limit to connect to all the machines
          ansible.limit = "all"
          #ansible.ask_vault_pass = true
          ansible.playbook = "provision/playbook.yml"
          ansible.groups = {
            "controller" => ["#{$PRE_NODE_NAME}#{1}"],
            "cluster_nodes" => node_names,
          }
          #ansible.tags = 'fence'
        end
      end
      node.trigger.before :destroy do |trigger|
        trigger.warn = "Remove this node to cluster members before destroy it"
        #trigger.on_error = :halt
        trigger.on_error = :continue
        trigger.run = {
	  inline: "ansible-playbook --inventory-file=.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory -e \"del_node=#{$PRE_NODE_NAME}#{i}\" --tags=del_node provision/playbook.yml"
          }
      end
    end
  end
end

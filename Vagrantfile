# -*- mode: ruby -*-
# vi: set ft=ruby :
$script = <<SCRIPT

yum update -y
yum install -y epel-release
yum install -y centos-release-openstack-rocky
yum-config-manager --enable openstack-rocky
yum update -y
yum install -y openstack-packstack
yum install -y vim-X11

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
setenforce 0

systemctl disable firewalld
systemctl stop firewalld
systemctl disable NetworkManager
systemctl stop NetworkManager
systemctl enable network
systemctl start network
systemctl restart sshd

echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "LANG=en_US.utf-8"  > /etc/environment
echo "LC_ALL=en_US.utf-8" > /etc/environment
SCRIPT

require 'yaml'

vagrant_config = YAML.load_file("provisioning/openstack.conf.yaml")
Vagrant.configure("2") do |config|
	config.vm.box = "centos/7"
  config.vm.provision :shell, :inline => "echo root:password  | chpasswd"
  config.vm.provision :shell, :inline => $script
	config.hostmanager.enabled = true
	config.hostmanager.manage_host = true
	config.hostmanager.manage_guest = true
	config.hostmanager.ignore_private_ip = false
	config.hostmanager.include_offline = true

	config.vm.define "ops_controller" do |ops_controller|
		ops_controller.vm.host_name = vagrant_config['ops_controller']['host_name']
		ops_controller.vm.network "private_network", ip: vagrant_config['ops_controller']['ip']
		ops_controller.vm.provider "virtualbox" do |vb|
			vb.memory = vagrant_config['ops_controller']['memory']
			vb.cpus = vagrant_config['ops_controller']['cpus']
		end
	end

	(1..2).each do |i|
		config.vm.define "ops_compute#{i}" do |ops_compute|
			ops_compute.vm.host_name = vagrant_config["ops_compute#{i}"]['host_name']
			ops_compute.vm.network "private_network", ip: vagrant_config["ops_compute#{i}"]['ip']
			ops_compute.vm.provider "virtualbox" do |vb|
				vb.memory = vagrant_config["ops_compute#{i}"]['memory']
				vb.cpus = vagrant_config["ops_compute#{i}"]['cpus']
			end

		end
	end
end

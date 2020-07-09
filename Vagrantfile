# This defines the version of vagrant
BOX_IMAGE="bento/opensuse-leap-15"
BOX_VERSION="202006.17.0"
NODE_COUNT=1
PUBLIC_INTERFACE="eno1"

Vagrant.configure("2") do |config|
        config.vm.define "controller" do |subconfig|
        	subconfig.vm.box=BOX_IMAGE
		subconfig.vm.box_version=BOX_VERSION
                subconfig.vm.hostname = "controller"
		subconfig.vm.network "private_network", ip:"10.1.0.100", netmask:"24"
		#subconfig.vm.network :hostonly, ip:"10.1.0.10", netmask:"24"
		subconfig.vm.network  "public_network", bridge: PUBLIC_INTERFACE
		subconfig.vm.provider "virtualbox" do |v|
        		v.memory = 8192
        		v.cpus = 2
			v.customize ['modifyvm',:id, '--nested-hw-virt','on']
    		end
		subconfig.vm.provision "shell", run:"once", inline: <<-SHELL
		cat /etc/sysctl.conf
                rm /etc/sysctl.conf
		echo "net.ipv6.conf.all.disable_ipv6=0"       >  /etc/sysctl.conf
		echo "net.ipv6.conf.default.disable_ipv6=0"   >>  /etc/sysctl.conf 
		echo "net.ipv6.conf.lo.disable_ipv6=0"        >>  /etc/sysctl.conf
		apt-get   install  -y  git vim curl
                useradd -s /bin/bash -d /opt/stack -m stack
		echo "stack ALL=(ALL) NOPASSWD: ALL" | tee /etc/sudoers.d/stack
                runuser -l stack  -c 'git clone  --single-branch --branch stable/train  https://opendev.org/openstack/devstack'
		SHELL
                subconfig.vm.provision "file", source: "./local.conf.controller", destination: "~/local.conf"
                subconfig.vm.provision "file", source: "./is_suse_devstack.patch", destination: "~/is_suse_devstack.patch"

		subconfig.vm.provision "shell", inline: <<-SHELL
		runuser -l stack -c "cp /home/vagrant/local.conf /opt/stack/devstack/"
		runuser -l stack -c "cp /home/vagrant/is_suse_devstack.patch /opt/stack/devstack/"
		runuser -l stack -c "cd /opt/stack/devstack/; git apply is_suse_devstack.patch"
		echo "Rebooting for changes to take effect"
		reboot
		SHELL
	end
	(1..NODE_COUNT).each do |i|
 		config.vm.define "compute#{i}" do |subconfig|
			subconfig.vm.box = BOX_IMAGE
        		subconfig.vm.box_version=BOX_VERSION
 			subconfig.vm.hostname = "compute#{i}"
 			subconfig.vm.network :private_network, ip: "10.1.0.#{i + 100}"
			subconfig.vm.network     :public_network, bridge: PUBLIC_INTERFACE
			subconfig.vm.provider "virtualbox" do |v|
        			v.memory = 1024
        			v.cpus = 1
    			end
			subconfig.vm.provision "shell", run:"once", inline: <<-SHELL
                	cat /etc/sysctl.conf
			rm /etc/sysctl.conf
			echo "net.ipv6.conf.all.disable_ipv6=0"       >  /etc/sysctl.conf
			echo "net.ipv6.conf.default.disable_ipv6=0"   >>  /etc/sysctl.conf 
			echo "net.ipv6.conf.lo.disable_ipv6=0"        >>  /etc/sysctl.conf
			apt-get   install  -y  git vim curl
			sudo zypper -n install patterns-openSUSE-kvm_server
                	useradd -s /bin/bash -d /opt/stack -m stack
			echo "stack ALL=(ALL) NOPASSWD: ALL" | tee /etc/sudoers.d/stack
                	runuser -l stack  -c 'git clone  --single-branch --branch stable/train  https://opendev.org/openstack/devstack'
			SHELL
                	subconfig.vm.provision "file", source: "./local.conf.compute", destination: "~/local.conf"
                	subconfig.vm.provision "file", source: "./is_suse_devstack.patch", destination: "~/is_suse_devstack.patch"

			subconfig.vm.provision "shell", inline: <<-SHELL
			runuser -l stack -c "cp /home/vagrant/local.conf /opt/stack/devstack/"
			runuser -l stack -c "cp /home/vagrant/is_suse_devstack.patch /opt/stack/devstack/"
			runuser -l stack -c "cd /opt/stack/devstack/; git apply is_suse_devstack.patch"
			echo "Rebooting for changes to take effect"
			reboot
			SHELL


    		end
  	end
end	

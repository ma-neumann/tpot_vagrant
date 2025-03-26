# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.box = "ubuntu/jammy64"
	config.vm.hostname = "tpot.box"
	
	# Make enough room for TPot on disk
	config.disksize.size = '128GB'

	# Add network adapter to interact with TPot
	config.vm.network "private_network", ip:"192.168.57.10", name: "vboxnet1"

	# Wait for the VM to boot up for 10mins max.
	config.vm.boot_timeout = 600

	# Any SSH connections made will enable agent forwarding
	config.ssh.forward_agent = true

	# Generate keypair and push into VM
	config.ssh.insert_key = true

	# Customizations of VirtualBox VM
	config.vm.provider :virtualbox do |vb|
		# Hide the VirtualBox GUI when booting the machine
		vb.gui = false

		# VirtualBox name
		vb.name = "TPot"

		# Set OS type
		vb.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]

		# Main memory 8GiB, 2 CPUs
		vb.customize ["modifyvm", :id, "--memory", "8192", "--cpus", "2"]
	end

	# Remove ubuntu user
	config.vm.provision :shell, :inline => "userdel ubuntu || true"

	# Upgrade OS
	config.vm.provision :shell, :inline => "apt-get update -y"
	config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get -o Dpkg::Options::=\"--force-confdef\" -o Dpkg::Options::=\"--force-confold\" dist-upgrade -y"

	# Install VirtualBox guest additions
	config.vm.provision :shell, :inline => "apt-get install -y virtualbox-guest-utils"

	# Install tpot
	config.vm.provision :shell, inline: <<-SHELL
		su vagrant <<-EOF
			set -e
			git clone https://github.com/ma-neumann/tpotce.git && cd tpotce
			myQST=y myTPOT_TYPE=i myWEB_USER=vagrant myWEB_PW=vagrant ./install.sh
		EOF
	SHELL

	# Enable password-based login to SSH
	config.vm.provision :shell, inline: <<-SHELL
		sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config.d/*
	SHELL
	
	# Cleanup
	config.vm.provision :shell, inline: <<-SHELL
		set -e
		apt-get autoremove -y
		apt-get clean -y
		apt-get autoclean -y
	SHELL

	# Reduce swappiness
	config.vm.provision :shell, :inline => "echo 'vm.swappiness = 90' >> /etc/sysctl.conf"

	# Reboot
	config.vm.provision :shell, :inline => "reboot"
end

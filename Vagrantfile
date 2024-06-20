Vagrant.configure(2) do |config|

	# before you must install these plugins to speed up vagrant provisionning
  # vagrant plugin install vagrant-faster
  # vagrant plugin install vagrant-cachier

  config.cache.auto_detect = true
	# Set some variables
  etcHosts = ""

	# some settings for common server (not for haproxy)
  common = <<-SHELL
  sudo apt update -qq 2>&1 >/dev/null
  sudo apt install -y -qq unzip iftop curl software-properties-common git vim tree net-tools telnet git 2>&1 >/dev/null
  sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
  sudo systemctl restart sshd
  SHELL

  config.vm.box = "bento/ubuntu-22.04"
  config.vm.box_url = "bento/ubuntu-22.04"

  
	# set servers list and their parameters
	NODES = [
  	{ :hostname => "k8s-master", :ip => "192.168.100.11", :cpus => 4, :mem => 2048, :type => "kube_master" },
  	{ :hostname => "k8s-worker1", :ip => "192.168.100.21", :cpus => 2, :mem => 2048, :type => "kube_worker" },
  	{ :hostname => "k8s-worker2", :ip => "192.168.100.12", :cpus => 2, :mem => 2048, :type => "kube_worker" }
  ]

	# define /etc/hosts for all servers
  NODES.each do |node|
			etcHosts += "echo '" + node[:ip] + "   " + node[:hostname] + "'>> /etc/hosts" + "\n"
  end #end NODES

	# run installation
  NODES.each do |node|
    config.vm.define node[:hostname] do |cfg|
			cfg.vm.hostname = node[:hostname]
      cfg.vm.network "private_network", ip: node[:ip]
      cfg.vm.provider "virtualbox" do |v|
				v.customize [ "modifyvm", :id, "--cpus", node[:cpus] ]
        v.customize [ "modifyvm", :id, "--memory", node[:mem] ]
        #v.customize [ "modifyvm", :id, "--natdnshostresolver1", "on" ]
        #v.customize [ "modifyvm", :id, "--natdnsproxy1", "on" ]
        v.customize [ "modifyvm", :id, "--name", node[:hostname] ]
				v.customize [ "modifyvm", :id, "--ioapic", "on" ]
        v.customize [ "modifyvm", :id, "--nictype1", "virtio" ]
      end #end provider
			
			#for all
      cfg.vm.provision :shell, :inline => etcHosts
			cfg.vm.provision :shell, :inline => common
      config.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        s.inline = <<-SHELL
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        SHELL
      end

    end # end config
  end # end nodes
end 

Vagrant.configure("2") do |config|
  config.vm.box = "centos7"
  config.vm.provider "virtualbox" do |v|
	v.memory = 256
	v.cpus = 1
  end
  config.vm.define "nfss" do |nfss|
	nfss.vm.network "private_network", ip: "192.168.50.10",
    virtualbox__intnet: "net1"
	nfss.vm.hostname = "nfss"
	nfss.vm.provision "shell", path: "nfss_script.sh"

  end
  config.vm.define "nfsc" do |nfsc|
	nfsc.vm.network "private_network", ip: "192.168.50.11",
    virtualbox__intnet: "net1"
	nfsc.vm.hostname = "nfsc"
	nfsc.vm.provision "shell", path: "nfsc_script.sh"
end
end

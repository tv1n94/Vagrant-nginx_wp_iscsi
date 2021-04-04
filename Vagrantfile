Vagrant.configure(2) do |config|

  N = 2
  (1..N).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "centos/7"
      node.vm.synced_folder ".", "/vagrant", disabled: true
      node.vm.hostname = "node#{i}"
      node.vm.network "private_network", ip:"192.168.2.1#{i}"
      node.vm.network "private_network", ip:"192.168.3.1#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.name = "node#{i}"
      end
    end
  end
  
    config.vm.define "iscsi" do |node|
      node.vm.box = "centos/7"
      node.vm.synced_folder ".", "/vagrant", disabled: true
      node.vm.hostname = "iscsi"
      node.vm.network "private_network", ip:"192.168.2.10"
      node.vm.network "private_network", ip:"192.168.3.10"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.name = "iscsi"
      end
    end

    config.vm.define "db1" do |node|
      node.vm.box = "centos/7"
      node.vm.synced_folder ".", "/vagrant", disabled: true
      node.vm.hostname = "db1"
      node.vm.network "private_network", ip:"192.168.2.21"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.name = "db1"
      end
    end

end
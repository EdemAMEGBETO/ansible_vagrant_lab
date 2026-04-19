Vagrant.configure("2") do |config|
  nodes = {
    "web" => "192.168.56.10",
    "db"  => "192.168.56.11",
    "lb"  => "192.168.56.12"
  }

  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_version = "202502.21.0"

  nodes.each do |hostname, ip|
    config.vm.define hostname do |node|
      node.vm.hostname = hostname
      node.vm.network "private_network", ip: ip
      node.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 1
      end
    end
  end
end
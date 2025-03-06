Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.cpus = 1
  end

  config.vm.define "node1" do |node|
    node.vm.hostname = "node1"
    node.vm.network "private_network", ip: "192.168.111.101"
    node.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y keepalived nginx netcat
      echo "Hello from node1" > /var/www/html/index.html
      systemctl enable nginx
      systemctl restart nginx
    SHELL
  end

  config.vm.define "node2" do |node|
    node.vm.hostname = "node2"
    node.vm.network "private_network", ip: "192.168.111.102"
    node.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y keepalived nginx netcat
      echo "Hello from node2" > /var/www/html/index.html
      systemctl enable nginx
      systemctl restart nginx
    SHELL
  end
end

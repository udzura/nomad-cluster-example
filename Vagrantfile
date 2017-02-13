# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT
which nomad 2>&1 >/dev/null && exit

# Update apt and get dependencies
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y unzip curl wget vim

# Download Nomad
echo Fetching Nomad...
cd /tmp/
curl -sSL https://releases.hashicorp.com/nomad/0.5.4/nomad_0.5.4_linux_amd64.zip -o nomad.zip

echo Installing Nomad...
unzip nomad.zip
sudo chmod +x nomad
sudo mv nomad /usr/bin/nomad

sudo mkdir -p /etc/nomad.d
sudo chmod a+w /etc/nomad.d

# Set hostname's IP to made advertisement Just Work
sudo sed -i -e "s/.*nomad.*/$(ip route get 1 | awk '{print $NF;exit}') nomad/" /etc/hosts

SCRIPT

$update_hosts = <<SCRIPT
grep -q nomad0.local /etc/hosts && exit

echo "172.17.8.100 nomad0.local" | sudo tee -a /etc/hosts
echo "172.17.8.101 nomad1.local" | sudo tee -a /etc/hosts
echo "172.17.8.102 nomad2.local" | sudo tee -a /etc/hosts
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64" # 16.04 LTS
  config.vm.hostname = "nomad"
  config.vm.provision "shell", inline: $script, privileged: false
  config.vm.provision "shell", inline: $update_hosts, privileged: false
  config.vm.provision "docker" # Just install it

  # Increase memory for Virtualbox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  3.times do |i|
    config.vm.define "nomad#{i}" do |c|
      ip = "172.17.8.#{i+100}"
      config.vm.network :private_network, ip: ip
    end
  end
end

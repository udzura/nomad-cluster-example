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

SCRIPT

$update_hosts = <<SCRIPT
grep -q nomad0.local /etc/hosts && exit

echo "172.17.8.100 nomad0.local" | sudo tee -a /etc/hosts
echo "172.17.8.101 nomad1.local" | sudo tee -a /etc/hosts
echo "172.17.8.102 nomad2.local" | sudo tee -a /etc/hosts
SCRIPT

$add_hcl = <<SCRIPT
cat <<EOC | sudo tee /etc/nomad.hcl >/dev/null
# Increase log verbosity
log_level = "DEBUG"

# Setup data dir
data_dir = "/tmp/server"

# Enable the server
server {
    enabled = true

    # Self-elect, should be 3 or 5 for production
    bootstrap_expect = 3
}
EOC
SCRIPT

$add_cli_hcl = <<SCRIPT
cat <<EOC | sudo tee /etc/nomad-client.hcl >/dev/null
# Increase log verbosity
log_level = "DEBUG"

# Setup data dir
data_dir = "/tmp/client"

# Enable the client
client {
    enabled = true

    # For demo assume we are talking to server1. For production,
    # this should be like "nomad.service.consul:4647" and a system
    # like Consul used for service discovery.
    servers = ["127.0.0.1:4647"]
}

# Modify our port to avoid a collision with server1
ports {
    http = 5656
}
EOC
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64" # 16.04 LTS
  config.vm.hostname = "nomad"
  config.vm.provision "shell", inline: $script, privileged: false
  config.vm.provision "shell", inline: $update_hosts, privileged: false
  config.vm.provision "shell", inline: $add_hcl, privileged: false
  config.vm.provision "shell", inline: $add_cli_hcl, privileged: false
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

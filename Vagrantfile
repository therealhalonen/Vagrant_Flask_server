# -*- mode: ruby -*-
# vi: set ft=ruby :

# Initial setup and installation of needed things
$initial = <<INITIAL
apt-get update && apt-get dist-upgrade
apt-get install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools -y
apt-get install python3-venv -y
apt-get install nginx -y
mkdir /home/vagrant/stix_project
cd /home/vagrant/stix_project
mkdir /home/vagrant/stix_project/data
python3 -m venv stixenv
source stixenv/bin/activate
pip install wheel
pip install gunicorn flask
deactivate
INITIAL

# Moving the provisioned files to theyr own places and fixing the permissions
$start = <<START
mv /tmp/files/wsgi.py /home/vagrant/stix_project/wsgi.py
mv /tmp/files/webhook.py /home/vagrant/stix_project/stix.py
mv /tmp/files/stix.service /etc/systemd/system/stix.service
echo "Hello World" > /home/vagrant/stix_project/data/Hello
chown -R vagrant:vagrant /home/vagrant/stix_project/
systemctl start stix
systemctl enable stix
START

# Configuring the nginx proxy
$nginx = <<NGINX
mv /tmp/files/stix /etc/nginx/sites-available/stix
sudo ln -s /etc/nginx/sites-available/stix /etc/nginx/sites-enabled
systemctl restart nginx
NGINX


Vagrant.configure(2) do |config|
  config.vm.define "Debian - Stix Server" do |db|
  config.vm.box = "debian/bookworm64"
    db.vm.synced_folder '.', '/vagrant', disabled: true
    db.vm.hostname = "stixsrv"
    # Configure the network adapters here, according to you needs.
    # Also remember to edit the files/stix according to these IP configurations
    config.vm.network "private_network", ip: "192.168.56.66",
      name: "vboxnet0"
    config.vm.network "private_network", ip: "192.168.66.66",
      virtualbox__intnet: "pentestNet"
      
    db.vm.provider :virtualbox do |vb|
      vb.name = "Debian - Stix Server"
      vb.memory = 256
      vb.cpus = 1
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
	  vb.customize ["modifyvm", :id, "--vram", "12"]
	  # vb.customize ['modifyvm', :id, '--firmware', 'efi']
	  vb.gui = false
	end
  # Intial installations
  config.vm.provision "initial", type: "shell", inline: $initial
  # Start Stix
  config.vm.provision "start", type: "shell", inline: $start
  # Setup nginx proxy
  config.vm.provision "nginx", type: "shell", inline: $nginx
  # Copy local files to remote tmp
  config.vm.provision "files", before: :all, type: "file", source: "files", destination: "/tmp/files"
  end
end

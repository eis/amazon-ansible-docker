# -*- mode: ruby -*-
# vi: set ft=ruby :

# Plugin installation procedure from http://stackoverflow.com/a/28801317
required_plugins = %w(vagrant-triggers)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

keyfile = 'amazon-testpair.pem'

$script_to_install_ansible = <<SCRIPT
echo Installing Ansible...
sudo apt-get -y install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get -y install ansible

echo Configuring Ansible...
cp /vagrant/ansible/ansible.cfg /home/vagrant/.ansible.cfg
cp /vagrant/ansible/inventory /home/vagrant/inventory
cp /vagrant/amazon-testpair.pem /home/vagrant/.ssh/
chmod 600 /home/vagrant/inventory && sudo chown vagrant:vagrant /home/vagrant/inventory
chmod 600 /home/vagrant/.ansible.cfg && sudo chown vagrant:vagrant /home/vagrant/.ansible.cfg
chmod 700 /home/vagrant/.ssh/amazon-testpair.pem && sudo chown vagrant:vagrant /home/vagrant/.ssh/amazon-testpair.pem
SCRIPT

$script_to_build_docker_image = <<SCRIPT
cd /vagrant/node-app
docker build -t eis/node-web-app .
SCRIPT


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version. Please don't change it unless
# you know what you're doing.
Vagrant.configure(2) do |config|

  # Ubuntu 14.04
  config.vm.box = "ubuntu/trusty64"

  # bail out on 'up' if we don't have amazon key
  config.trigger.before :up do
    unless File.exist?(keyfile)
      STDERR.puts "ERROR: The Amazon keyfile must be placed in this directory with name #{keyfile}."
      abort
    end
  end

  config.vm.provision "shell", privileged: true, :path => 'install-docker.sh'
  config.vm.provision "shell", inline: $script_to_install_ansible
  config.vm.provision "shell", inline: $script_to_build_docker_image
end

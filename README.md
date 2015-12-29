amazon-ansible-docker
=====================

Simple example to show Docker container usage in Amazon
using Ansible.

Uses Vagrant box, having Ubuntu, locally. This local instance
can be then used to issue Ansible commands.

Assumes amazon-testpair.pem in the local folder.

Install Docker to remote server
--------------------------------

vagrant ssh
ansible all -m ping
ansible-galaxy install angstwad.docker_ubuntu
ansible-playbook /vagrant/ansible/docker.yml

Run Docker image locally
------------------------
vagrant ssh
docker run -p 49160:8080 -d eis/node-web-app
curl -i localhost:49160
# You should see "Hello world"

Push local Docker image to Amazon
---------------------------------

vagrant ssh
docker save eis/node-web-app > /tmp/node-image.tar
# 700M, up to 30 minutes!
scp -i ~/.ssh/amazon-testpair.pem /tmp/node-image.tar ubuntu@52.32.162.69:
ssh -i ~/.ssh/amazon-testpair.pem ubuntu@52.32.162.69
sudo usermod -aG docker ubuntu # if not already done - solves connectivity thing
docker load < node-image.tar
docker images
docker run -p 49160:8080 -d eis/node-web-app

Build local Docker image in Amazon
----------------------------------
# This method is to avoid slow transfer
# Building takes about 2 minutes

vagrant ssh
scp -i ~/.ssh/amazon-testpair.pem /vagrant/node-app/* ubuntu@52.32.162.69:
ssh -i ~/.ssh/amazon-testpair.pem ubuntu@52.32.162.69
sudo usermod -aG docker ubuntu # if not already done - solves connectivity thing
docker build -t eis/node-web-app .
docker run -p 49160:8080 -d eis/node-web-app

curl localhost:49160

# Test below assumes security group allows the port to inbound traffic (TCP)
curl 52.32.162.69:49160

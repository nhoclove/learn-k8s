# How to setup

## Prerequisites
- [Virtualbox](https://www.virtualbox.org/)
- [Ansible](https://www.ansible.com/)
- [Vagrant](https://www.vagrantup.com/)

## Basic configuration for each VM [docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#before-you-begin):
- Box: bento/ubuntu-16.04
- CPU: 2 cores
- Memory: 1700MB

## 1. Init cluster
`$ vagrant up`
## 2. Connect to the cluster
### Install kubectl
`$ sudo apt-get install -y kubectl`
### Download kubernetes cluster credentials
`$ scp -r vagrant@192.168.14.10:/home/vagrant/.kube $HOME/`
### Show cluster info
`$ kubectl cluster-info`
## 3. Stop cluster
`$ vagrant halt`
## 4. Start cluster
`$ vagrant up`
## 5. Delete cluster
`$ vagrant destroy -f`
## References:
- https://github.com/ctienshi/kubernetes-ansible
- https://github.com/geerlingguy/ansible-for-devops/tree/master/kubernetes
- https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-18-04

## Troubleshoot:
- https://medium.com/@joatmon08/playing-with-kubeadm-in-vagrant-machines-part-2-bac431095706

## TODO
- Fix kubelet IP problem in woker nodes
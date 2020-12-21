# Create a local Kubernetes cluster Using VirtualBox, Flannel, and Vagrant

This repository has Ansible playbooks to create a multi-node Kubernetes cluster, running locally in three VMs. Included is a Vagrantfile, which allows Vagrant to automatically build three VirtualBox VMs and then run the playbooks agains them. The VM's use a Debian 10 image from "tkhinfra/debian10" as the base OS. This is a very minimal Debian 10 image.

## Usage

The Following packages/software will need to be installed before building the Cluster.

  1. [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
  2. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
  3. [Vagrant](https://www.vagrantup.com/downloads.html)

Once they are installed, in the git cloned directory, run:

    $ vagrant up

After a few minutes, the three VMs will be booted, and configured with Kubernetes using the Ansible playbook 'main.yml'.

## Managing Kubernetes

You can log into the master node with the command:

    $ vagrant ssh cluster1

Once you have a shell into cluster1, run `sudo -i` to switch to the root user. You can then manager the cluster using `kubectl`. There is
no default namespace defined so you muse use '--namespace=<ns>' or '--all-namespaces' with `kubectl` commands.

You can also get the kubernetes config file to add to your ~/.kube/config and Manage kubernetes from a local shell

    $ vagrant ssh -c "sudo cat /root/.kube/config" cluster1 > ~/.kube/config
    $ kubectl config set-context --current --namespace=kube-system
    $ kubectl get pods

## The Anatomy of the Ansible Playbook

The inventory file defines the hosts in the kubernetes array with their respective ip addresses and variables

The 'inventory' file...

    [kubenetes]
    cluster1 ansible_host=192.168.2.2 kubernetes_role=master
    cluster2 ansible_host=192.168.2.3 kubernetes_role=node
    cluster3 ansible_host=192.168.2.4 kubernetes_role=node

    [kubernetes:vars]
    ansible_ssh_user=vagrant
    ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
    kubernetes_apiserver_advertise_address=192.168.2.2


The inventory file defines the hosts in the kubernetes array with their respective ip addresses
and roles and the variables to use during a playbook run.  

The 'main.yml' playbook is fairly straight forward. It defines which host block to play on and defines the variable file location.
It then runs a setup for Vagrant and calls each role to be played on the hosts.

The 'main.yml' file...

    - hosts: kubernetes
      become: true

      vars_files:
        - vars/main.yml

      pre_tasks:
        - include_tasks: tasks/vagrant-setup.yml

      roles:
        - fail2ban
        - ssh
        - autoupdate
        - swap
        - docker
        - kubernetes

## Playbook Roles

The 'fail2ban' role will install fail2ban on each host for some tighter security. This allows for any unwanted 'ssh' attempts to be automaticall blocked.  

The 'ssh' role will configure the '/etc/sshd_config' for better security and restart ssh. It also updates '/etc/sudoers' file and adds the vagrant user to be able to 'sudo' without using a password.

The 'autoupdate' role installes the 'unattended-upgrades' package and installs the needed config files to use it.

The 'swap' role will make sure 'swap' is defined in the '/etc/fstab' file and configures swap.

The 'docker' role installs 'Docker' on all the hosts and will setup the docker users and docker compose and restart docker.

The 'kubernetes' role installs kubernetes on all the cluster nodes and defines cluster1 as the master node. It will setup 'Flannel' as the networking overlay. 



# Local Kubernetes Cluster Example Using VirtualBox

This example demonstrates the setup of a single-master, multi-node Kubernetes cluster, running locally in three VMs. You can run the Ansible playbook against any three servers, but this example includes a Vagrantfile, which allows Vagrant to automatically build three VirtualBox VMs and then runs the playbook agains them automatically.

## Usage

Before building the cluster, you need to install the following:

  1. [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
  2. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
  3. [Vagrant](https://www.vagrantup.com/downloads.html)

Then, in this directory, run:

    $ vagrant up

After a few minutes, the three VMs will be booted, and the Ansible playbook will have installed Kubernetes using the `main.yml` playbook.

## Managing Kubernetes

You can log into the master node with the command:

    $ vagrant ssh kube1

From there, run `sudo su` to switch to the root user, and then you can use `kubectl` to manage the Kubernetes cluster.

You can also get the kubernetes config file to add to your ~/.kube/config and Manage kubernetes from a local shell

    $ vagrant ssh -c "sudo cat /root/.kube/config" cluster1 > ~/.kube/config
    $ kubectl config set-context --current --namespace=kube-system
    $ kubectl get pods

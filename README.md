# k8s-cluster-setup

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

This repository contains [Ansible](https://www.ansible.com/) playbooks to set up a [Kubernetes](https://kubernetes.io) cluster using kubeadm. These playbooks have been tested on Debian 9 and 10 (Stretch and Buster). You can use them as a basis and adapt them to your needs.

## **Step 1**: Test the Kubernetes Configuration

You should test the configuration with the [Vagrant](https://www.vagrantup.com/) virtual machines before applying it to the Kubernetes cluster.

### Prerequisites

The following packages should be installed on your machine:

- [Vagrant](https://www.vagrantup.com/docs/installation),
- [Oracle Virtualbox](https://www.virtualbox.org/wiki/Downloads),
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html).

### Create the Virtual Machines

```sh
# Create a control-plane and worker nodes.
vagrant up

# To execute a specific playbook, use the PLAYBOOK environment variable.
PLAYBOOK=<my_playbook>.yaml vagrant provision
```

## **Step 2**: Deploy the Kubernetes Cluster

### Prerequisites

- **Complete the previous steps**.
- [Ansible shoud be installed on your machine](https://docs.ansible.com/ansible/latest/installation_guide/index.html).
- SSH access to Kubernetes nodes (control-plane and workers).

### Create a Non-Root User

Create the `kube` user and add it to the "sudoers":

```sh
useradd -m -s /bin/bash kube
echo "kube ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

### Install and Configure NFS

```sh
# Install Kubernetes and configure NFS
ansible-playbook 00-nfs.yaml
```

### Install and Configure Kubernetes

```sh
# Install Kubernetes and configure the cluster
ansible-playbook 10-k8s.yaml
```

This playbook:

- Prepares hosts (control-plane and workers) to run the kubelet daemon [according to Kubernetes's best-practices](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).
- Installs kubeadm, kubectl and kubelet packages.
- Installs [Flannel](https://github.com/coreos/flannel).
- Creates the cluster (the control-plane and the workers) with kubeadm.

During the playbook execution, you should watch:

- The file `/var/log/init.log` on the control-plane node.
- The files `/var/log/join.log` on worker nodes.

## Troubleshooting

- On a new node, you will maybe have to reboot the host server. Otherwise, the network may not work properly. Follow these instructions to safely restart a node.

  - Mark the node as unschedulable to prevent new pods from arriving:

  ```sh
  kubectl drain node_name --ignore-daemonsets
  ```

  - Reboot the server and check services status:

  ```sh
  systemctl status kubelet.service
  ```

  - Mark the node as schedulable again:

  ```sh
  kubectl uncordon node_name
  ```

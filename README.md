# Kubernetes-HA-Ansible

This repository contains Ansible playbooks and roles to automate the deployment of a High-Availability (HA) Kubernetes cluster. It sets up a multi-master, multi-worker cluster using `kubeadm`, with `HAProxy` for load balancing the control plane and `CRI-O` as the container runtime. Think of it as a robust toolkit for getting a production-ready Kubernetes setup running from scratch.

## Repository Diagram

![Cluster Diagram](img/k8s.svg)

## Getting Started

Before diving in, make sure you've got your environment prepped.

### Prerequisites

- Ansible 2.12 or higher on your control machine.
- Python 3.x on your control machine.
- SSH access (passwordless preferred) to all nodes from the control machine.
- Root or sudo privileges on all nodes.
- All nodes should be running a compatible OS (tested on Ubuntu 22.04+).
- Internet access on all nodes for package and image downloads.

First, set up SSH key-based authentication from your control machine to all cluster nodes. This makes the Ansible run smooth and secure.

1.  Generate an SSH key on your control machine if you don't have one:
    ```bash
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```

2.  Copy the public key to all nodes defined in your inventory. The `ssh-copy-id` utility is perfect for this:
    ```bash
    ssh-copy-id root@<node-ip>
    ```
    Repeat this for every single node (`haproxy`, `masters`, and `workers`).

### Inventory Setup

The inventory is the heart of the deployment. Edit `inventory/hosts.ini` to match your server IPs and hostnames. Here's a sample based on a 3-master, 3-worker setup:

```ini
[haproxy]
haproxy ansible_host=10.10.10.10

[kube_masters]
master1 ansible_host=10.10.10.11
master2 ansible_host=10.10.10.12
master3 ansible_host=10.10.10.13

[kube_workers]
worker1 ansible_host=10.10.10.14
worker2 ansible_host=10.10.10.15
worker3 ansible_host=10.10.10.16

[kubernetes:children]
kube_masters
kube_workers
Next, customize global settings in group_vars/all.yml. This is where you define your cluster's domain, API port, and network CIDR.

Installation
The main playbook playbooks/k8s-full-cluster.yml orchestrates the entire deployment in the correct order.

To run the full deployment from start to finish, execute this single command from the root of the project:

bash
ansible-playbook -i inventory/hosts.ini playbooks/k8s-full-cluster.yml
This playbook will:

Configure prerequisite settings on all nodes.

Set up HAProxy for control plane load balancing.

Install CRI-O, kubeadm, kubelet, and kubectl on all Kubernetes nodes.

Initialize the first master node (kube_masters[0]).

Install the Calico CNI (Container Network Interface) for cluster networking.

Join the remaining master nodes to the control plane.

Join the worker nodes to the cluster.

Distribute the admin.conf to all master nodes so kubectl works everywhere.

Roles Overview
Here's a quick rundown of the roles:

Role	Description
sni-config	Applies common SNI configuration (if needed).
docker	Installs Docker, used here for running the HAProxy container.
haproxy	Deploys and configures HAProxy as a load balancer for the Kubernetes API server.
k8s-common	Installs all common dependencies: CRI-O, kubeadm, kubelet, kubectl, etc.
k8s-control-plane-init	Initializes the very first master node and generates join tokens.
k8s-cni-calico	Deploys the Calico CNI to enable cluster networking.
k8s-control-plane-join	Joins additional master nodes to form an HA control plane.
k8s-worker-join	Joins worker nodes to the cluster.
k8s-kubeconfig	Distributes the admin kubeconfig to all control plane nodes.
Repository Structure
.
├── README.md
├── ansible.cfg
├── group_vars
│   └── all.yml
├── img
│   └── k8s.svg
├── inventory
│   └── hosts.ini
├── playbooks
│   ├── haproxy.yml
│   ├── k8s-common.yml
│   └── k8s-full-cluster.yml
└── roles
    ├── docker/
    ├── haproxy/
    ├── k8s-cni-calico/
    ├── k8s-common/
    ├── k8s-control-plane-init/
    ├── k8s-control-plane-join/
    ├── k8s-kubeconfig/
    ├── k8s-worker-join/
    └── sni-config/

---

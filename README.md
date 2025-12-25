# Kubernetes HA Cluster with Ansible

This repository provides Ansible playbooks and roles to automate the deployment of a **High-Availability Kubernetes cluster** using `kubeadm`.

The setup includes:
- Multi-master (HA) control plane
- Multiple worker nodes
- HAProxy as a load balancer for the Kubernetes API
- CRI-O as the container runtime
- Calico CNI for cluster networking

The goal is to deliver a **production-ready Kubernetes cluster** from scratch with minimal manual effort.


## Architecture Overview

   ![Kubernetes HA Diagram](img/k8s.svg)

## Prerequisites

Make sure your environment meets the following requirements:

- Ansible **2.12+** installed on the control machine
- Python **3.x** on the control machine
- Passwordless SSH access to all nodes
- Root or sudo privileges on all nodes
- Supported OS on all nodes (tested on **Ubuntu 22.04+**)
- Internet access for package and image downloads


## SSH Configuration

Set up SSH key-based authentication from the control machine to **all cluster nodes**.

1. Generate an SSH key if you don’t already have one:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
````

2. Copy the public key to every node:

```bash
ssh-copy-id root@<node-ip>
```

Repeat this step for:

* HAProxy node
* All master nodes
* All worker nodes

## Inventory Configuration

Edit `inventory/hosts.ini` to match your environment.

### Example: 3 Masters + 3 Workers

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
```

---

## Global Variables

Customize cluster-wide settings in:

```text
group_vars/all.yml
```

Common configurations include:

* Cluster domain
* Kubernetes API port
* Pod network CIDR
* Kubernetes version

---

## Installation

The main orchestration playbook is:

```text
playbooks/k8s-full-cluster.yml
```

Run the full deployment with:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/k8s-full-cluster.yml
```

### What This Playbook Does

1. Configures prerequisite system settings on all nodes
2. Deploys HAProxy for Kubernetes API load balancing
3. Installs CRI-O, kubeadm, kubelet, and kubectl
4. Initializes the first control plane node
5. Deploys Calico CNI
6. Joins additional master nodes
7. Joins worker nodes
8. Distributes `admin.conf` to all master nodes

---

## Roles Overview

| Role Name                | Description                                     |
| ------------------------ | ----------------------------------------------- |
| `sni-config`             | Applies common SNI configuration (optional)     |
| `docker`                 | Installs Docker (used for running HAProxy)      |
| `haproxy`                | Deploys HAProxy as Kubernetes API load balancer |
| `k8s-common`             | Installs CRI-O, kubeadm, kubelet, kubectl       |
| `k8s-control-plane-init` | Initializes the first master node               |
| `k8s-cni-calico`         | Deploys Calico CNI                              |
| `k8s-control-plane-join` | Joins additional master nodes                   |
| `k8s-worker-join`        | Joins worker nodes                              |
| `k8s-kubeconfig`         | Distributes admin kubeconfig                    |

---

## Repository Structure

```text
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
```

---

## Notes

* This project assumes **fresh nodes** with no existing Kubernetes installation.
* Re-running the playbook on a partially configured cluster may require cleanup.
* Always review variables before deployment in production.

---


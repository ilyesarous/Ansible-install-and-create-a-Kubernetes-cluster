# 🚀 Kubernetes Cluster Setup Using Ansible

This project automates the installation and configuration of a **Kubernetes cluster** (1 master, multiple workers) using **Ansible**.<br>
It sets up all required components — from networking and container runtime to Kubernetes installation and cluster initialization.

---

## 🧩 Project Overview

This Ansible setup provisions:<br>
- A **Kubernetes master node** (`control-plane`)<br>
- Multiple **worker nodes**<br>
- Common system configuration across all nodes (containerd, kernel modules, etc.)<br>
- Networking configuration (sysctl, kernel modules)<br>
- Automatic cluster initialization and join process

The playbook is modular and divided into roles:<br>
- `k8s_common` → installs and configures Kubernetes + containerd<br>
- `k8s_networking` → prepares system networking<br>
- `k8s_master` → initializes the control plane<br>
- `k8s_workers` → joins worker nodes to the cluster

---

## 📁 Repository Structure

├── inventory.ini <br>
├── playbook.yml<br>
└── roles/<br>
├── k8s_common/<br>
│ └── tasks/main.yml<br>
├── k8s_networking/<br>
│ └── tasks/main.yml<br>
├── k8s_master/<br>
│ └── tasks/main.yml<br>
└── k8s_workers/<br>
└── tasks/main.yml<br>

---

## ⚙️ Prerequisites

Before running the playbook, ensure that:<br>

1. You have **Ansible** installed on your control machine.<br>
   ```bash
   $ sudo apt update
   $ sudo apt install software-properties-common
   $ sudo add-apt-repository --yes --update ppa:ansible/ansible
   $ sudo apt install ansible
Your managed nodes (VMs or servers) are reachable via SSH (Vagrant boxes are used in this example).<br>

Password-based SSH access is enabled (as per inventory.ini).<br>

You have Python 3 installed on remote nodes.<br>

🧠 Inventory Configuration (inventory.ini)
Define your cluster topology:<br>

```bash
[master]
10.0.1.15 ansible_user=vagrant ansible_python_interpreter=/usr/bin/python3

[workers]
10.0.1.16 ansible_user=vagrant
10.0.1.17 ansible_user=vagrant

[all:vars]
ansible_python_interpreter=/usr/bin/python3

[all]
10.0.1.15 ansible_user=vagrant ansible_ssh_pass=vagrant ansible_become_pass=vagrant
10.0.1.16 ansible_user=vagrant ansible_ssh_pass=vagrant ansible_become_pass=vagrant
10.0.1.17 ansible_user=vagrant ansible_ssh_pass=vagrant ansible_become_pass=vagrant
```
▶️ How to Run
Test your inventory:<br>
```bash
ansible all -i inventory.ini -m ping
```
Run the full cluster setup:<br>
```bash
ansible-playbook -i inventory.ini playbook.yml
```
Run specific roles if needed:<br>

```bash
ansible-playbook -i inventory.ini playbook.yml --tags "master"
ansible-playbook -i inventory.ini playbook.yml --tags "workers"
```
🧱 Roles Description
🔹 k8s_common
Installs essential packages (curl, apt-transport-https, etc.)<br>

Disables swap<br>

Adds Kubernetes repository and installs:<br>

kubelet, kubeadm, kubectl<br>

Installs and configures containerd as the container runtime<br>

Enables SystemdCgroup for containerd<br>

🔹 k8s_networking
Loads required kernel modules (overlay, br_netfilter)<br>

Configures sysctl for Kubernetes networking<br>

Enables IP forwarding<br>

🔹 k8s_master
Initializes the Kubernetes cluster with:<br>

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```
Sets up kubeconfig for the Ansible user<br>

Saves the kubeadm join command for worker nodes<br>

🔹 k8s_workers
Retrieves the join command from the master node<br>

Executes the join command to add workers to the cluster<br>

🌐 Verifying the Cluster
After the playbook completes, SSH into your master node:<br>

```bash
vagrant ssh master
kubectl get nodes
````
You should see all your nodes (master + workers) in Ready state.<br>

🧩 Notes
This setup uses Flannel networking (10.244.0.0/16).<br>
You can apply the CNI plugin manually if needed:<br>

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
````
Make sure to disable swap permanently for all nodes.<br>

Tested on Ubuntu 22.04 (Jammy) VMs with Vagrant.<br>

🧑‍💻 Author
Ilyes Arous<br>
💼 GitHub: @ilyesarous

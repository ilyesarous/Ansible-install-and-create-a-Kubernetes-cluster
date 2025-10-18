# 🚀 Kubernetes Cluster Setup Using Ansible

This project automates the installation and configuration of a **Kubernetes cluster** (1 master, multiple workers) using **Ansible**.  
It sets up all required components — from networking and container runtime to Kubernetes installation and cluster initialization.

---

## 🧩 Project Overview

This Ansible setup provisions:
- A **Kubernetes master node** (`control-plane`)
- Multiple **worker nodes**
- Common system configuration across all nodes (containerd, kernel modules, etc.)
- Networking configuration (sysctl, kernel modules)
- Automatic cluster initialization and join process

The playbook is modular and divided into roles:
- `k8s_common` → installs and configures Kubernetes + containerd  
- `k8s_networking` → prepares system networking  
- `k8s_master` → initializes the control plane  
- `k8s_workers` → joins worker nodes to the cluster  

---

## 📁 Repository Structure

├── inventory.ini<br>
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

yaml
Copy code

---

## ⚙️ Prerequisites

Before running the playbook, ensure that:

1. You have **Ansible** installed on your control machine.
   ```bash
   sudo apt update && sudo apt install ansible -y
Your managed nodes (VMs or servers) are reachable via SSH (Vagrant boxes are used in this example).

Password-based SSH access is enabled (as per inventory.ini).

You have Python 3 installed on remote nodes.

🧠 Inventory Configuration (inventory.ini)
Define your cluster topology:

ini
Copy code
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
▶️ How to Run
Test your inventory:

bash
Copy code
ansible all -i inventory.ini -m ping
Run the full cluster setup:

bash
Copy code
ansible-playbook -i inventory.ini playbook.yml
Run specific roles if needed:

bash
Copy code
ansible-playbook -i inventory.ini playbook.yml --tags "master"
ansible-playbook -i inventory.ini playbook.yml --tags "workers"
🧱 Roles Description
🔹 k8s_common
Installs essential packages (curl, apt-transport-https, etc.)

Disables swap

Adds Kubernetes repository and installs:

kubelet, kubeadm, kubectl

Installs and configures containerd as the container runtime

Enables SystemdCgroup for containerd

🔹 k8s_networking
Loads required kernel modules (overlay, br_netfilter)

Configures sysctl for Kubernetes networking

Enables IP forwarding

🔹 k8s_master
Initializes the Kubernetes cluster with:

bash
Copy code
kubeadm init --pod-network-cidr=10.244.0.0/16
Sets up kubeconfig for the Ansible user

Saves the kubeadm join command for worker nodes

🔹 k8s_workers
Retrieves the join command from the master node

Executes the join command to add workers to the cluster

🌐 Verifying the Cluster
After the playbook completes, SSH into your master node:

bash
Copy code
vagrant ssh master
kubectl get nodes
You should see all your nodes (master + workers) in Ready state.

🧩 Notes
This setup uses Flannel networking (10.244.0.0/16).
You can apply the CNI plugin manually if needed:

bash
Copy code
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
Make sure to disable swap permanently for all nodes.

Tested on Ubuntu 22.04 (Jammy) VMs with Vagrant.

🧑‍💻 Author
Ilyes Arous
💼 GitHub: @ilyesarous

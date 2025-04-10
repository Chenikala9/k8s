#!/bin/bash

# --- System Update ---
sudo yum update -y

# --- Disable SELinux ---
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# --- Disable Swap ---
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# --- Load Kernel Modules ---
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

sudo modprobe br_netfilter

# --- Set sysctl parameters ---
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

# --- Install Container Runtime (containerd) ---
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum install -y containerd

# Create default config and enable systemd cgroup driver
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

# --- Add Kubernetes Repository ---
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
EOF

# --- Install Kubernetes tools ---
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet

# --- START Kubelet (will wait for kubeadm join) ---
sudo systemctl start kubelet

# --- IMPORTANT: Run the kubeadm join command here (REPLACE with actual command) ---
echo "🎯 NOW run the 'kubeadm join ...' command from the master node!"

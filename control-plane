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
sudo systemctl start kubelet

# --- Initialize Kubernetes Control Plane ---
# Replace the pod network CIDR if you're using a different CNI
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# --- Configure kubectl for current user (assumes ec2-user or similar) ---
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# --- Deploy a Pod Network Addon (Calico) ---
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

# --- Print join command for worker nodes ---
echo ""
echo "🎯 Run this on your worker nodes to join the cluster:"
sudo kubeadm token create --print-join-command

# Kubernetes Cluster on EC2s

## Master Node & Worker Nodes

Follow these steps on all nodes to configure the EC2 for Kubernetes, then install kubelet, kubeadm and kubectl.

### System Config

You must disable Swap for kubelet to work properly, set up containerd, and configure system parameters for Kubernetes networking.

```bash
# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Create containerd configuration file
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

# Load modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params for Kubernetes networking
# This ensures that iptables correctly see bridged traffic
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Install containerd from official Docker repositories
sudo yum update -y
sudo yum install -y containerd

# Create configuration directory for containerd
sudo mkdir -p /etc/containerd

# Generate containerd configuration
containerd config default | sudo tee /etc/containerd/config.tom

# Restart containerd to apply the new configuration
sudo systemctl restart containerd

# Verify that containerd is running
sudo systemctl status containerd
```

### Kubernetes Install

Install kubeadm, kubelet and kubectl:

```bash
# Download the Google Cloud public signing key and set up Kubernetes repo
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

# Set SELinux in permissive mode (effectively disabling it)
# This is required to allow containers to access the host filesystem
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# Install kubelet, kubeadm and kubectl
sudo yum install -y kubelet-1.28.2 kubeadm-1.28.2 kubectl-1.28.2 --disableexcludes=kubernetes

# Enable kubelet
sudo systemctl enable --now kubelet
```

## Master Node

### Cluster Configuration

First, create the kube-config.yml file which is needed for initializing the cluster:

```bash
# Create kube-config.yml file
cat <<EOF > kube-config.yml
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.28.2
kind: ClusterConfiguration
networking:
  podSubnet: 192.168.0.0/16
apiServer:
  extraArgs:
   service-node-port-range: 1024-1233
EOF
```

Initialize the cluster and set up kubectl access:

```bash
# Initialize the Kubernetes cluster with kubeadm
sudo kubeadm init --config kube-config.yml --ignore-preflight-errors=all

# Set kubectl access
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Check the cluster nodes (control plane will be in 'Not Ready' state until Calico is applied)
kubectl get nodes

# Apply Calico pod networking configuration
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Check the cluster nodes again (may take a few moments to become available)
kubectl get nodes

# Create a token for worker nodes to join (copy the output to use on worker nodes)
kubeadm token create --print-join-command
```

In the Master (Control Plane) Node, check the cluster status periodically until all nodes are available:

```bash
kubectl get nodes
```

## Worker Node

### Configure a worker node

Repeat all of the "System Config" and "Kubernetes Install" steps from above on each worker node.

Once the installation is complete, run the join command output generated from the `kubeadm token create --print-join-command` (make sure to prefix with sudo):

```bash
# Example (your actual command will be different):
sudo kubeadm join 192.168.1.10:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:1234...
```

Return to the control plane and run the following command until all worker nodes have joined the cluster:

```bash
kubectl get nodes
```

## Deploy a container to the cluster

Copy react-app-pod.yml to the master node and apply:

```bash
# Create the deployment
kubectl create -f react-app-pod.yml

# Verify that the pod is up and running
kubectl get pods -o wide

# Verify that the deployment is complete
kubectl get deployment
```

# common.sh Script Explained

This document provides a detailed explanation of the `common.sh` script, which prepares an Ubuntu system for Kubernetes installation.

## Script Breakdown

### Shebang
```bash
#!/bin/bash
```
- Specifies the script should be executed using the Bash shell.

### Update the System
```bash
sudo apt-get update
sudo apt-get upgrade -y
```
- **`sudo apt-get update`**: Updates the list of available packages and their versions.
- **`sudo apt-get upgrade -y`**: Installs the latest versions of all installed packages.

### Install Required Packages
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```
- Installs essential packages:
  - **`apt-transport-https`**: Enables the use of HTTPS for package downloads.
  - **`ca-certificates`**: Ensures HTTPS downloads are secure by validating certificates.
  - **`curl`**: A command-line tool to transfer data from or to a server.
  - **`software-properties-common`**: Adds and manages software repositories.

### Disable Swap
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
- **`sudo swapoff -a`**: Disables swap memory, which is required for Kubernetes to function properly.
- **`sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab`**: Comments out the swap entry in `/etc/fstab` to prevent swap from re-enabling after a reboot.

### Load kernel modules (required for Kubernetes networking)
```bash
sudo tee /etc/modules-load.d/kubernetes.conf <<EOF
overlay
br_netfilter
nf_conntrack
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
sudo modprobe nf_conntrack
```
- Creates a configuration file (`/etc/modules-load.d/kubernetes.conf`) to ensure required kernel modules are loaded at boot.
- **Modules Loaded**:
  - **`overlay`**: Enables overlay filesystem support.
  - **`br_netfilter`**: Ensures proper handling of network bridge traffic.
  - **`nf_conntrack`**: Enables connection tracking for Kubernetes networking.
- **`sudo modprobe <module>`**: Loads the specified kernel modules immediately.

### Set Kernel Parameters
```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOT
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOT

sudo sysctl --system
```
- Creates a configuration file (`/etc/sysctl.d/kubernetes.conf`) to set kernel parameters required by Kubernetes:
  - **`net.bridge.bridge-nf-call-ip6tables = 1`**: Enables bridge network traffic for IPv6 to pass through iptables.
  - **`net.bridge.bridge-nf-call-iptables = 1`**: Enables bridge network traffic for IPv4 to pass through iptables.
  - **`net.ipv4.ip_forward = 1`**: Enables packet forwarding for IPv4.
- **`sudo sysctl --system`**: Applies the kernel parameter changes immediately.

## Summary
This script:
1. Updates and upgrades the system to ensure it is up to date.
2. Installs essential packages for Kubernetes.
3. Disables swap, a Kubernetes requirement.
4. Loads necessary kernel modules for networking and overlay functionality.
5. Configures kernel parameters for proper networking and packet forwarding.

It is an essential preparatory step for setting up Kubernetes on an Ubuntu system.

#
#
# master.sh Script Explained =====Master=====

This document provides a detailed explanation of the `master.sh` script, which configures a Kubernetes master node with essential networking components.

## Script Breakdown

### Initialize Kubernetes Master Node
```bash
kubeadm init --apiserver-advertise-address=$MASTER_IP \
    --pod-network-cidr=$POD_CIDR \
    --service-cidr=$SERVICE_CIDR
```
- **`kubeadm init`**: Sets up the Kubernetes control plane on the master node.
- **Options**:
  - **`--apiserver-advertise-address=$MASTER_IP`**: Advertises the master’s API server address, making it accessible to other nodes.
  - **`--pod-network-cidr=$POD_CIDR`**: Specifies the CIDR block for Pod networking.
  - **`--service-cidr=$SERVICE_CIDR`**: Specifies the CIDR block for Services networking.

### Configure kubectl for the Vagrant User
```bash
mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown -R vagrant:vagrant /home/vagrant/.kube
```
- Sets up Kubernetes CLI (kubectl) for the `vagrant` user:
  - **`mkdir -p /home/vagrant/.kube`**: Creates a directory for the kubectl configuration.
  - **`cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config`**: Copies the admin kubeconfig file to the user’s `.kube` directory.
  - **`chown -R vagrant:vagrant /home/vagrant/.kube`**: Ensures the `vagrant` user owns the `.kube` directory.

### Install Calico CNI (Container Network Interface)
```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```
- **Calico**: Provides Pod networking and network policy enforcement.
- **Command**: Applies the Calico manifest using kubectl with the admin kubeconfig file.

### Install MetalLB (Load Balancer)
```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```
- **MetalLB**: Implements load balancer functionality for Kubernetes clusters.
- **Command**: Applies the MetalLB native manifest.

### Wait for MetalLB to be Ready
```bash
sleep 30
```
- Pauses the script for 30 seconds to allow MetalLB components to initialize.

### Configure MetalLB with an IP Address Pool
```bash
cat << EOF | kubectl --kubeconfig=/etc/kubernetes/admin.conf apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.56.200-192.168.56.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advert
  namespace: metallb-system
EOF
```
- Configures MetalLB to use a specific IP address range for load balancing:
  - **IPAddressPool**: Defines a pool of IP addresses (`192.168.56.200-192.168.56.250`) for MetalLB to allocate.
  - **L2Advertisement**: Configures Layer 2 mode for MetalLB.

### Generate Join Command for Worker Nodes
```bash
kubeadm token create --print-join-command > /vagrant/join.sh
chmod +x /vagrant/join.sh
```
- **`kubeadm token create --print-join-command`**: Generates a command for worker nodes to join the cluster.
- **`> /vagrant/join.sh`**: Saves the join command to a file (`join.sh`) in the shared `/vagrant` directory.
- **`chmod +x /vagrant/join.sh`**: Makes the `join.sh` script executable.

## Summary
The `master.sh` script:
1. Initializes the Kubernetes master node.
2. Sets up kubectl for the `vagrant` user.
3. Installs essential networking components:
   - **Calico**: For Pod networking.
   - **MetalLB**: For load balancing.
4. Configures MetalLB with an IP address pool.
5. Generates a join command for worker nodes to connect to the master node.

This script ensures the master node is ready for managing a Kubernetes cluster and connecting worker nodes.


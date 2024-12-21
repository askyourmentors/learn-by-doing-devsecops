# Vagrantfile Explain

This document provides a detailed explanation of a Vagrantfile that sets up a Kubernetes cluster using Vagrant and VirtualBox. It provisions one Kubernetes v1.30 master node and a configurable number of worker nodes.

## Vagrantfile Breakdown

### Header
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
```
- Indicates that this file uses Ruby syntax.
- Configures the editing mode for compatibility with text editors like `vi`.

### Variables
```ruby
MASTER_IP = "192.168.56.10"
POD_CIDR = "10.244.0.0/16"
SERVICE_CIDR = "10.96.0.0/12"
NODE_COUNT = 2
```
- **`MASTER_IP`**: The private IP address assigned to the Kubernetes master node.
- **`POD_CIDR`**: CIDR block for Kubernetes Pods networking.
- **`SERVICE_CIDR`**: CIDR block for Kubernetes Services networking.
- **`NODE_COUNT`**: Number of worker nodes to be created.

### Vagrant Configuration
```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/jammy64"
    config.vm.box_check_update = false
```
- **`Vagrant.configure("2")`**: Specifies the configuration version (2 is standard).
- **`config.vm.box`**: Uses the `ubuntu/jammy64` base image (Ubuntu 22.04 Jammy Jellyfish).
- **`config.vm.box_check_update`**: Prevents frequent updates of the base box, speeding up setup.

### Kubernetes Master Node
```ruby
config.vm.define "k8s-master" do |master|
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", ip: MASTER_IP

    master.vm.provider "virtualbox" do |vb|
        vb.name = "k8s-master"
        vb.memory = 2048
        vb.cpus = 2
        vb.gui = false
    end

    master.vm.provision "shell", path: "scripts/common.sh"
end
```
- **`config.vm.define "k8s-master"`**: Defines a VM for the Kubernetes master node.
- **`master.vm.hostname`**: Sets the hostname as `k8s-master`.
- **`master.vm.network`**: Configures a private network with the master’s IP address.
- **VirtualBox Provider Configurations**:
  - **`vb.name`**: Names the VM as `k8s-master`.
  - **`vb.memory`**: Allocates 2048 MB RAM.
  - **`vb.cpus`**: Allocates 2 CPU cores.
  - **`vb.gui`**: Disables GUI mode for efficiency.
- **`master.vm.provision`**: Runs a shell script (`scripts/common.sh`) during provisioning to set up common dependencies.

### Kubernetes Worker Nodes
```ruby
(1..NODE_COUNT).each do |i|
    config.vm.define "k8s-worker#{i}" do |worker|
        worker.vm.hostname = "k8s-worker#{i}"
        worker.vm.network "private_network", ip: "192.168.56.#{i + 10}"

        worker.vm.provider "virtualbox" do |vb|
            vb.name = "k8s-worker#{i}"
            vb.memory = 2048
            vb.cpus = 2
            vb.gui = false
        end

        worker.vm.provision "shell", path: "scripts/common.sh"
    end
end
```
- Uses a loop (`1..NODE_COUNT`) to dynamically create worker nodes.
- **Worker Node Details**:
  - Hostname: `k8s-worker1`, `k8s-worker2`, ..., up to `NODE_COUNT`.
  - IP Address: Starts at `192.168.56.11` and increments for each worker.
- **Resources**: Allocates 2048 MB RAM and 2 CPU cores per worker node.
- **Script**: Runs `scripts/common.sh` for common setup.

### Optional Scripts
```ruby
# master.vm.provision "shell", path: "scripts/master.sh",
#     env: {
#         "MASTER_IP" => MASTER_IP,
#         "POD_CIDR" => POD_CIDR,
#         "SERVICE_CIDR" => SERVICE_CIDR
#     }
# worker.vm.provision "shell", path: "scripts/worker.sh"
```
- Placeholder for custom provisioning scripts for master (`scripts/master.sh`) and worker (`scripts/worker.sh`) nodes.
- These are commented out but can be used to set up Kubernetes components like kubeadm or kubelet.

## Summary
This Vagrantfile:
1. Sets up a multi-VM environment to simulate a Kubernetes cluster.
2. Creates one master node and multiple worker nodes.
3. Configures networking and resources for each node.
4. Includes placeholders for further customization (e.g., Kubernetes-specific setup).

Feel free to modify the file or ask for help running it!


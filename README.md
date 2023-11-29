# Deployment of a Static Website with Nginx on Kubernetes using Kubeadm, Hosted on AWS

## Project Objective
The goal of this project is to set up a static website using Nginx on a Kubernetes cluster. This cluster will be established and managed using Kubeadm and will be hosted on AWS EC2 instances. The entire infrastructure setup will be automated through Terraform.

## Prerequisites
- AWS Account
- Basic knowledge of AWS, Kubernetes, Nginx, and Terraform
- Terraform installed on your local machine

## Architecture Overview
The project involves setting up a Kubernetes cluster using Kubeadm on AWS EC2 instances. A Network Load Balancer will distribute the traffic across Kubernetes nodes. The DNS configuration will be managed by AWS Route 53, and Nginx Ingress will control the external access to the static website. TLS certificates will be managed by Cert-Manager for secure connections.

## Detailed Steps

### 1. Infrastructure Provisioning with Terraform
- Automate the setup of AWS EC2 instances and other necessary infrastructure components using Terraform scripts.

Provided is the Terraform scripts used to automate the setup of 3 AWS EC2 instances and other necessary infrastructure components for the Kubernetes cluster.

- **ec2_instances.tf**

- **gateway.tf**

- **output.tf**

- **provider.tf** 

- **security_group.tf**

- **vpc.tf**

Initialize Terraform

Open your terminal.
Navigate to the directory containing the Terraform files.

Run
```bash
terraform init
```
This command initializes Terraform, installs the AWS provider, and prepares the environment for deployment.
Plan Infrastructure Deployment


```bash
terraform plan
```
Review the actions Terraform will perform before actually making changes to your AWS infrastructure.
Apply Configuration


```bash
terraform apply
```
Confirm the action by typing yes when prompted.
Terraform will now provision the AWS infrastructure as defined in your configuration files.
Access Outputs

After successful deployment, use terraform output to view important outputs like public IP addresses of your EC2 instances.
Clean Up Resources

To remove the infrastructure and avoid unnecessary AWS charges, run terraform destroy.
Confirm the action when prompted, and Terraform will delete the resources it created.

### 2. Kubernetes Cluster Setup using Kubeadm
- Deploy and configure a Kubernetes cluster on the provisioned AWS EC2 instances using Kubeadm for cluster creation, Containerd as container runtime and weave for Weave Net for Network Policy.

**Preparation of EC2 Instances**

 - Ensure all EC2 instances are up and running as per the Terraform deployment.
 - SSH into each EC2 instance.
 - Update the package lists and install any pending updates.

Run the following script on all nodes
```bash
#!/bin/bash

# Stop the script if any command fails
set -e

# Disable swap
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load the overlay and br_netfilter modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set sysctl params, these will persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Install prerequisites
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# Add Docker’s official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker’s repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install containerd
sudo apt-get update
sudo apt-get install -y containerd.io

# Start and enable containerd service
sudo systemctl start containerd
sudo systemctl enable containerd

# Configure containerd to use systemd as the cgroup driver
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Modify the configuration file to set the systemd cgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Restart containerd to apply the changes
sudo systemctl restart containerd

# Update the apt package index and install packages needed to use the Kubernetes apt repository
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Download the public signing key for the Kubernetes package repositories
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update apt package index, install kubelet, kubeadm, and kubectl, and pin their version
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
Copy the above script into a new file, e.g., setup-kubernetes.sh.

Run ```bash chmod +x setup-kubernetes.sh ``` to make the script executable.

Execute the script with 
```bash
sudo ./setup-kubernetes.sh
```
You should see
```output
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
```
This script automates the setup process for a Kubernetes node. It starts by disabling swap and setting up necessary kernel modules like 'overlay' and 'br_netfilter'. Then, it adjusts sysctl parameters to ensure proper network forwarding and bridge filtering. The script proceeds to install containerd, a container runtime, and configures it to use systemd as the cgroup driver. It also handles the addition of Docker's and Kubernetes' official repositories and GPG keys to the system's package manager. Finally, the script installs Kubernetes components including kubelet, kubeadm, and kubectl, and ensures their versions are held to prevent unintended upgrades.

**On the master node, you will initialize the cluster**
```bash
sudo kubeadm init
```

**To start using your cluster, you need to run the following as a regular user**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Alternatively, if you are the root user, you can run:
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```
**Deploying the Weave CNI**
To allow your pods to communicate, you need to deploy a CNI. I've chosen Weave:
```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

**Joining the Worker Nodes**
On each worker node, you'll need to join them to the cluster using the token generated by kubeadm init. You will see an output like this:
```bash
kubeadm join [your-master-ip]:6443 --token [your-token] --discovery-token-ca-cert-hash sha256:[your-hash]
```
**Verifying the Cluster**
Finally, you can check if your nodes are part of the cluster:
```bash
kubectl get nodes
```


### 3. Traffic Distribution via AWS Network Load Balancer
- Implement an AWS Network Load Balancer to distribute incoming traffic efficiently across the Kubernetes nodes.

### 4. DNS Configuration with Route 53
- Use AWS Route 53 to manage the DNS for effective domain name resolution and routing to the static website.

### 5. External Access Control with Nginx Ingress
- Configure Nginx Ingress in the Kubernetes cluster to manage and control external access to the website.

### 6. Securing Connections with Cert-Manager
- Integrate Cert-Manager in the Kubernetes cluster for handling TLS certificate issuance and management.

## Deployment Steps
(Here, you can provide a step-by-step guide on how to deploy the project, including Terraform commands, Kubernetes configurations, etc.)

## Contributing
(Instructions for contributing to the project, if applicable.)

## License
(Include information about the project's license, if any.)

## Author
(Your name and contact information.)

---

*This README.md is part of the project on deploying a static website using Kubernetes, Nginx, Kubeadm, and AWS, managed with Terraform.*

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
- Deploy and configure a Kubernetes cluster on the provisioned AWS EC2 instances using Kubeadm.

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

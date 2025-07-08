# Kubernetes-Cluster-Setup-on-AWS-EC2-using-Terraform
In this lab, I created a Kubernetes cluster on AWS EC2 with Terraform. I set up EC2 instances, installed Kubernetes tools, initialized the cluster, added worker nodes, and deployed workloads. I managed the cluster via command line and verified access through a web browser

Key highlights:

Automated and repeatable infrastructure provisioning with Terraform, ensuring consistent environments

In-depth Kubernetes cluster lifecycle management: initialization, node joining, scaling, and workload deployment

Cluster health monitoring and management using kubectl commands

Troubleshooting and resolving configuration and deployment challenges

Verification of cluster and application accessibility via CLI and web browser interfaces

Application of industry best practices for container orchestration on cloud platforms

My Implementation Steps

1)Provisioned AWS EC2 Instances:
Used Terraform scripts to create and configure the master and worker EC2 instances as the infrastructure foundation.

2)Installed Kubernetes Components:
Installed kubelet, kubeadm, and kubectl on all nodes to prepare them for cluster operations.

3)Initialized Kubernetes Cluster:
Ran kubeadm init on the master node to initialize the cluster and configured networking.

4)Joined Worker Nodes:
Executed the join commands on worker nodes to add them to the Kubernetes cluster.

5)Deployed Containerized Workloads:
Deployed sample applications and workloads to the cluster using Kubernetes manifests.

6)Managed and Verified Cluster:
Used kubectl get nodes and kubectl get services to monitor cluster status and service availability.

7)Tested Application Accessibility:
Verified the deployment by accessing application endpoints on both master and worker public IPs via a web browser.



# K8s vs Docker Swarm
**This project demonstartes the differences between dockerswarm and k8s with Example**

![swarmvskubes_newcover](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/079ec15a-6894-46a1-9647-244f4d507bba)

## Introduction

This repository contains resources and information comparing Kubernetes (K8s) and Docker Swarm, two popular container orchestration platforms. The goal of this project is to provide insights into their features, differences, use cases, and considerations to help users make informed decisions when choosing between them.



### Kubernetes (K8s)

Kubernetes is an open source container orchestration platform that was initially designed by Google to manage their containers. Kubernetes has a more complex cluster structure than Docker Swarm. It usually has a builder and worker nodes architecture divided further into pods, namespaces, config maps.

### Docker Swarm
Docker Swarm is an open-source container orchestration platform built and maintained by Docker. Under the hood, Docker Swarm converts multiple Docker instances into a single virtual host. A Docker Swarm cluster generally contains three items:
- Nodes
- Services and tasks
- Load balancers
Nodes are individual instances of the Docker engine that control your cluster and manage the containers used to run your services and tasks. Docker Swarm clusters also include load balancing to route requests across nodes.
## Differences

- **Architecture**: Kubernetes follows a master-worker architecture with a separate control plane, whereas Docker Swarm uses a simpler architecture with manager and worker nodes.
- **Complexity**: Kubernetes is more complex to set up and manage compared to Docker Swarm, requiring a steeper learning curve.
- **Scaling**: Kubernetes provides more advanced scaling capabilities and is better suited for larger, more complex deployments.
- **Community and Ecosystem**: Kubernetes has a larger and more mature community with a vast ecosystem of third-party tools and resources compared to Docker Swarm.

## Use Cases

- **Kubernetes (K8s)**: Ideal for large-scale, complex deployments requiring advanced orchestration features, high availability, and scalability.
- **Docker Swarm**: Suitable for smaller projects or organizations looking for a simpler, integrated container orchestration solution with less complexity and overhead.

## Setup Kubernetes Cluster Using Kubeadm

![Kubeadm-800x500](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/12c0510a-9437-485b-8786-8d73b4ae9c72)

## Prerequisites
- Minimum two Ubuntu nodes [One master and one worker node]
- Nodes should have  minimum of 2 CPU and 2GB RAM.
- Add these ports to security group of nodes:
  
  ![Screenshot 2024-03-19 001830](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/51db383e-8217-4f4e-93fa-f5eb30f5905c)

## Installation Steps:

- Launch 2 ubuntu instances with 2 CPU and 2GB RAM
  ![Screenshot (332)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/f1e576c1-72db-4f47-95b9-7f37e1f7c199)
- Create a script file for needed steps:  <br><br>
   Step 1: Enable iptables Bridged Traffic on all the Nodes<br>
   Step 2: Disable swap  on all the Nodes<br>
   Step 3: Install CRI-O Runtime on all the nodes<br>
   Step 4: Install Kubeadm & Kubelet & Kubectl on all Nodes<br><br>
- `vim kube.sh` and add all commands for these steps<br>
   ![Screenshot (333)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/268dc76a-361e-48e9-962b-7c1c1720843e)
  
- `chmod +x kube.sh && ./kube.sh` Run this shell script on both 2 nodes
  
###  Initialize Kubeadm On Master Node To Setup Control Plane

- Initialize the master node control plane configurations using the kubeadm command. Replace with your actual private ip
  ```
  sudo kubeadm init --apiserver-advertise-address="PrivateIp"  --apiserver-cert-extra-sans="PrivateIp"  --pod-network-cidr=192.168.0.0/16  --ignore-preflight-errors Swap
  ```

  ![Screenshot (335)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/7dfb9404-9995-4ac9-b8f9-68367cbfe579)

- Use the following commands from the output to create the kubeconfig in master so that you can use kubectl to interact with cluster API.

  ```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  ```
- You can get the cluster info using the following command.`kubectl cluster-info `

  ![Screenshot 2024-03-19 004838](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/5cbf8b9d-7f58-41a4-af0a-a05046c103c9)

  


   






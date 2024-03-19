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

- Use the following commands from the output to create the kubeconfig in master so that you can use kubectl to interact with cluster API.<br>
  To start using your cluster, you need to run the following as a regular user:

  ```

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  ```
  <br>
  Alternatively, if you are the root user, you can run:
  ```
   export KUBECONFIG=/etc/kubernetes/admin.conf
   ```
- You can get the cluster info using the following command.`kubectl cluster-info `

  ![Screenshot 2024-03-19 004838](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/5cbf8b9d-7f58-41a4-af0a-a05046c103c9)

- By default, apps won’t get scheduled on the master node. If you want to use the master node for scheduling apps, taint the master node.

  ### Join Worker Nodes To Kubernetes Master Node

     
- We have set up cri-o, kubelet, and kubeadm utilities on the worker nodes as well.
- Copy the script file and create in worker node and run it.
- Execute the following command in the master node to recreate the token with the join command.
   ```
  kubeadm token create --print-join-command
  ```
- This command performs the TLS bootstrapping for the nodes.
  
  ```
  sudo kubeadm join 172.31.43.208:6443 --token j4eice.33vgvgyf5cxw4u8i \
    --discovery-token-ca-cert-hash sha256:37f94469b58bcc8f26a4aa44441fb17196a585b37288f85e22475b00c36f1c61
  ```
  ![Screenshot 2024-03-19 092956](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/c7712c43-6d2d-4640-84c9-56bcf9308e39)

- Now execute the kubectl command from the master node to check if the node is added to the master.`kubectl get nodes`<br>
  ![Screenshot 2024-03-19 093318](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/8e088cb1-a765-4712-904f-9932153b9c1e)

### Install Calico Network Plugin for Pod Networking

- Kubeadm does not configure any network plugin. You need to install a network plugin of your choice for kubernetes [PodNetworking](https://kubernetes.io/docs/concepts/cluster-administration/addons/) and enable network policy
- I am using the Calico network plugin for this setup.
- Execute the following commands to install the [Calico](https://docs.tigera.io/calico/latest/about/) network plugin operator on the cluster.
  ```
  kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
  ```
- Kubeadm doesn’t install metrics server component during its initialization. We have to install it separately.
- To verify this, if you run the `kubectl top nodes` command, you will see the Metrics API not available error.
- To install the metrics server, Explore the following metric server manifest file [Repository](https://github.com/kubernetes-sigs/metrics-server)<br>

  ![Screenshot 2024-03-19 094932](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/2dfab6a3-3d1d-4fa6-8cdf-c7e36a37c620)

### Deploy IPSR Application

- Now that we have all the components to make the cluster and applications work, let’s deploy a sample IPSR Application and see if we can access it over a NodePort
- Create a deployment `vim deployment.yml` It deploys the pod in the default namespace.<br>
  ![Screenshot (340)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/23cc6d3b-0d19-439a-9676-8184667bb9dc)

- Execute the following command for create and view deployment<br>
  ```
  kubectl create -f deployment.yml
  kubectl get deployments
  ```
  

- Expose the Nginx deployment on a NodePort 32000
- Create a service for the ipsr app `vim service.yml` <br>
  ![Screenshot (342)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/3156c98a-f51b-4c1a-aa70-9d3d7407854f)<Br>
- Add this port Number to node Security group 

- Execute the following command for create and view Services
  ```
  kubectl create -f service.yml
  kubectl get services
  ```
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/10aa6e80-8c3b-4d40-9ee5-db96f0241af1)<br>
- Check the pod status using the following command.`kubectl get pods`
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/c3217373-b724-44cf-8dc8-a017ff271e8f)<br>
- Once the deployment is up, you should be able to access the IPSR home page on the allocated NodePort.Copy the Workernode Public ip and acces it on Alllocated port 
  ![Screenshot (348)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/942ed6b3-8d07-4f16-af57-0c71d0248048)<br><br>
  ![Screenshot (347)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/d2a2a6b8-8185-4f02-97d6-889ddcaab15b)



  
  

  
  

  


   






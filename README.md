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
  Alternatively, if you are the root user, you can run:<br>
  
   ```
   export KUBECONFIG=/etc/kubernetes/admin.conf
   ```
- You can get the cluster info using the following command.`kubectl cluster-info `

  ![Screenshot 2024-03-19 004838](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/5cbf8b9d-7f58-41a4-af0a-a05046c103c9)

- By default, apps won’t get scheduled on the master node. If you want to use the master node for scheduling apps, taint the master node.
  ```
  kubectl taint nodes --all node-role.kubernetes.io/control-plane-
  ```

### Kubernetes Cluster Important Configurations 
   ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/eaf5bb45-9293-48b4-ade5-009c7f404000)

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
- Create a deployment `vim deployment.yml` It deploys the pod in the default namespace.<br><br>
  ![Screenshot (340)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/23cc6d3b-0d19-439a-9676-8184667bb9dc)<br>

- Execute the following command for create and view deployment<br>
  ```
  kubectl create -f deployment.yml
  kubectl get deployments
  ```
  

- Expose the Nginx deployment on a NodePort 32000
- Create a service for the ipsr app `vim service.yml` <br>
  ![Screenshot (342)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/3156c98a-f51b-4c1a-aa70-9d3d7407854f)<Br><br>
- Add this port Number to node Security group 

- Execute the following command for create and view Services
  ```
  kubectl create -f service.yml
  kubectl get services
  ```
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/10aa6e80-8c3b-4d40-9ee5-db96f0241af1)<br><br>
- Check the pod status using the following command.`kubectl get pods`<br><br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/c3217373-b724-44cf-8dc8-a017ff271e8f)<br>
- Once the deployment is up, you should be able to access the IPSR home page on the allocated NodePort.Copy the Workernode Public ip and acces it on Alllocated port<br>

  ![Screenshot (348)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/942ed6b3-8d07-4f16-af57-0c71d0248048)<br><br>
  ![Screenshot (347)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/d2a2a6b8-8185-4f02-97d6-889ddcaab15b)<br>

- Now, Iam trying to delete the running pods and it will be created new pods automatically<br><br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/48599f59-3042-4837-ba7d-a6ef880d2731)

### Add Kubeadm Config to Workstation
- If you prefer to connect the Kubeadm cluster using kubectl from your workstation, you can merge the kubeadm `admin.conf` with your existing kubeconfig file.
- Follow the steps given below for the configuration.<br>
  Step 1: Copy the contents of `admin.conf` from the control plane node and save it in a file named `kubeadm-config.yaml` in your workstation.<br>
  Step 2: Take a backup of the existing kubeconfig. `cp ~/.kube/config ~/.kube/config.bak` <br>
  Step 3: Merge the default config with `kubeadm-config.yaml` and export it to KUBECONFIG variable<br>
         `export KUBECONFIG=~/.kube/config:/path/to/kubeadm-config.yaml`<br>
  Step 4: Merger the configs to a file `kubectl config view --flatten > ~/.kube/merged_config.yaml`<br>
  Step 5: Replace the old config with the new config `mv ~/.kube/merged_config.yaml ~/.kube/config`<br>
  Step 6: List all the contexts `kubectl config get-contexts -o name`<br>
  Step 7: Set the current context to the kubeadm cluster. `kubectl config use-context <cluster-name-here>`<br>
- Now, you should be able to connect to the Kubeadm cluster from your local workstation kubectl utility.


### Destroy Created Services

- ```
  kubectl delete pods --all
  kubectl delete servies --all
  kubectl delete deployments --all
  kubectl delete nodes --all
  kubeadm reset
  ```
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/7bddd42d-013e-4359-a445-625bb886f9bf)<br>

## Docker Swarm  

 Step-By-Step Implementation to Deploying Example web app using Docker Swarm

### Prerequisites
- Minimum two Ubuntu nodes [One Leader and one worker node]
- Add the port  2377 to security group of nodes

### Steps:

-  Install docker with docker engine on all nodes
   ```
   sudo apt-get update -y
   sudo apt install docker.io -y
   ```
   
-  Now, Open the “Swarm Manager” node and run the command
   ```
   sudo docker swarm init
   ```
- This will be creating empty swarm<br><br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/f8040b75-8e62-41f2-89ce-237a584de88c)

- After initializing the swarm on the “swarm-manager” node, there will be one key generated to add other nodes into the swarm as a worker. Copy and run that key on the rest of the servers.<br><br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/ef5fc1c8-999d-4ea0-a45d-e49a8d877e31)<br><br>
  
- To check about all the nodes in the manager, run this command `docker node ls`<br><br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/1de7ed12-50c7-4349-b87c-b95c552b7a74)<br><br>

- Now, on Manager Node we will create a service,
  ```
  sudo docker service create --namename ipsr-app-service --replicas 2 --publish 8080:80 safuvanh/ipsr
  ```
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/f285b7dd-a4ba-41f0-855f-25e8f3aaa018)

- Now, this service will be running on all nodes. To check this, Copy the Ip Address of any of the nodes followed by port 8080. As  `<public ip of any instance>:8080`<br><br>
  ![Screenshot (358)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/b372d3c7-f038-455e-bd00-0ed946eb8960)<br><br>
  ![Screenshot (359)](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/a24b4aae-5a55-4e2b-bbab-89f0a05c5446)<br>

- Now, Iam trying to delete the running container and it will be created new container automatically<br><br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/6128c9bb-83b7-4c00-a5c2-62d5743e2e51)

- If you want to remove any node from the environment,Run this command `sudo docker swarm leave` , As we removed, one of the workers and inside the status, we can see it’s down.<br><br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/f7f7f119-49ff-4246-9f46-b8a476a8328b)
  <br>
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/807d5c05-3b46-4d97-bf9e-f0049823326b)

- For deleting created docker service, Run this command
  ```
  sudo docker service rm <service-name>
  ```
  ![image](https://github.com/safuvanh/K8S-vs-DockerSwarm/assets/156053146/102db248-add5-40ce-b16d-8a2d244de523)

  ## Conclusion

  Choosing between Kubernetes and Docker Swarm depends on various factors such as project requirements, team expertise, scalability needs, and complexity tolerance. Kubernetes offers 
  advanced features and scalability, making it suitable for large-scale deployments with complex requirements. On the other hand, Docker Swarm provides simplicity and ease of use, 
  making it a preferred choice for smaller projects or teams with limited DevOps expertise. Evaluating these platforms based on your specific use case and considering factors like 
  community support and integration capabilities will help you make the right decision for your container orchestration needs.
  




  



  
  

  



  
  

  
  

  


   






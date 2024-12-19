# Nutanix and Containers
Lab created for Nutanix Partner Elite Group - 3rd Edition / Sao Paulo-Brazil. 

- Luiz Datri | Channel SE
- Igor Menezes | Field SE
- Thiago Coral | Field SE

Spraedsheet to consolidate all the users, passwords, and IPs:
[NPEG - Kubernetes Lab Template.xlsx](https://github.com/user-attachments/files/17997863/NPEG.-.Kubernetes.Lab.Template.xlsx)

# LAB Overview
<p align="center">
  <img src="https://github.com/user-attachments/assets/c9429ae0-0d8f-4101-a9b7-1566e14ea362">
</p>

## VM Deploy and Prepare
Steps to install Minikube in the Nutanix HPOC, using the Rocky Linux VM provided in the environment using the NKP Bootcamp workload.

1. Each participant show create a virtual machine through Prism Central with:
  - Name: NPEG-_YOUR_NAME_-MINIKUBE
  - CPU: 1 vCPU with 4 cores
  - RAM: 8 GiB

<p align="center">
  <img src="https://github.com/user-attachments/assets/582c4166-d069-403f-8e1a-5811debb1c31">
</p>

  - Type: Disk
  - Operation: Clone from Image
  - Image: nkp-rocky-9.4-release-1.29.9 (Consider the last version available in the HPOC enviroment)
  - Capacity: 100 GiB
  - Bus Type: SCSI
  - BIOS: Keep default option

<p align="center">
  <img src="https://github.com/user-attachments/assets/62fe6a15-6eb6-4909-a25c-bbd0a1a3ff12">
</p>

  - Subnet: Secondary

<p align="center">
  <img src="https://github.com/user-attachments/assets/ed48e296-2c27-4481-a9e0-f2b43f63f37b">
</p>

  - Customize the guest OS with the following Cloud-init scritp.
  - Our Cloud-Init will set prepare the user, passoword and also disable unecessary repositories in our lab image.
<p align="center">
  <img src="https://github.com/user-attachments/assets/2893e5f9-1ea4-4637-a44b-f6c695103888">
</p>

```yaml
#cloud-config
ssh_pwauth: true
chpasswd:
  expire: false
  users:
  - name: nutanix
    password: nutanix/4u # Recommended to change the password or update the script to use SSH keys
    type: text
runcmd:
- dnf config-manager --disable \*
final_message: "Cloud-init Worked!"
```
2. Once the VM creation finished, power on your virtual machine and open the VM console through Nutanix Prism. Check if there is a message in the top of the screen saying "Cloud-init Worked!". If there is, you are free to log into the VM through SSH.
   - user: nutanix
   - password: nutanix/4u


## Installing Docker

3. Minikube requires a virtual or containter infrastructure to provide Kubernetes services. In our lab, we are going to install and use Docker!

Aditional documentation:
  - https://docs.docker.com/engine/install/centos/
  - https://docs.docker.com/engine/install/linux-postinstall/ 

```shell
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo    #Adds Docker repository
```
```shell
dnf repolist    #Checks if the new Docker repository is listed
```
```shell
sudo dnf -y install docker-ce docker-ce-cli containerd.io    #Installs docker and its componentes
```
```shell
sudo systemctl --now enable docker    #Enables Docker
```
```shell
sudo groupadd docker    #Creates a new group
```
```shell
sudo usermod -aG docker nutanix    #Gives our user "nutanix" permissions to run Docker without using sudo all the time
```
```shell
newgrp docker    #Enables our group and permissions changes
```
```shell
docker version    #To test it, we can check the docker version and also run the hello-world container
```
```shell
docker run hello-world    #To test it, we can run the hello-world container
```

## Installing kubectl

4. kubectl is the mais CLI tool to interact with Kubernetes environments.

Aditional documentation:
  - https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ 

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"    #Downloads the lastest version of kubectl
```
```shell
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl    #Installs the downloaded version of kubectl
```
```shell
kubectl version --client    #Command to test the installation
```

## Installing Minikube

Aditional documentation:
  - https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64    #Downloads the lastest version of Minikube for Linux
```
```shell
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64    #Installs the downloaded version of Minikube
```
```shell
minikube start --driver=docker    #Starts Minikube using the Docker infrastructure
```

## LAB 1 - My first container

In this lab we are going to create a Kubernetes Deployment and a Service using Minikube.
```shell
vi npeg-deployment.yaml    #Creates a new Yaml file.
```
The following code will be the content of our _deployment.yaml_ file.
```yaml
apiVersion: apps/v1 #The v1 API only does not have the replicaset or deployment instruction.
kind: Deployment
metadata:
  name: npeg-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata: #POD definition came from the POD manifest. It has to be a child content of template
      name: npeg-pod
      labels:
        app: myapp
        type: front-end #Must be the same label as the Selector
    spec:
      containers:
        - name: nginx-controller
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end #Must be the same label as the POD
```
To exit _vi editor_ press _ESC_ then _:wq_
```shell
cat npeg-deployment.yaml    #Checks the content of the created file
```
```shell
kubectl apply -f npeg-deployment.yaml    #Deploys the new Deployment manifest based on the created file
```
```shell
kubectl get deployment    #Gets Kubernetes Deployment information
```
```shell
kubectl get all    #Gets information about Deployments, ReplicaSets and etc
```
```shell
kubectl get pod #Gets information only about PODs
```
Copy a name of a POD, example: _npeg-deployment-76cb9fb5c4-8p5zg_
```shell
kubectl delete pod npeg-deployment-76cb9fb5c4-8p5zg    #It's going to delete the specified POD
```
```shell
kubectl get pod #Gets information only about PODs
```
Once you execute again the _get pod_ command, take a look at the _AGE_ column. You will see that one of the PODs is brand new. It happens because our deployment manifest has a replicaSet defined.
```shell
vi npeg-service.yaml    #Creates a new Yaml file.
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: npeg-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80 #Only mandatory field
      nodePort: 30008 #If not allocated, a random one will be generated
  selector: #Same as in the replicaSet, connect by labels
    app: myapp
    type: front-end
```
To exit _vi editor_ press _ESC_ then _:wq_
```shell
cat npeg-service.yaml    #Checks the content of the created file
```
```shell
ls    #Check if you have a new file npeg-service.yaml in you local folder
```
```shell
kubectl apply -f npeg-service.yaml    #Deploys the new Deployment manifest based on the created file
```
```shell
kubectl get service    #Gets Kubernetes Service information
```
```shell
minikube service npeg-service  --url    #Returns the URL of a specific Kubernetes service in the cluster managed by Minikube.
```
Copy the IP and Port output and execute with the _curl_ command. Example: _curl http://192.168.49.2:30008_

## LAB 2 - NKP + My first container

Now that we have already learned and practiced about the foundation of Kubernetes, let's create a cluster inside of NKP environment and redeploy ours manifests there!

To simplify the LAB workflow, it is recommended to have following consoles opened:
  - Prism Central
  - SSH Terminal from the Rocky Linux VM we used in LAB 1
  - NKP Console

### Accessing NKP Console:
  1. From Prism Central drop down menu, click in _Infrastructure_ and then select _Apps and Marketpalce_:
<p align="center">
  <img src="https://github.com/user-attachments/assets/286b23c7-59b3-469c-b520-c6058e34eb51">
</p> 

  2. Now in the Admin Center page, click on _My Apps_ and look for our NKP cluster already deployed in our HPOC environment. Once founded, click in _Manage_:
<p align="center">
  <img src="https://github.com/user-attachments/assets/606ceff8-a4ca-4c2c-8b83-3c9cb8334fd5">
</p> 

  3. Inside our NKP application, we'll have a link to our NKP console and also the username and password to use:
<p align="center">
  <img src="https://github.com/user-attachments/assets/5da89620-1d7d-43eb-9d2a-2bf0b0fe2664">
</p> 

### Creating our cluster through NKP Console

For this next step, the LAB instructor will have already configured a _Workspace_ and an _Infrastructure Provider_ for you to use during the lab. 
In our exemple, we have:
  - Workspace: _npeg89_
  - Infrastructure Provider: (Nutanix) _dm3poc89_

Also, the LAB instructor will provide:
  - Subnet name: Example _Secondary_
  - Control Plane Endpoint IP: Example: _10.55.89.153_
  - Load Balancer Starting IP: Example: _10.55.89.154_
  - Load Balancer Ending IP: Example: _10.55.89.155_

1. Select the determined _Workspace_:
<p align="center">
  <img src="https://github.com/user-attachments/assets/20486f43-9b64-45ff-8d60-35d836de2fe1">
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/2bdd7238-ef33-450d-b5bb-24fc2516c32b">
</p>

2. On the left, click in _Clusters_ and then _+ Add Cluster_
<p align="center">
  <img src="https://github.com/user-attachments/assets/953f557d-6f1a-4446-bd8e-efac95793baa">
</p>

3. Define a name for your cluster, considering lowercase letters
4. Complete the information to configure the _Control Plane Nodes_:
  - Here we are going to use the _Control Plane Endpoint IP_ shared by the LAB instructor
  - It's important to set the Control Plane Node Count to _1_
<p align="center">
  <img src="https://github.com/user-attachments/assets/f03e3e61-8c1f-4661-856a-3315962ede1d">
</p>

5. Complete the information to configure the _Worker Nodes_:
  - It's important to set the Worker Node Count to _1_
  - We can increase the vCPU per node to _12_, to minimize some alerts
<p align="center">
  <img src="https://github.com/user-attachments/assets/653b1c21-f7e8-4c21-aa2e-bfd2b873e126">
</p>

6. We can use the _default_ Storage Container
7. In the Networking part, we are going to provide the LoadBalancer IP Range provided by the LAB Instructor:
<p align="center">
  <img src="https://github.com/user-attachments/assets/657d0509-1ae7-4512-89a5-ff9a8b5e047b">
</p>

8. Last, we need to configure the Registry. As we are using the HPOC environment, we are going to use the _IMAGE REGISTRY MIRROR_. Click _SHOW ADVANCED_.
  - We don't need to provide any credentials
  - Double attention to paste the url in the correct fild. We user the _REGISTRY MIRROR_
```html
https://registry.nutanixdemo.com/docker.io
```
<p align="center">
  <img src="https://github.com/user-attachments/assets/ca79ddfd-6c2c-459a-8856-ce1bfaabdc91">
</p>

9. We can create our cluster and wait for it to become _Active_!
  - During the creation, observe if the Nutanix logo is appering. If it is not, it is because the registry was pasted in the wrong field.
<p align="center">
  <img src="https://github.com/user-attachments/assets/2e53ff18-c2de-477c-b9fd-3ad7cfbea4c5">
</p>

10. Once finished, we can download the _kubeconfig_ file to connect on it:
<p align="center">
  <img src="https://github.com/user-attachments/assets/4105f513-9ddc-45e8-98af-28763d30559d">
</p>

### Accessing our cluster and redeploying our manifests, but now inside a cluster created through NKP

1. Connect through SSH Terminal from the Rocky Linux VM we used in LAB 1

2. To connect into our kubernetes cluster, we need to copy our downloaded _kubeconfig_ file to our Linux VM. We are going to do it manually:
```shell
vi kubeconfig.yaml    #Creates a new Yaml file.
```
- Paste inside of this new file the whole content from the downloaded _kubeconfig_ file.
To exit _vi editor_ press _ESC_ then _:wq_

3. Once the _kubeconfig.yaml_ is ready, we can execute it through following command:
```shell
export KUBECONFIG=kubeconfig.yaml    #Execute the kubeconfig file and connects into our created cluster
```
```shell
kubectl config current-context    #Commands to validade if we are in the correct context (cluster)
```
```shell
kubectl config get-contexts    #Commands to validade if we are in the correct context (cluster)
```
```shell
kubectl get nodes    ##Gets Kubernetes Nodes information
```
Compare this last output with the VM names that compose your cluster through Prism Central. 

To recap from LAB 1:
  - We have already created two manifests in this VM:
    -  npeg-deployment.yaml
    -  npeg-service.yaml

4. We are going to redeploy our _npeg-deployment.yaml_
```shell
kubectl apply -f npeg-deployment.yaml    #Deploys the new Deployment manifest based on the created file
```
```shell
kubectl get deployment    #Gets Kubernetes Deployment information
```

5. Our first Service manifest, did not consider a LoadBalancer, that's why we couldn't acess our application outside Minikube. NKP already come with MetalLB, and we can use it to expose our POD. To accomplish it, we need to create a new Service manifest:
  - It is the same Service manifest we used before, the only difference is the _spec type: LoadBalancer_
```shell
vi npeg-loadbalancer.yaml    #Creates a new Yaml file.
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: npeg-loadbalancer
spec:
  type: LoadBalancer    #This is the difference
  ports:
    - targetPort: 80
      port: 80 #Only mandatory field
      nodePort: 30008 #If not allocated, a random one will be generated
  selector: #Same as in the replicaSet, connect by labels
    app: myapp
    type: front-end
```
To exit _vi editor_ press _ESC_ then _:wq_
```shell
cat npeg-loadbalancer.yaml    #Checks the content of the created file
```
```shell
kubectl apply -f npeg-loadbalancer.yaml    #Deploys the new Deployment manifest based on the created file
```
```shell
kubectl get service    #Gets Kubernetes Service information
```

6. Now we might have a new Service deployed with the name _npeg-service_
7. We can copy the _External IP_ and _Port_ to connecto into our application through the browser:
   - In our lab we got: _10.55.89.155:80_
<p align="center">
  <img src="https://github.com/user-attachments/assets/4cbc49b7-bebc-46a5-8e85-348826ad5c79">
</p>

### Exploring our cluster through NKP:

Finally, we can explore a few applications that come with NKP:
  - Cluster Dashboard
  - KubeCost
  - AI Navigator

1. To access the Cluster Dashboard we can click in _View Details_ and then we click in _Dashboard_
<p align="center">
  <img src="https://github.com/user-attachments/assets/031368a0-ccad-4b1e-939e-64abf67a5679">
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/d860b244-ed84-43bb-b2e2-f2eaa0be9d19">
</p>

2. In the _Dashboard_ page, we can we a grafic view of all the commands we executed through this LAB (get nodes, get deployments, get services and etc)
<p align="center">
  <img src="https://github.com/user-attachments/assets/061b7179-532e-4a57-afa4-635aca7aff41">
</p>

3. If we go back to the _Management Cluster Workspace_ we can access and use the _NKP AI Navigator_:
  - If you don't see this small balloon, check if the NKP AI Navigator is enabled.
<p align="center">
  <img src="https://github.com/user-attachments/assets/b7e97a32-0059-4499-8cb6-da0dcbd0e280">
</p>

4. We can ask, for instance, "How can I integrate my manifest with Nutanix Files?". It will bring an example, like this:
<p align="center">
  <img src="https://github.com/user-attachments/assets/743548e3-2506-42ae-83ad-7c5e23f08025">
</p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: my-image
      volumeMounts:
        - name: my-volume
          mountPath: /path/to/mount
  volumes:
    - name: my-volume
      flexVolume:
        driver: "nutanix/nutanix-files"
        options:
          volumeName: my-nutanix-volume
          shareName: my-nutanix-share
```




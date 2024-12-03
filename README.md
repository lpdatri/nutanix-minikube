# Nutanix and Containers
Lab created for Nutanix Partner Elite Group - 3rd Edition / Sao Paulo-Brazil. 

- Luiz Datri | Channel SE
- Igor Menezes | Field SE
- Thiago Coral | Field SE

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
  - Capacity: 128 GiB
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
docker run hello-world
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

# LAB Overview
<p align="center">
  <img src="https://github.com/user-attachments/assets/c9429ae0-0d8f-4101-a9b7-1566e14ea362">
</p>

## LAB 1 - My first container

In this lab we are going to create a Kubernetes Deployment and a Service.
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
To leave _vi editor_ press ESC then :wq
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
vi npeg-services.yaml    #Creates a new Yaml file.
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
To leave _vi editor_ press ESC then :wq
```shell
ls    #Check if you have a new file npeg-services.yaml in you local folder
```
```shell
kubectl apply -f npeg-services.yaml    #Deploys the new Deployment manifest based on the created file
```
```shell
kubectl get service    #Gets Kubernetes Service information
```
```shell
minikube service npeg-service  --url    #Returns the URL of a specific Kubernetes service in the cluster managed by Minikube.
```
Copy the IP and Port output and execute with the _curl_ command. Example: _curl http://192.168.49.2:30008_

## LAB 2 - NKP + My first container


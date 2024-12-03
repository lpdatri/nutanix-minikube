# Nutanix and Minikube

Steps to install Minikube in the Nutanix HPOC, using the Rocky Linux VM provided in the environment using the NKP Bootcamp workload.

## VM Deploy and Prepare

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
  


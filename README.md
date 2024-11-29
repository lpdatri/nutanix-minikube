# Nutanix and Minikube

Steps to install Minikube in the Nutanix HPOC, using the Rocky Linux VM provided in the environment using the NKP Bootcamp workload.

## VM Deploy and Prepare

1. Each participant show create a virtual machine through Prism Central
<img width="1073" alt="image" src="https://github.com/user-attachments/assets/84009b92-9318-4ccb-829c-a89fef800d28">

```yaml
cat deploy-npeg.yaml

kubectl apply -f deploy-npeg.yaml
kubectl get deployment
kubectl get all
kubectl get pod
#Copiar um nome de pod, exemplo: npeg-deployment-76cb9fb5c4-8p5zg
```

# Kubernetes Installation on AWS EC2 Instance

## Pre-requisites

* Ubuntu OS (Xenial or later)
* sudo privileges
* Internet access
* t2.medium instance type or higher

## Both Master & Worker Node

Run the following commands on both the master and worker nodes to prepare them for kubeadm.

```bash
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo apt install docker.io -y

sudo systemctl enable --now docker # enable and start in single command.


# Adding GPG keys.
curl -fsSL "https://packages.cloud.google.com/apt/doc/apt-key.gpg" | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg

# Add the repository to the sourcelist.
echo 'deb https://packages.cloud.google.com/apt kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update 
sudo apt install kubeadm=1.20.0-00 kubectl=1.20.0-00 kubelet=1.20.0-00 -y
<img width="947" alt="curl-echo" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/b083edd0-33b0-44d6-9854-f6fadbae0766">

<img width="948" alt="apt-install" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/7072af79-a5d4-436d-91ce-153efddcef0c">

```

## Master Node

1. Initialize the Kubernetes master node.

    ```bash
    sudo kubeadm init
    ```

   <img width="945" alt="kubeadm-init" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/791f4cea-0eec-42fd-a44d-86af16396e6b">

    After succesfully running, your Kubernetes control plane will be initialized successfully.

3. Set up local kubeconfig (both for root user and normal user):

    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    <img width="606" alt="cmds" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/51ed965b-ba56-495b-8ede-04b557ec8219">
    
    <img width="944" alt="apiVersion" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/7dafc10b-0a13-4367-a4fb-de374ac7fe0e">


4. Apply Weave network:

    ```bash
    kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
    ```
<img width="950" alt="worker2" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/5aa16ae3-9380-4528-91b5-f9fb501bd56f">

6. Generate a token for worker nodes to join:

    ```bash
    sudo kubeadm token create --print-join-command
    
<img width="944" alt="mastertoken-1" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/2ec94d0d-7d00-4ef4-b8f3-0a67a97671cb">

7. Expose port 6443 in the Security group for the Worker to connect to Master Node


## Worker Node

1. Run the following commands on the worker node.

    ```bash
    sudo kubeadm reset pre-flight checks
    ```
<img width="949" alt="worker pre-flight" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/169eba21-184c-48fc-980b-a139e201b0a2">

2. Paste the join command you got from the master node and append `--v=5` at the end.
*Make sure either you are working as sudo user or use `sudo` before the command*
<img width="945" alt="worker-token-added" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/89cb428f-f859-49e0-adf2-d0366fd5f2e6">

---

## Verify Cluster Connection

On Master Node:

```bash
kubectl get nodes
```
<img width="546" alt="master-list-nodes" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/b4b2bc45-ee64-428d-b009-a9fe0eb21158">

---

## HELM
# What is Helm?
Helm is a tool that automates the creation, packaging, configuration, and deployment of Kubernetes applications by combining your configuration files into a single reusable package.

# What is Helm Chart?
A Helm chart is a package that contains all the necessary resources to deploy an application to a Kubernetes cluster. This includes YAML configuration files for deployments, services, secrets, and config maps that define the desired state of your application.

## Helm installation 
 Get Helm Package
   ```bash
   curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
   sudo apt-get install apt-transport-https --yes
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee         /etc/apt/sources.list.d/helm-stable-debian.list
    sudo apt-get update
    sudo apt-get install helm
   ```
## Installing a package from Helm Repository
1. Get the list of repo - I'm getting bitnami repo
   ```bash
   helm search repo bitnami
   ```
<img width="940" alt="helm-bitnami" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/f7502959-adf1-4025-ac41-ff957af61c91">

2. Select a package from list which you want to install - I'm installing nginx
   
    ```bash
        helm install my-nginx bitnami/nginx
    ```
<img width="731" alt="helm-nginx" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/5ef07ab7-4094-4547-9acb-0bb05eeeea0a">

3. Check if the nginx pod installed is running

   ```bash
    kubectl get pode
   ```
   To check which port nginx will be running on run following command

    ```bash
    kubectl get svc --namespace default -w my-nginx
    ```
<img width="929" alt="helm-pod" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/892c2e5b-6a02-480e-a48c-77395b5bd8e0">



4. Nginx up and running - You can check that by using worker node's PublicIP along with port assigned to nginx

   <img width="960" alt="nginx-deployed" src="https://github.com/NavreetK/K8s-HelmChart/assets/46690891/0fe8b9d8-c25f-49d0-bb58-7d5b24ff0af2">



# For more information about Helm nginx chart click on the link below:
[HELM - nginx](https://github.com/NavreetK/K8s-HelmChart/blob/main/Helm%20Chart/README.md)

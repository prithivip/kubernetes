# kubernetes

![image](https://github.com/user-attachments/assets/596da7f6-0b63-4367-a4ab-79cd265662bd)

This setup requires 
Ansible Control Node with version 2.1.14 and above
Proxy Machine [Linux box with kubectl, python, kube config)
Kubernetes cluster [On-prem or Cloud provider]
Password SSH login between Ansible Control Node and Proxy Machine


On Ansible Control Node :
sudo apt update && sudo apt upgrade –y
ansible-galaxy collection install kubernetes.core:4.0.0
ansible-galaxy collection install cloud.common:3.0.0
apt-get install python3-kubernetes
apt install python3.11-venv
python3 -m venv my_env
source my_env/bin/activate
pip install kubernetes

On Proxy Machine :
sudo apt update && sudo apt upgrade –y
sudo apt install python3-pip

#Download Kubernetes Tools using Curl:
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

#Verify Checksum (Response should be kubectl:OK): 
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

#Install Kubernetes Tools: 
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#Validate install:
kubectl version

On Ansible Control Node :
Create Inventory /root/ansible/kube_inventory
[master]
10.x.x.x

[workers]
10.x.x.x
10.x.x.x

[proxy-servers]
10.x.x.x


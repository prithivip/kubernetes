# kubernetes

![image](https://github.com/user-attachments/assets/566d970b-3af5-4782-a77a-d29d46761e0d)





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

ansible-playbook ~/ansible/playbook/kube_dependencies.yml -i ~/ansible/inventory/kube_inventory

ansible-playbook ~/ansible/playbook/kube_master.yml -i ~/ansible/inventory/kube_inventory

ansible-playbook ~/ansible/playbook/kube_workers.yml -i ~/ansible/inventory/kube_inventory

Once the playbook runs successfully, you can validate the cluster is working properly by running the commands below from the Master Node:

kubectl get nodes

kubectl get all -A

We will now add the kube config of the master node to our /root/.kube/config of our proxy. 

From the master node, you can run the command below to copy the config over to your proxy:

On Proxy Node :

cd /root/

mkdir .kube

cd .kube/

scp config root@MASTER_NODE_IP:~/etc/kubernetes/admin.conf

kubectl get nodes

kubectl get all -A

Now its time to manage kubernetes resources with ansible

On Ansible Control Node :

Update /root/ansible/inventory/kube_inventory

[master]

10.x.x.x

[workers]

10.x.x.x

10.x.x.x

[proxy-servers]

10.x.x.x

Let’s create a simple playbook file as create_namespace.yml in /root/ansible/playbook/ as the following to create a namespace in your Kubernetes cluster:

- name: Create K8S resource
  hosts: proxy-servers
  tasks:
  - name: Get K8S namespace
    kubernetes.core.k8s:
      name: my-namespace
      api_version: v1
      kind: Namespace
      state: present
OR

You can also pass in your Kubernetes task manifest as a file into your Ansible playbook:

- name: Create a Namespace from K8S YAML File
  kubernetes.core.k8s:
    state: present
    src: /kube_manifests/create_namespace.yml
  
On Ansible Control Node :

ansible-playbook ~/ansible/playbooks/create_namespace.yml -i ~/ansible/inventory/kube_inventory

On Proxy Node :

kubectl get namespace

Create a file called nginx_deployment.yaml

Now let's deploy Nginx application 

On Ansible Control Node :

- name: Application Deployment
  hosts: proxy_servers
  tasks:
    - name: Create a Deployment
      kubernetes.core.k8s:
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: myapp
            namespace: my-namespace
          spec:
            replicas: 3
            selector:
              matchLabels:
                app: myapp
            template:
              metadata:
                labels:
                  app: myapp
              spec:
                containers:
                  - name: myapp-container
                    image: nginx:latest
                    ports:
                      - containerPort: 80

    - name: Expose Deployment as a Service
      kubernetes.core.k8s:
        definition:
          apiVersion: v1
          kind: Service
          metadata:
            name: myapp-service
            namespace: my-namespace
          spec:
            selector:
              app: myapp
            ports:
              - protocol: TCP
                port: 80
                targetPort: 80
            type: LoadBalancer

    
ansible-playbook ~/ansible/playbooks/nginx_deployment.yml -i ~/ansible/inventory/kube_inventory


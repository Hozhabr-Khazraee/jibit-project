# Jibit Project Documentation

## Step 1: Initial Setup

1. Update and upgrade system packages:
   ```
   sudo apt update && sudo apt upgrade
   ```

2. Install Ansible:
   ```
   sudo apt install ansible
   ```

3. Update remote servers:
   ```
   ssh user@IP_ADDRESS_1  --> sudo apt update && sudo apt upgrade
   ssh user@IP_ADDRESS_2  --> sudo apt update && sudo apt upgrade
   ```

4. Generate and copy SSH keys:
   ```
   ssh-keygen
   ssh-copy-id user@IP_ADDRESS_1
   ssh-copy-id user@IP_ADDRESS_2
   ```

5. Set up Kubernetes cluster:
   ```
   cd 1-Kubernetes-cluster
   vim inventory.ini
   ansible-playbook -i inventory.ini deploy.yml --ask-become-pass
   ```

6. Verify Kubernetes setup:
   ```
   sudo /var/lib/rancher/rke2/bin/kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get node
   ```

7. Configure kubectl:
   ```
   id
   mkdir /home/user/.kube -p
   sudo cp -R /etc/rancher/rke2/rke2.yaml /home/user/.kube/config
   sudo chmod -R 777 /home/user/.kube/config
   sudo cp -R /var/lib/rancher/rke2/bin/kubectl /usr/local/bin/
   kubectl get pod -A
   ```

## Step 2: Docker Setup and Image Building

1. Install Docker:
   ```
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt update
   apt-cache policy docker-ce
   sudo apt install -y docker-ce
   sudo usermod -aG docker ${USER}
   ```

2. Build and push Docker image:
   ```
   cd 2-boz-main
   docker build -t myapp .
   docker login
   docker tag myapp sahere/test
   docker push sahere/test
   ```

## Step 3: Kubernetes Deployments

1. Install NGINX Ingress Controller:
   ```
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   helm install nginx-ingress ingress-nginx/ingress-nginx \
     --namespace ingress-nginx \
     --create-namespace \
     --set controller.service.type=NodePort \
     --set controller.service.nodePorts.http=32080 \
     --set controller.service.nodePorts.https=32443
   ```

2. Install PostgreSQL:
   ```
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   helm search repo postgresql
   helm pull bitnami/postgresql
   tar -xvf postgresql-15.5.24.tgz
   cd postgresql/
   vim values.yaml
   ```

   Update `values.yaml`:
   ```yaml
   auth:
     postgresPassword: "qazwsx"
     username: "boz"
     password: "bozi"
     database: "boz"
   ```

   Install PostgreSQL:
   ```
   helm install postgresql .
   ```

3. Install MyApp:
   ```
   cd 3-boz-helm
   helm install myapp .
   kubectl get svc
   kubectl port-forward svc/myapp-service 1025:80
   curl 127.0.0.1:1025
   kubectl get nodes -o wide
   kubectl get ingress
   vim /etc/hosts
   curl NODE-IP:32080
   ```

## Step 4: HAProxy and Keepalived Setup

1. Update remote servers:
   ```
   ssh user@IP_ADDRESS_1  --> sudo apt update && sudo apt upgrade
   ssh user@IP_ADDRESS_2  --> sudo apt update && sudo apt upgrade
   ```

2. Generate and copy SSH keys:
   ```
   ssh-keygen
   ssh-copy-id user@IP_ADDRESS_1
   ssh-copy-id user@IP_ADDRESS_2
   ```

3. Deploy HAProxy and Keepalived:
   ```
   cd 4-Haproxy&Keepalived
   vim inventory.ini
   ansible-playbook -i inventory.ini deploy_haproxy.yml --ask-become-pass
   ```

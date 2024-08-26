# Jibit Project Documentation

## step 1:
sudo apt update && sudo apt upgrade
sudo apt install ansible
ssh user@IP_ADDRESS_1  --> sudo apt update && sudo apt upgrade
ssh user@IP_ADDRESS_2  --> sudo apt update && sudo apt upgrade
ssh-keygen
ssh-copy-id user@IP_ADDRESS_1
ssh-copy-id user@IP_ADDRESS_2
cd 1-Kubernetes-cluster
vim inventory.ini
ansible-playbook -i inventory.ini deploy.yml --ask-become-pass
sudo /var/lib/rancher/rke2/bin/kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get node
id
mkdir /home/user/.kube -p
sudo cp -R /etc/rancher/rke2/rke2.yaml /home/user/.kube/config
sudo chomd -R 777 /home/user/.kube/config
sudo cp -R /var/lib/rancher/rke2/bin/kubectl /usr/local/bin/
kubectl get pod -A

## step 2:
### install docker:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg ; echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null ; sudo apt update ; apt-cache policy docker-ce ; sudo apt install -y docker-ce ; sudo usermod -aG docker ${USER}
### build docker image and push it to docker hub:
cd 2-boz-main
docker build -t myapp .
docker login
docker tag myapp sahere/test
docker push sahere/test

## step 3:
### install nginx ingress controller:
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=32080 \
  --set controller.service.nodePorts.https=32443
### install postgresql:
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo postgresql
helm pull bitnami/postgresql
tar -xvf postgresql-15.5.24.tgz
cd postgresql/
vim values.yaml
    ```json
    auth:
      postgresPassword: "qazwsx"
      username: "boz"
      password: "bozi"
      database: "boz"
helm install postgresql .

### install myapp:
cd 3-boz-helm
helm install myapp .
kubectl get svc
kubectl port-forward svc/myapp-service 1025:80
curl 127.0.0.1:1025
kubectl get nodes -o wide
kubectl get ingress
vim /etc/hosts
curl NODE-IP:32080

## step 4:
ssh user@IP_ADDRESS_1  --> sudo apt update && sudo apt upgrade
ssh user@IP_ADDRESS_2  --> sudo apt update && sudo apt upgrade
ssh-keygen
ssh-copy-id user@IP_ADDRESS_1
ssh-copy-id user@IP_ADDRESS_2
cd 4-Haproxy&Keepalived
vim inventory.ini
ansible-playbook -i inventory.ini deploy_haproxy.yml --ask-become-pass

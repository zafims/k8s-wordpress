################ Process of Deployment #########################

=============== Deploy Wordpress in EC2 Instances ==============

---------------------------- Install k8s -----------------------
Tested OS: Ubuntu 20.14

##### Add the security group Kubernetes-related port number and calico port number

---> Protocol: TCP, Direction: Inbound
---> 6443, 2379-2380, 10250, 10259, 10257, 22, 179, 5473, 30000-32768


############# REMOVE OLD DOCKER #############
sudo apt-get remove docker docker-engine docker.io contained runs

############# INSTALL DOCKER PRE-REQUISITES ################
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release


############ ADD GPG KEY ############################
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docke echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.co $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


########### INSTALL DOCKER ENGINE ###################
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io sudo docker run hello-world
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf overlay
br_netfilter
EOF


########### Restart nodes to load them the br_netfilter and overlay ################
sudo modprobe overlay
sudo modprobe br_netfilter


######### ALLOW BRIDGED TRAFFIC FOR KUBEADM ##########################
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1 EOF
sudo sysctl --system


######### INSTALL K8S PRE-REQUISITES ######################
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl


########## DOWNLOAD GOOGLE CLOUD PUBLIC SIGNINIG KEY ###########
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg


######### ADD K8S APT REPO #######################
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee
/etc/apt/sources.list.d/kubernetes.list


######### INSTALL K8S COMPONENTS ################
sudo apt-get update
sudo apt-get install -y kubelet=1.20.1-00 kubeadm=1.20.1-00 kubectl=1.20.1-00 sudo apt-mark hold kubelet kubeadm kubectl
kubectl taint nodes --all node-role.kubernetes.io/master-
sudo touch /etc/docker/daemon.json
cat <<EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
sudo swapoff –a       #to disable the swapping 
sudo sed -i '/ swap / s/^/#/' /etc/fstab       # To persist the swap disable



############### init ##############
sudo systemctl daemon-reload
sudo systemctl restart docker



########### On the Master, node instance, follow the below commands to Initiate the Kubeadm,  generate the token and Install Calico network plugin #######

############# Kubeadm init ################
kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all export KUBECONFIG=/etc/kubernetes/admin.conf
sudo cp /etc/kubernetes/admin.conf $HOME/admin.conf sudo chown $(id -u):$(id -g) $HOME/admin.conf

############# Generate Token ###################
token=$(kubeadm token generate)
rm -f home/ubuntu/nodes-join-token.out
kubeadm token create $token --print-join-command --ttl=0 > /home/ubuntu/nodes-join-token.out cat /home/ubuntu/nodes-join-token.out


############ Install calico networking ############
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml kubectl get cs
kubectl get components tatus
kubectl cluster-info
kubectl get pods -n kube-system
mv $HOME/.kube $OME/.kube.bak
mkdir $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo systemctl restart docker.service sudo systemctl enable docker.service sudo service kubelet restart



########### In Master Node: #################

******  A ﬁle is created in the master node named “nodes-join-token”. Open it and copy the token
# cat nodes-join-token

-------------------------------- Install Wordpress --------------------------

### Install GIt
sudo apt-get install git-all


### make a directory
mkdir -p /home/ubuntu/Kubernetes 
cd /home/ubuntu/Kubernetes 

git init
git remote add kube https://github.com/zafims/k8s-wordpress.git
git pull kube master


#### Step 9.1: Go to the Kubernetes directory
cd /home/ubuntu/Kubernetes

### Step 9.2: Deploy persistent volume for MySQL
kubectl apply -f pvMysql.yml

### Step 9.3: Deploy persistent volume claim for MySQL
kubectl apply -f  pvcMysql.yml

### Step 9.4: Deploy service for MySQL
kubectl apply -f mysql-svc.yml

### Step 9.5: Deploy secrets
---> run this in CLI: echo -n 'zafimsPqS96' | base64
---> nano secret.yml

kubectl apply -f  secret.yml

### Step 9.6: Deploy Deployment of MySQL
kubectl apply -f  mysql.yml

### Step 9.7: Deploy persistent volume for WordPress
kubectl apply -f  pvWordpress.yml

### Step 9.8: Deploy persistent volume claim for WordPress
kubectl apply -f  pvcWordpress.yml

### Step 9.9: Deploy service for WordPress
kubectl apply -f wordpress-svc.yml

### Step 9.10: Deploy Deployment of WordPress
kubectl apply -f  wordpress.yml

Step 10: Access the WordPress application in browser.
http://<master/worker instance public Ip address>:30050

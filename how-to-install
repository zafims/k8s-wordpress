
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

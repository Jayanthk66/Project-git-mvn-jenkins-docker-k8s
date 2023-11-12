# devops-automation
youtubelink: https://www.youtube.com/watch?v=YggRuCiE054

Server1-Amazon linux2:  Git, Java, Maven, Jenkins, Docker
server2-Ubuntu: k8s-master:
server3-Ubuntu: k8s-worker:  

#!/bin/bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
sudo amazon-linux-extras install epel -y
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo yum install git -y
cd /opt
sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.4/binaries/apache-maven-3.9.4-bin.tar.gz
sudo tar -xvzf apache-maven-3.9.4-bin.tar.gz
sudo mv apache-maven-3.9.4 maven
sudo hostnamectl set-hostname jenkins
sudo init 6

##JAVA and MVN Environment veriable set
vi .bash_profile		#set environemt in bash_profile 

#add below java and maven path
M2_HOME=/opt/mvaven
M2=/opt/maven/bin
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.20.0.8-1.amzn2.0.1.x86_64
PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2

source .bash_profile	#to save changes in .bash_profile file
echo $PATH				#to check PATH showing java and mvn paths

find / -name java-11*	##to find java path

##Docker installation and configuration
sudo usermod -aG docker jenkins		#Add Jenkins User to the Docker Group
sudo systemctl restart docker
sudo systemctl restart jenkins
groups jenkins						#Verify Docker Group Membership
ls -l /var/run/docker.sock			$Check Docker Socket Permissions



#k8s
##do it on both master and worker
sudo apt-get install docker.io -y
systemctl restart docker

sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00

kubeadm version
kubectl version --short

#if any issues in kubeadm or kubectl try below steps
apt-get remove kubelet kubeadm kubectl -y
apt-get update
apt-get install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00 -y

kubeadm version
kubectl version --short

#Only on master node
kubeadm reset   # Reset the Kubernetes cluster if needed
kubeadm init --pod-network-cidr=192.168.0.0/16
----
##note after abouve cmd in result collect below details to join worker node
kubeadm join 172.31.43.71:6443 --token q866gi.4ydqmtwtulzicqmx \
    --discovery-token-ca-cert-hash sha256:148865a07e727ce5acea10a108edfd1d7034a13eb63be
---

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml

##Execute on workr node
#make sure Enable port:6443 on SG
kubeadm join 172.31.43.71:6443 --token q866gi.4ydqmtwtulzicqmx \
    --discovery-token-ca-cert-hash sha256:148865a07e727ce5acea10a108edfd1d7034a13eb63be
###folloe below steps only if you unable to join workinr nodes
kubeadm token create
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
kubeadm join 172.31.43.71:6443 --token NEW_TOKEN --discovery-token-ca-cert-hash sha256:NEW_HASH		#as below

kubeadm join 172.31.43.71:6443 --token rqzint.o7kwdllnqxrlp8pk --discovery-token-ca-cert-hash sha256:148865a07e727ce5acea10a108edfd1d7034a13eb63befdd456dbc199c3b1af6

#Enable port:6443 on SG




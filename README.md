Install Kubeadm (source: https://www.server-world.info/en/note?os=Ubuntu_20.04&p=kubernetes&f=2)

sudo nano /etc/apt/sources.list: comment #deb….

root@dlp:~# apt -y install docker.io apt-transport-https
# cgroup driver uses systemd
root@dlp:~# cat > /etc/docker/daemon.json <<EOF  
{
  "exec-opts": ["native.cgroupdriver=systemd"],  
  "log-driver": "json-file",  
  "log-opts": {  
    "max-size": "100m"  
  },  
  "storage-driver": "overlay2"  
}  
EOF  

root@dlp:~# systemctl restart docker  
root@dlp:~# systemctl enable docker  
root@dlp:~# cat > /etc/sysctl.d/k8s.conf <<EOF  
net.bridge.bridge-nf-call-ip6tables = 1  
net.bridge.bridge-nf-call-iptables = 1  
EOF  

root@dlp:~# sysctl –system  
root@dlp:~# swapoff -a  
root@dlp:~# vi /etc/fstab  
# comment out swap line
#/dev/mapper/ubuntu--vg-swap_1 none swap sw 0 0

------------------------------------------------------------------------------------------------------
# Create the keyrings directory
sudo mkdir -p /etc/apt/keyrings

# Add Kubernetes signing key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update package list
sudo apt update

apt -y install kubeadm kubelet kubectl  

#In Master node 
{  
  kubeadm init --apiserver-advertise-address=198.18.1.30 --pod-network-cidr=198.244.0.0/16  
  # set cluster admin user, if you set common user as cluster admin, login with it and run [sudo cp/chown ***]  
  root@dlp:~# mkdir -p $HOME/.kube  
  root@dlp:~# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
  root@dlp:~# chown $(id -u):$(id -g) $HOME/.kube/config  
  #	Configure Pod Network with Flannel.
  kubectl apply -f https://raw.githubusercontent.com/hieptran159/k8s_cluster/refs/heads/main/kube-flannel.yml  
  # show state ⇒ OK if STATUS = Ready
  root@dlp:~# kubectl get nodes  
  # show state ⇒ OK if all are Running
  root@dlp:~# kubectl get pods -A  
}
#Worker node join follow on Master

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

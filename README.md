# install-multi-node-kubernetes-cluster

## Set up the repository
Link: https://docs.docker.com/engine/install/ubuntu/

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:

Login with root user and Install Docker ( in Master & Worker Node Both)
sudo su -
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

2. Add Docker's official GPG key:

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

3. Use the following command to set up the repository:

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

  4. Update the apt package index:

  sudo apt-get update

  ## Install Docker Engine

Step 1: Install Docker Engine, containerd, and Docker Compose.
To install the latest version, run:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Step 2:  Create a file with the name containerd.conf using the command:
vim /etc/modules-load.d/containerd.conf

And add the following lines:
overlay
br_netfilter

Step 3: Save the file and run the following commands:
modprobe overlay
modprobe br_netfilter

Step 4: Create a file with the name kubernetes.conf in /etc/sysctl.d folder:
vim /etc/sysctl.d/kubernetes.conf

Add the following lines in the file and Save it:
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

Step 5: Run the commands to verify the changes:
sysctl --system
sysctl -p

Step 6: Remove the config.toml file from /etc/containerd/ Folder and run reload your system daemon:
rm -f /etc/containerd/config.toml
systemctl daemon-reload

Step 7: Add Kubernetes Repository:
apt-get update && apt-get install -y apt-transport-https ca-certificates curl

curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

Step 8: Disable Swap
vim /etc/fstab    (Remove swap mount point from this file)
swapoff -a

Step 9: Export the environment variable:
export KUBE_VERSION=1.23.0

Step 10: Install Kubernetes:
apt-get update
apt-get install -y kubelet=${KUBE_VERSION}-00 kubeadm=${KUBE_VERSION}-00 kubectl=${KUBE_VERSION}-00 kubernetes-cni=0.8.7-00
apt-mark hold kubelet kubeadm kubectl
systemctl enable kubelet
systemctl start kubelet

Step 11: Now it's time to initialize our Cluster!
---MASTER NODE ONLY---
kubeadm init --kubernetes-version=${KUBE_VERSION} (Only on master node)
kubeadm token create --print-join-command (To regenrate the tokens)

Step 12:
cp /etc/kubernetes/admin.conf /root/
chown $(id -u):$(id -g) /root/admin.conf
export KUBECONFIG=/root/admin.conf
echo 'export KUBECONFIG=/root/admin.conf' >> /root/.bashrc

Step 13: Download the daemonset yaml file of required version like following link:
wget https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

Step 14: Now apply the daemonset yaml!
kubectl apply -f weave-daemonset-k8s.yaml
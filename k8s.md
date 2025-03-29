# install ssh adn enable 

sudo apt update
sudo apt install -y openssh-server

sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh

# check firewall

if want
sudo ufw allow ssh
sudo ufw enable
sudo ufw status


# Step 1: Remove Old Kubernetes & Docker Configurations
If you've previously attempted to install Kubernetes, clean up old files to avoid conflicts:
sudo apt-get remove -y kubeadm kubelet kubectl containerd.io docker.io docker-ce docker-ce-cli

# Delete Kubernetes configuration directories
sudo rm -rf /etc/kubernetes /var/lib/kubelet /var/lib/etcd /etc/cni /opt/cni

# Remove old Kubernetes and Docker repositories & keys
sudo rm -f /etc/apt/sources.list.d/kubernetes.list
sudo rm -f /etc/apt/keyrings/kubernetes-archive-keyring.gpg
sudo rm -f /etc/apt/sources.list.d/docker.list
sudo rm -f /etc/apt/keyrings/docker.gpg

# Reload system daemon to clear any references to old services
sudo systemctl daemon-reload

# Remove old container runtime configurations (if any)
sudo rm -rf /etc/containerd /var/lib/containerd
🚀 Step 2: Install Dependencies



# Update package list
sudo apt update

# Install required packages
sudo apt install -y apt-transport-https ca-certificates curl gpg
📦 Step 3: Install & Configure Container Runtime (containerd)


# Install containerd
sudo apt install -y containerd

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# Enable systemd cgroups for better compatibility with Kubernetes
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Restart containerd service
sudo systemctl restart containerd
sudo systemctl enable containerd




🔑 Step 4: Add the Official Kubernetes Repository

# Create directory for Kubernetes keyrings
sudo mkdir -p /etc/apt/keyrings

# Add the Kubernetes signing key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository for Ubuntu
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null

# Update package list
sudo apt update

# Step 5: Install Kubernetes Components

# Install a specific version of kubeadm, kubelet, and kubectl
sudo apt install -y kubeadm=1.29.9-1.1 kubelet=1.29.9-1.1 kubectl=1.29.9-1.1

# Prevent automatic updates to these packages
sudo apt-mark hold kubelet kubeadm kubectl
🛠 Step 6: Initialize the Kubernetes Cluster (Master Node)

# Initialize Kubernetes master node with a default pod network (Flannel)
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
🔗 Step 7: Set Up kubectl for the Current User


# Set up kubeconfig for the current user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
### Step 8: Deploy a Network Plugin (Flannel)

# Install Flannel CNI for networking
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


✅ Final Verification

# Check if the node is ready
kubectl get nodes

# Check if system pods are running
kubectl get pods -n kube-system






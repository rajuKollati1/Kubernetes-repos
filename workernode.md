🔧 Prerequisites for Worker Node (Ubuntu)
Make sure these are already done:

OS: Ubuntu 20.04+ (same as master node)

Hostname set (e.g., k8s-worke-node)

Swap disabled (swapoff -a and comment in /etc/fstab)

Time synced (ntp or chrony)

Docker or container runtime installed (we’ll use containerd)

🛠️ Step-by-Step Setup
Step 1: Install Container Runtime (containerd)
bash
Copy
Edit
apt update && apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install containerd
apt update && apt install -y containerd.io

# Configure containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml

# Use systemd cgroup driver (IMPORTANT!)
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Restart containerd
systemctl restart containerd
systemctl enable containerd
Step 2: Install Kubernetes Packages
bash
Copy
Edit
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt update
apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
Step 3: Join the Worker Node to the Cluster
From your master node, run:

bash
Copy
Edit
kubeadm token create --print-join-command
It will give something like:

bash
Copy
Edit
kubeadm join 192.168.0.145:6443 --token abcdef.0123456789abcdef \
--discovery-token-ca-cert-hash sha256:<hash>
👉 Copy and run that command on the worker node.

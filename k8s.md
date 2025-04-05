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

🛠 Steps to Fix the Worker Node Join Issue
1️⃣ Reset the Worker Node
Since Kubernetes files already exist, we need to reset the node before joining. Run the following on the worker node:

bash
Copy
Edit
sudo kubeadm reset -f
🔹 This will remove any previous configurations.

2️⃣ Clean Up Kubernetes Directories
After resetting, delete any leftover Kubernetes files:

bash
Copy
Edit
sudo rm -rf /etc/kubernetes /var/lib/kubelet /var/lib/etcd /var/lib/cni /var/run/kubernetes
Then restart the kubelet service:

bash
Copy
Edit
sudo systemctl restart kubelet
3️⃣ Rejoin the Worker Node
Now, run the join command again:

bash
Copy
Edit
sudo kubeadm join 192.168.0.145:6443 --token oxyhkk.dx8eax997v35znqs \
    --discovery-token-ca-cert-hash sha256:add30507d4e43195b73901ea34ff61e6563e0c94364cd2bdcb6



# AFter reboot your node vms you have to follow this



# Turn off swap immediately
sudo swapoff -a

# Remove any swap entries from /etc/fstab (to prevent it from re-enabling on reboot)
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Verify swap is disabled
free -h


✅ Step-by-Step: Set Up GitHub Self-Hosted Runner with k8s-m User
1. Switch to k8s-m user
bash
Copy
Edit
su - k8s-m
2. Create the actions-runner folder and download the runner
bash
Copy
Edit
mkdir -p ~/actions-runner && cd ~/actions-runner

curl -o actions-runner-linux-x64-2.316.0.tar.gz -L \
https://github.com/actions/runner/releases/download/v2.316.0/actions-runner-linux-x64-2.316.0.tar.gz

tar xzf actions-runner-linux-x64-2.316.0.tar.gz
3. Get your GitHub runner registration token
Go to your GitHub repo:

Click Settings → Actions → Runners

Click "New self-hosted runner"

Select Linux → x64

It will show a command with your token. Use that in the next step.

4. Register the runner
bash
Copy
Edit
./config.sh --url https://github.com/<your-username>/<your-repo> --token <your-token>
Replace with your actual GitHub username/repo and token.

5. Start the runner
bash
Copy
Edit
./run.sh
You should now see:

css
Copy
Edit
√ Connected to GitHub
Listening for Jobs
🔁 Optional: Start the Runner as a Service
To run it in background after reboots:

bash
Copy
Edit
sudo ./svc.sh install
sudo ./svc.sh start
Let me know if you want help generating the token or writing a simple test workflow to try it out 🚀





# github runners 

You said:
what i need to do with 4th step


ChatGPT said:
Great question, Raju! Step 4 is where you register your self-hosted runner with your GitHub repository using a special token. Here's exactly what to do:

✅ Step 4: Register Your Runner with GitHub
🔹 1. Go to your GitHub repo in the browser
For example:
👉 https://github.com/rajuKollati1/your-repo-name

Replace your-repo-name with the actual repo where you want the runner.

🔹 2. Open the Runner setup page:
Click on Settings (top-right of your repo)

Scroll down to "Actions" → "Runners"

Click “New self-hosted runner”

Choose:

OS: Linux

Arch: x64

🔹 3. Copy the config command from GitHub
GitHub will give you something like this:

bash
Copy
Edit
./config.sh --url https://github.com/rajuKollati1/your-repo-name --token AABBBCCCDDDEEEFFF
This command registers your runner with GitHub using a temporary token (valid for ~1 hour).
Make sure you paste it into your terminal where you already did Step 3 (cd ~/actions-runner).

🔹 4. Paste and run it on your k8s-m machine
In your terminal:

bash
Copy
Edit
./config.sh --url https://github.com/rajuKollati1/your-repo-name --token YOUR_TOKEN_HERE
GitHub will then ask:

Runner name (press Enter to accept default)

Work folder (press Enter to accept default)

✅ Once it’s registered:
Then just start the runner:

bash
Copy
Edit
./run.sh
Let me know if you want help creating a test workflow that will use this runner!







You said:
./config.sh --url https://github.com/rajuKollati1/TestTest --token A7AB2HQMKFX3JTHPHQGAA7TH6F7CG


ChatGPT said:
Perfect, Raju! You're right on track 🎯

Now, just follow these steps after running that ./config.sh command:

✅ After You Run ./config.sh:
You'll be prompted with a few setup questions in the terminal:

Enter name of the runner
You can press Enter to accept the default name (usually your hostname), or type something like k8s-runner.

Enter work folder
Just press Enter again to accept the default (_work).

Once done, you’ll see something like:

css
Copy
Edit
√ Connected to GitHub
√ Successfully added the runner
✅ Then Start the Runner
After registration is complete:

bash
Copy
Edit
./run.sh
You should see:

css
Copy
Edit
√ Connected to GitHub
2024-04-05 15:40:59Z: Listening for Jobs
That means your runner is live and waiting for GitHub Actions jobs! 🟢





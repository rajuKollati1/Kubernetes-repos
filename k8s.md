sudo apt update
sudo apt install -y openssh-server

sudo systemctl enable ssh
sudo systemctl start ssh


sudo systemctl status ssh

if want
sudo ufw allow ssh
sudo ufw enable
sudo ufw status

*Check latest URN's for Ubuntu
```
az vm image list --location southeastasia --offer UbuntuServer --publisher Canonical --sku 18.04 --all --output table
```
```
$ az group create --name ubu1804-rg --location southeastasia
$ az vm create --location southeastasia --resource-group ubu1804-rg --name ubu1804 --public-ip-address-dns-name ubu1804 \
--image Canonical:UbuntuServer:18.04-DAILY-LTS:18.04.201804262 --admin-username myuser --admin-password 'SS12345678$$' \
--size Standard_B1ms \
--data-disk-sizes-gb 5 --tags environmenttype=dev owner=harry@oceanliner.com
ssh -l myuser ubu1804.southeastasia.cloudapp.azure.com
# Install updates
sudo yum -y update ; sudo reboot
```
## Configure 1st Data Disk
```
$ (echo n; echo p; echo 1; echo ; echo ; echo w) | sudo fdisk /dev/sdc
$ sudo mkfs -t ext4 /dev/sdc1
$ sudo mkdir /datadrive1 && sudo mount /dev/sdc1 /datadrive1
$ df -h
$ mount
$ sudo -i blkid | grep sdc1
/dev/sdc1: UUID="cda63657-f1a5-4739-b50c-8339768e8ec8" TYPE="ext4"
# Add to /etc/fstab
UUID=cda63657-f1a5-4739-b50c-8339768e8ec8   /datadrive1  ext4    defaults,nofail,barrier=0   0  2
```
```
# Configure Ubuntu 18.04 Server on Azure
# 30th Jan 2018
# Apps / Tools : az, docker-ce

# Edit Inbound NSG's - 80/tcp, 443/tcp, 8080/tcp

MYUSER=motorhead

# Configure the time zone after deploying on Azure
timedatectl set-timezone Asia/Singapore

# Install packages to allow apt to use a repository over HTTPS:
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Install docker community edition
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
# Update the apt package index
sudo apt-get update

sudo apt-get install docker-ce
systemctl unmask docker
systemctl unmask docker.socket
sudo usermod -aG docker $MYUSER
sudo systemctl start docker
sudo docker run hello-world
docker info
docker version

# Install az CLI 2
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
     sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893
sudo apt-get install apt-transport-https
sudo apt-get update && sudo apt-get install azure-cli

# Install latest kubectl using az
sudo az acs kubernetes install-cli

# Install additional apps+tools
sudo apt install golang-go links python-pip jq

# Configure firewalling and allow 22/tcp,80/tcp,443/tcp
sudo ufw allow ssh/tcp
sudo ufw allow http/tcp
sudo ufw allow https/tcp
sudo ufw logging on
sudo ufw enable
sudo ufw status verbose
```

<pre>
# Configure Ubuntu 17.04 Server on Azure
# 16th July 2017
# Apps / Tools : az, docker-ce

# Edit Inbound NSG's - 80/tcp, 443/tcp, 8080/tcp

MYUSER=motorhead

# Configure the time zone after deploying on Azure
timedatectl set-timezone Asia/Singapore

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

# Deploy Jenkins
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
sudo echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt-get update
sudo apt-get install jenkins
sudo usermod -aG docker jenkins
systemctl start jenkins
sudo systemctl status jenkins
sudo ufw allow 8080
sudo ufw enable

wget http://127.0.0.1:8080/jnlpJars/jenkins-cli.jar
JENKINSADMINPASS=$(cat /var/lib/jenkins/secrets/initialAdminPassword)
java -jar jenkins-cli.jar -s http://127.0.0.1:8080 who-am-i --username admin --password $JENKINSADMINPASS

# https://www.digitalocean.com/community/tutorials/how-to-set-up-continuous-integration-pipelines-in-jenkins-on-ubuntu-16-04
# http://www.scmgalaxy.com/tutorials/complete-guide-to-use-jenkins-cli-command-line

# Configure acs-engine
mkdir $HOME/gopath
# Add to $HOME/.profile
cat >> $HOME/.profile <<_EOF_
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/gopath
_EOF_
source $HOME/.profile
go get github.com/Azure/acs-engine
go get all
cd $GOPATH/src/github.com/Azure/acs-engine
go build
./acs-engine

# https://github.com/Azure/acs-engine/blob/master/docs/acsengine.md

</pre>

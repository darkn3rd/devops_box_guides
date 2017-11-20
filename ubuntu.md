# **Pedantic ADHD Guide to DevOps ToolBox (Ubuntu Xenial)**

This is my guide to getting essential tools for DevOps on Zorion 12.1 falvor of Ubuntu 16.04 Xenial:

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Zorin
Description:	Zorin OS 12.1
Release:	12
Codename:	xenial
```

### **Essential GNU Tools**

Linux is already bundled with desirable tools like GNU Sed and GNU Awk.  Here are some other small tools you may want to use:

```bash
# Basic Essentials
sudo apt install tree
# Useful tools
sudo apt install xml2
sudo apt install jq
sudo apt install sqlite3
sudo apt install ngrep
```

## **Vagrant and Docker**

Developers need to test in Linux, as production systems are not on Macbooks.

### **Virtualization with Vagrant**

```bash
# Install Virtualbox for Vagrant
VBOX_DEB_KEY="https://www.virtualbox.org/download/oracle_vbox_2016.asc"
VBOX_DEB_SRC='deb http://download.virtualbox.org/virtualbox/debian xenial contrib'
wget -q ${VBOX_DEB_KEY}-O- | sudo apt-key add -
sudo echo ${VBOX_DEB_SRC} >> /etc/apt/sources.list
sudo apt update
sudo apt install -y virtualbox-5.1
```

### **Virtualization with QEMU/KVM**

```bash
sudo apt-get -y install qemu-kvm libvirt-bin bridge-utils virt-manager
sudo adduser ${USER} libvirtd
sudo virsh -c qemu:///system list
```

## **Vagrant**

Download and Install Vagrant:

```bash
VERSION='1.9.5'
PACKAGE="vagrant_${VERSION}_x86_64.deb"
curl -O https://releases.hashicorp.com/vagrant/1.9.5/${PACKAGE}
sudo dpkg -i ${PACKAGE}
```

### **Docker CE**

Docker was OSS, but has since divided into CE and EE distributions.  

This is CE (Community Edition):

```bash
# Remove OSS Docker Engine
sudo apt-get remove docker docker-engine
# Install Prerequisites
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
# Install Packages
DCKR_DEB_SRC="deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
DCKR_DEB_KEY="https://download.docker.com/linux/ubuntu/gpg"
curl -fsSL ${DCKR_DEB_KEY} | sudo apt-key add
sudo add-apt-repository ${DCKR_DEB_SRC}
# Check Out Specific Versions Use:
#   apt-cache madison docker-ce
sudo apt-get install docker-ce
# Test Docker
sudo docker run hello-world
```

## **Text Editors**

### **VIM 8**

```bash
sudo add-apt-repository ppa:jonathonf/vim
sudo apt update && sudo apt install vim
# To Remove
#   sudo apt install ppa-purge && sudo ppa-purge ppa:jonathonf/vim
```

Reference: https://launchpad.net/~jonathonf/+archive/ubuntu/vim

## **Sublime 3**

```bash
DEB_KEY="https://download.sublimetext.com/sublimehq-pub.gpg"
DEB_SRC='deb https://download.sublimetext.com/ apt/stable/'
wget -qO - ${DEB_KEY} | sudo apt-key add -
echo "${DEB_SRC}" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt update && sudo apt install sublime-text
```

## **VisualStudio Code**

```bash
#################
# Add the Repository Entry
######################
# Add GPG Key
KEY="https://packages.microsoft.com/keys/microsoft.asc"
REPO="https://packages.microsoft.com/repos/vscode"
curl ${KEY} | gpg --dearmor > microsoft.gpg
# Add Entry to List
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c "echo 'deb [arch=amd64] ${REPO} stable main' > /etc/apt/sources.list.d/vscode.list"
#################
# Install the Text Editor
######################
sudo apt-get update
sudo apt-get install -y code
```

Let's get some packages:

```bash
#################
# Create a List of Packages
######################
cat <<-CODEFILE_EOF > code_packages.txt
#####
# VAGRANT/DOCKER INTEGRATION
##########
bbenoist.vagrant
marcostazi.VS-code-vagrantfile
PeterJausovec.vscode-docker
#####
# CHANGE CONFIGURATION
##########
haaaad.ansible
Pendrica.Chef
azaugg.vscode-cfengine
jpogran.puppet-vscode
#####
# WEB CONFIGURATIONS
##########
shanoor.vscode-nginx
mrmlnc.vscode-apache
#####
# LANGUAGES
##########
luggage66.AWK
rebornix.Ruby
donjayamanne.python
luggage66.VBScript
#ms-vscode.PowerShell
avli.clojure
kalitaalexey.vscode-rust
Kasik96.swift
lukehoban.Go
mjmcloug.vscode-elixir
keyring.Lua
wholroyd.hcl
mauve.terraform
bungcip.better-toml
jakeboone02.cypher-query-language
stevejpurves.cucumber
kumar-harsh.graphql-for-vscode
#####
# LINTER
##########
misogi.ruby-rubocop
t-sauer.autolinting-for-javascript
lkytal.coffeelinter
faustinoaq.javac-linter
shinnn.stylelint
#####
# WEB CLIENT TOOLS
##########
hdg.live-html-previewer
wcwhitehead.bootstrap-3-snippets
#####
# STATISTICS
##########
Ikuyadeu.r
#####
# PACKAGE CREATION
##########
idleberg.nsis
idleberg.nsis-plugins
idleberg.pynsist
#####
# MISC. USEFUL
##########
robertohuertasm.vscode-icons
eamodio.gitlens
CODEFILE_EOF
#################
# Install the Packages (1 at a time)
######################
for PKG in $(cat code_packages.txt | grep -v "^#"); do code --install-extension ${PKG}; done
```



## **Atom**

```bash
sudo add-apt-repository ppa:webupd8team/atom
sudo apt update && sudo apt install atom
# ton of color syntax highlighting, curate list as desired
cat <<-APMFILE_EOF > apm_packages.txt
ansible-vault
atom-jade
atom-jinja2
atom-toolbar
atom-typescript
console-panel
file-icons
intentions
language-ansible
language-apache
language-autotools
language-awk
language-batch
language-cfengine3
language-chef
language-csv
language-cucumber
language-cypher
language-diff
language-docker
language-elixir
language-gradle
language-groovy
language-habitat
language-haproxy
language-hcl
language-hosts
language-ini
language-kickstart
language-log
language-nginx
language-nsis
language-patch
language-powershell
language-puppet
language-rpm-spec
language-rspec
language-rust
language-salt
language-sln
language-swift
language-syslog-ng
language-systemd
language-tcl
language-vbscript
linter
linter-ui-default
APMFILE_EOF
apm install --packages-file apm_packages.txt
```


## **Language Platforms**

### **Highly Concurrent Languages**

#### **Go Language**

Installation of Go Language environment:

```bash
#################
# INSTALL Manually Install of Go-Lang from the Source
######################
ARCH=amd64
OS=$(uname -s)
VERSION="1.9.2"
ARCHIVE="go${VERSION}.${OS,,}-$ARCH.tar.gz"
cd ~/Downloads
# Download locally
curl -O https://storage.googleapis.com/golang/${ARCHIVE}
tar -xvf ${ARCHIVE}
# place go directory structure
sudo mv go /usr/local
# make binaries accessible in path
for F in /usr/local/go/bin/*; do sudo ln -sf $F /usr/local/bin/$(basename $F); done
```

Configure a local Go workspace:

```bash
#################
# Setup General Workspace
######################
mkdir ~/go
echo 'export GOPATH=${HOME}/go' >> ~/.bashrc
echo 'export PATH=${GOPATH}/bin:${PATH}' >> ~/.bashrc
export GOPATH=${HOME}/go
export PATH=${GOPATH}/bin:${PATH}
```

Compile and Run a program:

```bash
#################
# Example Build/Run Example Program
######################
GITHUB_USER=# >>> Your Account Name Goes Here <<<<
if [[ ! -z $GITHUB_USER ]]; then
  # Setup Workspace Related to your GitHub Account
  mkdir -p ${GOPATH}/src/github.com/${GITHUB_USER}
  # Setup Project in User Workspace
  mkdir -p ${GOPATH}/src/github.com/${GITHUB_USER}/hello
  # Create a Program
  cat <<-'HELLO_EOF' > ${GOPATH}/src/github.com/${GITHUB_USER}/hello/hello.go
package main
import "fmt"
func main() {
 fmt.Printf("Hello, world. I'm running Go!\n")
}
HELLO_EOF
  # Compile the Program
  go install github.com/${GITHUB_USER}/hello
  # Run the Program
  hello
fi
```

For installing library packages, you'd do something like this:

```bash
#################
# Example Install a Package
######################
go get -u github.com/ramya-rao-a/go-outline # JSON from GO
go get -u goji.io                       # Web Micro-Framework
go get -u github.com/gorilla/mux        # Web Micro-Framework
go get -u github.com/astaxie/beego      # Web Framework
go get -u github.com/aws/aws-sdk-go     # AWS
go get -u github.com/spf13/cobra/cobra  # CLI tool framework
go get -u github.com/stretchr/testify   # Test framework
go get -u github.com/alecthomas/log4go  # Logging
go get -u github.com/graphql-go/graphql # API framework
go get -u github.com/johnnadratowski/golang-neo4j-bolt-driver # Graph DB connect
```

### **Java Platform**

Java, Ant, Groovy, Gradle, Maven, Rhino added later.


### **CLR Platform**


#### **PowerShell**

```bash
# Import the public repository GPG keys
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
# Register the Microsoft Ubuntu repository
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list | sudo tee /etc/apt/sources.list.d/microsoft.list
# Update the list of products
sudo apt-get update -qq
# Install PowerShell
sudo apt-get install -y powershell
# Start PowerShell
pwsh
```

### **Perl Environment**

Perl 5.22 is already installed with the system.

```bash
# Install Package Manager
sudo apt install -y cpanminus
cpanm --local-lib=~/perl5 local::lib
eval $(perl -I ~/perl5/lib/perl5/ -Mlocal::lib)

# Install Perl Modules
cpanm YAML::XS
cpanm HTTP::Tiny
cpanm Log::Log4perl
```

### **Python 2 Environment**

Ubuntu 16.04 comes with aversion of Python (python 2.7.12 as of June 2017). Unfortunately, the package manager `pip` is not installed.

Python 2.7.12 is already installed with the system.

```bash
# Install PIP Package Manager
sudo apt install -y  python-pip
# Upgrade SetupTools (easy_install) and PIP
sudo -H pip install --upgrade pip setuptools
# VirtualEnv setup
sudo -H pip install virtualenv virtualenvwrapper
mkdir ${HOME}/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
# Install Some Libraries
sudo -H pip install configparser xmltodict PyYAML
```

### **NodeJS using NVM**

```bash
# From http://nvm.sh
NVM_VERSION='v0.33.2' # Annoying, VERSION keeps shifting, think they'd make latest
NVM_INSTALL_SCRIPT="https://raw.githubusercontent.com/creationix/nvm/${NVM_VERSION}/install.sh"
curl -o- ${NVM_INSTALL_SCRIPT} | bash
# Use NVM immediately
export NVM_DIR="${HOME}/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
# Install Some Node Platforms
command -v nvm &> /dev/null && nvm install node # latest
command -v nvm &> /dev/null && nvm install 6    # specific version
```

Afterward, install Node modules:

```bash
npm -g install grunt mocha bower
npm -g install typescript coffee-script
```

### **Ruby using RVM**

Install RVM and Ruby

```bash
# install rvm
gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
source ${HOME}/.rvm/scripts/rvm
# ruby installs take long time - download/compile
command -v rmv &> /dev/null && rvm requirements
command -v rvm &> /dev/null && rvm install ruby
command -v rvm &> /dev/null && rvm install 2.3
```

Select the desired ruby version, and then update the gems and install some gems.

Update gems and install some gems:

```bash
# select desired ruby
rvm use ruby-2.4.0
# update gems
gem update
# install some gems
gem install bundler rake nori inifile sqlite3
```

## **Advanced Tools**

### **Change Configuration Tools**

Alright, I am not endorsing any particular tool, install the ones you like.

#### **Ansible**

```bash
# Installing Ansible
sudo -H pip install ansible

# Upgrading Ansible
sudo -H pip uninstall pyOpenSSL  # if needed
sudo -H pip install -U ansible
```

#### **SaltStack**

```bash
# Salt Stack
sudo -H pip install salt
```

#### **CFEngine3**

```bash
# CFEngine
sudo apt install -y cfengine3
```

#### **Puppet 3.x**

The default package on Ubuntu Xenial is for Puppet Agent 3.8.5 (Oct. 2017).  

***NOTE***: This may cause problems with projects on [PuppetForge](https://forge.puppet.com/)., as libraries or dependencies on those libraries are no longer compatible with Puppet 3.x.  This is only a problem if you use [PuppetForge](https://forge.puppet.com/).

```bash
# Puppet
sudo apt install puppet
```

#### **ChefDK**

The [ChefDK](https://downloads.chef.io/chefdk) bundles several tools used with Chef, so that they do not have to be installed invidually.  It comes with its own embedded Ruby.

```bash
# Chef DK
VERSION='2.3.5'
PACKAGE="chefdk_${VERSION}-1_amd64.deb"
curl -O  https://packages.chef.io/files/stable/chefdk/${VERSION}/ubuntu/16.04/${PACKAGE}
sudo dpkg -i ${PACKAGE}
```

### **AWS Tools**

AWS Command Line Tools and [AWS Vault](https://github.com/99designs/aws-vault).

```bash
sudo -H pip install awscli
mkdir -p ${HOME}/.aws
# configure environment(s)
touch ${HOME}/.aws/credentials
touch ${HOME}/.aws/config
# manage secrets w/ keychain
brew cask install aws-vault
# you can setup different environments (for multiple AWS accounts)
# convenience functions to toggle environment
cat <<-PROFILE_EOF >> ${HOME}/.profile
setaws() {
  if [ $# -ne 0 ]; then
    export AWS_DEFAULT_PROFILE=$1
  fi
}

getaws() {
    echo AWS_DEFAULT_PROFILE=${AWS_DEFAULT_PROFILE}
}

listaws() {
  PROFILES=$(grep profile ${HOME}/.aws/config | sed 's/profile //' | tr -d '[]')
  for PROFILE in ${PROFILES}; do
     if [[ ${PROFILE} == ${AWS_DEFAULT_PROFILE} ]]; then
       SEL="=>"
     else
       SEL="  "
     fi
     printf " ${SEL} %s\n" ${PROFILE};
  done
}
PROFILE_EOF

cat <<-BASHRC_EOF >> ${HOME}/.bashrc
setaws <% YOUR SELECTION GOES HERE %>
BASHRC_EOF
```

#### **AWS Profile Examples**

This is a sample AWS configuration file for fictional ACME company.  Put something like this into your `${HOME}/.aws/config`.

```ini
[default]
region = us-west-2
output = json

[profile myhomestuff]
region = us-west-2
output = json

[profile acme-dev]
region = us-west-2
output = json

[profile acme-prod]
region = us-west-2
output = json
```

This would be a corresponding credentials file.  Put data into `${HOME}/.aws/credentials`.  Replace REDACTED and fictional profiles with one's that make sense to your environment, and id/key generated from AWS.

```ini
[default]
aws_access_key_id = REDACTED
aws_secret_access_key = REDACTED

[myhomestuff]
aws_access_key_id = REDACTED
aws_secret_access_key = REDACTED

[acme-dev]
aws_access_key_id = REDACTED
aws_secret_access_key = REDACTED

[acme-prod]
aws_access_key_id = REDACTED
aws_secret_access_key = REDACTED
```

### **HashiCorp Tools: Howbow Dah**

```bash
# Consul
VERSION='0.8.4'
PACKAGE="consul_${VERSION}_linux_amd64.zip"
curl -O https://releases.hashicorp.com/consul/${VERSION}/${PACKAGE}
unzip ${PACKAGE}
sudo mv consul /usr/local/bin/


# Consul-Template
VERSION='0.18.5'
PACKAGE="consul-template_${VERSION}_linux_amd64.zip"
curl -O https://releases.hashicorp.com/consul-template/${VERSION}/${PACKAGE}
unzip ${PACKAGE}
sudo mv consul-template /usr/local/bin/


# Nomad
VERSION='0.5.6'
PACKAGE="nomad_${VERSION}_linux_amd64.zip"
curl -O https://releases.hashicorp.com/nomad/${VERSION}/${PACKAGE}
unzip ${PACKAGE}
sudo mv nomad /usr/local/bin/

# Vault
VERSION='0.7.3'
PACKAGE="vault_${VERSION}_linux_amd64.zip"
curl -O https://releases.hashicorp.com/vault/${VERSION}/${PACKAGE}
unzip ${PACKAGE}
sudo mv vault /usr/local/bin/

# Packer
VERSION='1.1.1'
PACKAGE="packer_${VERSION}_linux_amd64.zip"
curl -O https://releases.hashicorp.com/packer/${VERSION}/${PACKAGE}
unzip ${PACKAGE}
sudo mv packer /usr/local/bin/

# Terraform
VERSION='0.10.8'
PACKAGE="terraform_${VERSION}_linux_amd64.zip"
curl -O https://releases.hashicorp.com/terraform/${VERSION}/${PACKAGE}
unzip ${PACKAGE}
sudo mv terraform /usr/local/bin/
```

### **Orchestration Tools**

As container orchestration platforms are becoming popular, modeling infrastructure scenarios on a laptop is convenient.

#### **Kubernetes with Mini-Kube**

[MiniKube](https://github.com/kubernetes/minikube) is a popular tool for running a small [Kubernetes](https://kubernetes.io/) cluster on your laptop.

```bash
# kubctl
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
# minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/
```

Starting Mini-Kube environment

```bash
# Configure MiniKube environment
minikube config set memory 4096
minikube config set cpus 2
# Start up Kubernetes Cluster
minikube start --vm-driver=virtualbox
# Testing Environment
kubectl get node/minikube -o json | jq
kubectl describe node/minikube
```

```
kubectl run metrics-k8s --image=quay.io/ryanj/metrics-k8s \
--expose --port=2015 --service-overrides='{ "spec": { "type": "NodePort" } }' \
--dry-run -o yaml > deployment.yaml
```

#### **Mesos with Mini-Mesos**
MiniMesos](https://minimesos.org/) is a popular tool for running small [Apache Mesos](http://mesos.apache.org/) cluster on your laptop.  Further documentation: http://minimesos.readthedocs.io/en/latest/

```bash
# minimesos
curl -sSL https://minimesos.org/install | sh
export PATH=$PATH:/home/joaquin/.minimesos/bin
```

#### **Mesos with dcos-vagrant**

[DC/OS](https://dcos.io/) is a distributed operating system using Apache Mesos.

```bash
vagrant plugin install vagrant-hostmanager
git clone https://github.com/dcos/dcos-vagrant
cd dcos-vagrant
cp VagrantConfig-1m-1a-1p.yaml VagrantConfig.yaml
vagrant up
```

## **Applications**

Some open source applications that you may want, edit as desired.

```bash
# Spotify
DEB_SRC="deb http://repository.spotify.com stable non-free"
DEB_REC_KEY='BBEBDCB318AD50EC6865090613B00F1FD2C19886'
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys ${DEB_REC_KEY}
echo ${DEB_SRC} | sudo tee /etc/apt/sources.list.d/spotify.list
sudo apt-get update
sudo apt-get install spotify-client
```

For Slack, visit: https://slack.com/downloads/linux

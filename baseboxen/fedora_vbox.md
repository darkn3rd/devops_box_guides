# **Fedora 28 Base Box Guide**

Created: 2018年07月22日

Tools:
* VirtualBox `5.2.16`
* Vagrant `2.0.2`
* Test Kitchen `1.22`
* Docker Machine `0.14.0`
    * Docker `18.06.0-ce`
* Minikube `0.28.1`
    * Kubectl `1.11.1`

## **VirtualBox**

```bash
# Install Repository Entry
$ sudo cat <<-'VBOXREPOENTRY' > /etc/yum/repos.d/virtualbox.repo
[virtualbox]
name=Fedora $releasever - $basearch - VirtualBox
baseurl=http://download.virtualbox.org/virtualbox/rpm/fedora/$releasever/$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc
VBOXREPOENTRY

# Upgrade Packages
sudo dnf -y update
# Test if reboot is needed
NEW_VER=$(rpm -qa kernel | sort -V | tail -n 1 | cut -d- -f2)
CUR_VER=$(uname -r | cut -d- -f1)
[[ ${NEW_VER//.} > ${OLD_VER//.} ]] \
  && echo "Kernel Updated from '${OLD}' to '${NEW}', please reboot"
```

Potentially after reboot:

```bash
# Install kernel development packages
sudo dnf install -y \
 binutils \
 gcc \
 make \
 patch \
 libgomp \
 glibc-headers \
 glibc-devel \
 kernel-headers \
 kernel-devel \
 dkms
# Install/Setup VirtualBox 5.2.x
sudo dnf install -y VirtualBox-5.2
sudo /usr/lib/virtualbox/vboxdrv.sh setup

# verify install
vboxmanage --version

# Enable Current User
usermod -a -G vboxusers ${USER}
```

## **Vagrant**


### **Install Vagrant using Hashicorp**

```bash
VER=$(
 curl -s https://releases.hashicorp.com/vagrant/ | \
 grep -oP '(\d\.){2}\d' | \
 head -1
)
PKG="vagrant_${VER}_$(uname -p).rpm"
curl -oL https://releases.hashicorp.com/vagrant/${VER}/${PKG}
sudo rpm -Uvh ${PKG}

# verify install
vagrant --version
```

### **Test Vagrant**

```bash
# Test Download of Arch
cd
mkdir myarch && cd myarch
vagrant init archlinux/archlinux && vagrant up
# install and run neofetch
vagrant ssh --command 'sudo pacman -S neofetch'
vagrant ssh --command 'neofetch'

# Test Download/Run of Gentoo
cd
mkdir mygentoo && cd mygentoo
vagrant init generic/gentoo && vagrant up
# install & run neofetch
vagrant ssh --command 'sudo emerge -a app-misc/neofetch'
vagrant ssh --command 'neofetch'
```

### **Cleanup**

```bash
######## vagrant w/ gentoo linux cleanup ########
cd
cd mygentoo
vagrant halt     # stop running vm guest
vagrant destroy  # delete vm guest entirely
######## vagrant w/ archlinux cleanup ########
cd
cd myarch
vagrant halt     # stop running vm guest
vagrant destroy  # delete vm guest entirely
```

## **TestKitchen**

### **Install TestKitchen using ChefDK**

```bash
VER=3.1.0
PKG=chefdk-${VER}-1.el7.x86_64.rpm
URL=https://packages.chef.io/files/stable/chefdk/${VER}/el/7/${PKG}
curl -O ${URL}
sudo rpm -Uvh ${PKG}

# verify install
kitchen --version
```

### **Testing TestKitchen using ChefDK**

```bash
# Generate example
chef generate cookbook helloworld && cd helloworld
# Create Ubuntu and CentOS systems
kitchen create

# Download screnfetch
wget https://github.com/KittyKatt/screenFetch/archive/master.zip
unzip master.zip
mv screenFetch-master/ ${HOME}/.kitchen/cache/

# Install pciutils on CentOS (required by screenfetch)
kitchen exec centos --command='sudo yum -y install pciutils'
# Install a snap on Ubuntu (avoids warnings w/ screenfetch)
kitchen exec ubuntu --command='sudo snap install hello-world'
# Run screenfetch script on all systems
kitchen exec default* \
 --command='sudo \
  /tmp/omnibus/cache/screenFetch-master/screenfetch-dev'
```

### **Cleanup**

```bash
######## testkitchen cleanup ########
cd
cd helloworld
kitchen destroy # destroys all test systems
```


## **Docker Machine**

### **Install Docker Machine**

```bash
VER=v0.14.0
BASE=https://github.com/docker/machine/releases/download/${VER}
curl -L ${BASE}/docker-machine-$(uname -s)-$(uname -m) \
  > /tmp/docker-machine
sudo install /tmp/docker-machine /usr/local/bin/docker-machine

# verify install
docker-machine --version
```

### **Install Docker Client**

```bash
REPOURL=https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf config-manager --add-repo ${REPO_URL}
sudo dnf install -y docker-ce

# verify install
docker --version
```

### **Test Docker Machine**


```bash
# Create a docker machine environment
docker-machine create --driver virtualbox default
# Tell docker engine to use our machine's docker
eval $(docker-machine env default)
# Run a container form docker hub
docker run docker/whalesay cowsay Hello World
```

### **Cleanup**

```bash
######## dockermachine cleanup ########
docker-machine stop       # stop vm hosting docker
docker-machine rm default # remove vm entirely
```

## **Minikube**


### **Install Minikube**

```bash
# Install minikube
BASE=https://storage.googleapis.com/minikube/releases
curl -Lo minikube ${BASE}/v0.28.1/minikube-linux-amd64 && \
 chmod +x minikube && \
 sudo mv minikube /usr/local/bin/

# verify install
minikube version
```

### **Install Kubectl**

```bash
BASE=https://storage.googleapis.com/kubernetes-release/release
VER=$(curl -s ${BASE}/stable.txt)
curl -Lo kubectl ${BASE}/${VER}/bin/linux/amd64/kubectl && \
 chmod +x kubectl && \
 sudo mv kubectl /usr/local/bin/

# verify install
printf "Kubectl Client: %s\n" $(kubectl version | awk -F\" '/Client/{ print $6 }')
```


### **Test Minikube**

```bash
# Start minikube environment
minikube start --vm-driver=virtualbox
$ # Deploy Something
kubectl run hello-minikube \
  --image=k8s.gcr.io/echoserver:1.4 \
  --port=8080
kubectl expose deployment hello-minikube \
  --type=NodePort

kubectl get pod
curl $(minikube service hello-minikube --url)
```

### **Cleanup**

```bash
######## minkube cleanup ########
minikube stop # stop kubernetes cluster
minikube rm   # remove vm hosting cluster and kubectl config entries
```

# **Pedantic ADHD Guide to DevOps ToolBox (macOS w/ Homebrew)**

by Joaquin Menchaca
Last Update: 2020年12月07日

This is my guide to getting DevOps oriented tools on [Mac OS X 10.15.7 (Catalina)](https://en.wikipedia.org/wiki/MacOS_Catalina) with [Hombrew](https://brew.sh/)

In the scope of DevOps oriented tools: these are tools used to implement DevOps philosophy, which includes tools used to build, test and deploy software that has a operational context (deployed as XaaS or supporting function).  

Such tools can include change configuration, deployment, scheduling/orchestration, build/release (such as tools supporting disciplines of CI/CD).  These tools also support both local and cloud deployment solutions for containers and virtual machines.  This is by no means, comprehensive, just popular tools in the industry since 2012.

I try my best to make these generic as possible, to use the latest version, but at any momement, the package owner can change the URLs, paths, packagename, etc.  For this reason, I am dropping comments to when it was last updated and verified, using `YYYY年MM月DD日` format (CJK characters because I am a language dork)

**Platform Tested**:

```
ProductName:	Mac OS X
ProductVersion:	10.15.7
BuildVersion:	19H15
```

## **Table of Contents**

* [Getting Started](#getting-started)
  * [AppArmor](#apparmor)
  * [Using SSH](#using-ssh)
  * [Using Git](#using-git)
* [Essential Tools](#essential-tools)
* [Shell Environment](#shell-environment)
  * [Modular Bash](#modular-bash)
  * [Shell Functions](#shell-functions)
    * [ZSH Functions](#zsh-functions)
    * [Bash Functions](#bash-functions)
* [Virtual Machines](#virtual-machines)
  * [KVM](#kvm)
  * [Virtualbox](#virtualbox)
* [Vagrant](#vagrant)
  * [vagrant-libvirt](#vagrant-libvirt)
    * [test libvirt provider](#test-libvirt-provider)
* [Docker](#docker)
  * [Docker Engine](#docker-engine)
  * [Docker Machine](#docker-machine)
    * [Docker Machine Bash Completion Scripts](#docker-machine-bash-completion-scripts)
    * [Docker Machine with KVM](#docker-machine-with-kvm)
      * [KVM Machine Driver](#kvm-machine-driver)
      * [Test Docker-Machine](#test-docker-machine)
  * [Docker Compose](#docker-compose)
* [Kubernetes](#kubernetes)
  * [Client Tools](#kubernetes-client-tools)
    * [kubectl](#kubectl)
    * [helm](#helm)
      * [Helm Diff Plugin](#helm-diff-plugin)
      * [Helm Secrets Plugin](#helmsudo apt-get install atom-secrets-plugin)
    * [helmfile](#helmfile)
  * [Platforms](#kubernetes-platforms)
    * [EKS](#eks-eksctl)
    * [AKS](#aks-az)
    * [GKE](#gke-gcloud)
    * [MiniKube](#minikube)
      * [Virtualbox Driver (Default)](#virtualbox-driver-default)
      * [KVM2 Driver](#kvm2-driver)
* [Text Editors](#text-editors)
  * [VS Code](#vs-code)
  * [Atom](#atom)
* [Language Platforms](#language-platforms)
  * [Python (pyenv)](#python-pyenv)
  * [Ruby (rbenv)](#ruby-rbenv)
  * [Node JS (nvm)](#nodejs-nvm)
  * [Go Language](#go-language)
    * [Setup GOPATH](#setup-gopath)
* [Cloud Provider Tools](#cloud-provider-tools)
  * [Google Cloud SDK](#google-cloud-sdk)
    * [Google Cloud SDK Compatible Python](#google-cloud-sdk-compatible-python)
    * [Google Cloud SDK Setup](#google-cloud-sdk-setup)
    * [Testing Google Cloud SDK](#testing-google-cloud-sdk)
  * [AWS Tools](#aws-tools)
    * [AWS CLI](#aws-cli)
    * [AWS Vault](#aws-vault)
    * [AWS Configuration](#aws-configuration)
    * [AWS Named Profile Examples](#aws-named-profile-examples)
    * [AWS Profile Helper Functions](#aws-profile-helper-functions)
  * [Azure CLI](#azure-cli)
* [Change Configuration Tools](#change-configuration-tools)
  * [Ansible](#ansible)
  * [Salt Stack](#salt-stack)
  * [CFEngine 3](#cfengine-3)
  * [Puppet](#puppet)
  * [ChefDK](#chefdk)
* [HashiCorp Tools: Howbow Dah](#hashicorp-tools-howbow-dah)
* [Applications](#applications)
  * [Spotify](#spotify)
  * [Slack](#slack)

## **Getting Started**

## **Getting Started**

### **Homebrew**

Homebrew (https://brew.sh/) is a package management system that will fetch tools and libraries from the Internet and install them locally.  Included with this is `cask` that can install full applications as well using the same underlying tools.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

For this guide, we'll create some reusable Brewfiles for use later with the `brew bundle` feature:

```
mkdir -p ~/brewfiles
```


### **Using SSH**

SSH is needed for many services including getting code or releases using SSH.  You can setup an SSH private and public key with this:

```bash
ssh-keygen -t rsa -b 4096 -C "user@example.com"
eval "$(ssh-agent -s)"

cat <<-EOF > ~/.ssh/config
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
EOF

ssh-add -K ~/.ssh/id_rsa
```

### **Using Git**

For access to source code from a git repository, like GitHub, GitLab, or BitBucket, using SSH, you need to copy your public key generated earlier and add it to the git server.  You can copy public from the command line with this:

```bash
pbcopy < ~/.ssh/id_rsa.pub
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

After configuring this public key using whatever method the git server or online service requires.  For example, if you configured a public key in your GitHub account, checkout code using SSH, e.g. `git clone git@github.com:darkn3rd/devops_box_guides.git`

## **Essential Tools**

### **GNU Tools**

macOS bundles BSD flavors of popular POSIX tools like `grep`, `sed`, and `awk` instead of GNU flavored tools popular on Linux.  This will make shell scripts that use these tools on Linux incompatible with Linux.  Therefore, we need to get such GNU tools to have a consistent environment as Linux.

Create a file `~/brewfiles/gnu.Brewfile` with the following contents:

```ruby
# Got GNU?
brew 'coreutils'
brew 'diffutils'
brew 'gnu-sed'         # GNU Sed (BSD Sed sucks)
brew 'gnu-tar'         # GNU Tar
brew 'grep'            # GNU Grep (BSD Grep sucks)
brew 'gnu-which'       # GNU Which
brew 'gawk'            # GNU Awk (BSD Awk sucks)
brew 'gnu-getopt'
brew 'findutils'  
brew 'gnutls'
brew 'bash'            # Bash v5 (Bash v3 = old)
brew 'bash-completion'
brew 'gzip'
brew 'screen'
brew 'zlib'            # req to build python

# Other popular tools
brew 'tmux'
brew 'watch'
brew 'zsh'
brew 'less'
brew 'curl'
```

Now install these packages with:

```bash
brew bundle --verbose --file ~/brewfiles/gnu.Brewfile
```

### **Adding and Changing The Shell**

Now that we have updated versions of `bash` and `zsh`, we'll want configure these to be available:

```bash
GNU_SED=/usr/local/opt/gnu-sed/libexec/gnubin/sed

grep -q /usr/local/bin/zsh /etc/shells || \
  sudo $GNU_SED -i '/^\/bin\/zsh/ i /usr/local/bin/zsh' /etc/shells

grep -q /usr/local/bin/bash /etc/shells || \
  sudo $GNU_SED -i '/^\/bin\/bash/ i /usr/local/bin/bash' /etc/shells
```

After change to your desired shell:

```bash
chsh -s /usr/local/bin/bash
```

### **Using GNU Tools by Default**

Create a paths shell file that we can source to includes these tools in our path as defaults:

```bash
cat <<-GNU_PATHS > ~/.toolbox.d/gnu_path.sh
## coreutils
export PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"
## GNU grep
PATH="/usr/local/opt/grep/libexec/gnubin:$PATH"
## GNU sed
export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
## GNU tar
export PATH="/usr/local/opt/gnu-tar/libexec/gnubin:$PATH"
## GNU which
export PATH="/usr/local/opt/gnu-which/libexec/gnubin:$PATH"
## GNU getopt
export PATH="/usr/local/opt/gnu-getopt/bin:$PATH"
## GNU findutils
export PATH="/usr/local/opt/findutils/libexec/gnubin:$PATH"
## OpenSSL
export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"
## Ncurses
export PATH="/usr/local/opt/ncurses/bin:$PATH"
## OpenLDAP tools
export PATH="/usr/local/opt/openldap/bin:$PATH"
export PATH="/usr/local/opt/openldap/sbin:$PATH"
## Curl
export PATH="/usr/local/opt/curl/bin:$PATH"
GNU_PATHS
```

Now any time you need access to these tools you can type:

```
. ~/.toolbox.d/gnu_path.sh
```


Linux is already bundled with desirable tools like GNU Sed and GNU Awk. Here are some other small tools you may want to use:

* `bat` - cat with color syntax highlighting
* `curl` - download content off web
* `gdebi` - install `.deb` packages and dependencies
* `html2` - convert `html` to a format usable with shell programming
* `inxi` - get hardware system information
* `jq` - json pretty print + filter
* `ngrep` - network grep
* `silversearch-ag` - super grep on steroids
* `sqlite3` - small sql db from binary file
* `tree` - print directory in tree view
* `xml2` - convert `xml` to a format usable with shell programming

```bash
# Basic Essentials
sudo apt install -y tree \
 xml2 \
 jq \
 sqlite3 \
 ngrep  \
 silversearcher-ag \
 curl \
 gdebi

## BAT - cat with color syntax highlighting
LATEST_BAT=$(curl -s https://api.github.com/repos/sharkdp/bat/releases | jq '[.[].tag_name][0]' -r)

pushd
wget https://github.com/sharkdp/bat/releases/download/${LATEST_BAT}/bat_${LATEST_BAT##v}_amd64.deb
sudo apt install ./bat_${LATEST_BAT##v}_amd64.deb
popd
```

## **Shell Environment**

### **Modular Bash**

This is a modular approach to manage configuration as an alternative putting everything in `~/.bash_profile`.  If you don't want to use this, then put these configurations into `.bashrc`

```bash
sed -i '/^for .*toolbox.d.*$/d' ~/.bash_profile
printf '\nfor TOOLBOX_SCRIPT in ~/.toolbox.d/*.sh;do . "${TOOLBOX_SCRIPT}";done\n' >> ~/.bash_profile
```

### **Shell Functions**

Here are some functions that can be useful for managing shell environment

#### **ZSH Functions**

```shell
function add_to_path() {
  for p in ${(s.:.)2}; do
    if [[ ! "${(P)1}" =~ "${p%/}" ]]; then
      new_path="$p:${(P)1#:}"
      export "$1"="${new_path%:}"
    fi
  done
}
```

#### **Bash Functions**

```bash
function add_to_path() {
  for path in ${2//:/ }; do
    if ! [[ "${!1}" =~ "${path%/}" ]]; then # ignore last /
      new_path="$path:${!1#:}"
      export "$1"="${new_path%:}" # remove trailing :
    fi
  done
}
```


## **Virtual Machines**

This covers virtual machines.  Only one virtual machine type can be used with Linux. Choose wisely.

### **KVM**

Still eliciting the exact requirements needed for KVM.  These are the tools thus far identified:

* `bridge-utils` - Utilities for configuring the Linux Ethernet bridge
* `cpu-checker` - Evaluate certain CPU (or BIOS) features
* `libguestfs-tools` - guest disk image management system
* `libvirt-clients`
* `libvirt-daemon-system`
* `libvirt-dev` - development files for the libvirt library
* `qemu-kvm` - QEMU Full virtualization on x86 hardware
* `virt-manager` - desktop application for managing virtual machines

```bash
sudo apt-get install -y \
 bridge-utils \
 cpu-checker \
 libguestfs-tools \
 libvirt-clients \
 libvirt-daemon-system
 libvirt-dev \
 qemu-kvm \
 virt-manager
```

Source: [Ubuntu Community Page](https://help.ubuntu.com/community/KVM/Installation):

After this, I noticed a new route and links setup:

```yaml
# ip route show
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown
# ip link | grep -A1 virbr
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:58:5f:3e brd ff:ff:ff:ff:ff:ff
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:58:5f:3e brd ff:ff:ff:ff:ff:ff
```

I also noticed new users and groups, `libvirt-qemu` and `libvirt-dnsmasq`, which look like these are used for services, and a group `libvirt` that allows users to create VMs.

```bash
virt-host-validate
```

Links:
* https://tutorialforlinux.com/2019/03/30/step-by-step-kvm-ubuntu-19-04-installation-guide/2/
* https://www.hiroom2.com/2019/06/18/ubuntu-1904-kvm-en/

### **VirtualBox**

Virtualbox will require a kernel extension to work, which will be blocked from installation unless enabled:
  System Preferences → Security & Privacy → General

More information about this is documented in an Apple Technical Note:
  https://developer.apple.com/library/content/technotes/tn2459/_index.html

```bash
brew install virtualbox
```

## **Vagrant**

```bash
brew install vagrant
```

### **Vagrant Virtualbox**

This is the default provider, so nothing needs to be configured.  You can explicitly set the default configuration (though redundant):

```bash
printf "export VAGRANT_DEFAULT_PROVIDER=virtualbox\n" > ~/.toolbox.d/vagrant.sh
```


## **Docker**

General docker related technologies.

### **Docker Desktop**

Verified: 2021年1月7日

This installs the `Docker.app` application, which will further install additional components upon launching:

* [HyperKit](https://github.com/moby/hyperkit) (variant of [xhyve](https://github.com/machyve/xhyve)) and [VPNKit](https://github.com/moby/vpnkit) virtualization solutions
* Docker CLI (client) - client `docker` command to interact with services

```bash
brew install --cask docker
```

Most users will install [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac) as it offers the smoothest experience -- invisibly handling routing through virtual network, so that the docker server is available on `localost`.  The virtual machine environment, HyperKit, is rather limited and oftentimes consumes enormous resources (cpu, memory, battery).

### **Docker Toolbox**

Verified: 2021年1月7日

Docker Toolbox is a collection of tools that includes the following:

* Docker CLI (client) - client `docker` command to interact with services
* [Docker Machine](https://github.com/docker/machine) - virtual machine manager tool to use virtualization solution of your choice to host the docker server.  Default driver is virtualbox.
* [Docker Compose](https://docs.docker.com/compose/) - popular tool to manage several Docker containers with intuitive
* [Kitematic](https://kitematic.com/) - graphical UI application that can visualize and manage docker containers

```bash
brew install docker-toolbox
```

[Docker Toolbox](https://docs.docker.com/docker-for-windows/docker-toolbox/) is an easy way to get several Docker tools installed with a single package, but it looks like [Docker Inc.](https://www.docker.com/company) is deprecating this project.  

One of the components, [Docker Machine](https://docs.docker.com/machine/), allows you to use the virtualization solution of your choice, which can include the same [HyperKit](https://github.com/moby/hyperkit) used by [Docker Desktop](https://www.docker.com/products/docker-desktop), [VMWare](https://www.vmware.com/products/fusion.html) or alternatives like [Parallels](https://www.parallels.com/), [Virtualbox](https://www.virtualbox.org/) and cloud solutions.  There is a slight higher learning curve, as the Docker server will be available from an IP address other than `localhost`.

#### **Docker Machine Bash Completion Scripts**

```bash
## ref. https://docs.docker.com/machine/install-machine/
BASE=https://raw.githubusercontent.com/docker/machine/$LATEST_MACHINE
for IDX in docker-machine-prompt.bash docker-machine-wrapper.bash docker-machine.bash
do
  sudo wget "$BASE/contrib/completion/bash/${IDX}" -P /etc/bash_completion.d
done
```

#### **Docker Machine with KVM**

Currently, neither of the two open source machine drivers `kvm` or MiniKube's `kvm2` work now with `docker-machine`:

* Boot2docker Issues:
    [1407](https://github.com/boot2docker/boot2docker/issues/1407) - `rtl8139cp` missing kernel configuration
* KVM Issues:
  * [77](https://github.com/dhiltgen/docker-machine-kvm/issues/77) (Oc 2020) - Solution https://github.com/afbjorklund/docker-machine-kvm (thank you @afbjorklund)
  * [76](https://github.com/dhiltgen/docker-machine-kvm/issues/76) (May 2020)
* KVM2 Issues:
  * [9453](https://github.com/kubernetes/minikube/issues/9453) (Oct 2020)
  * [5831](https://github.com/kubernetes/minikube/issues/5831) (Nov 2019)

In overview of the problems, the `kvm` project is no longer making releases, so users can build binary and patch locally. For `kvm2` is only supported with the embedded `docker-machine` and the configuration that is sent to it is not documented.  Currently, the default options do not work as `kvm2` creates an invalid configuration.  You can access the docker environment managed by minikube using `minikube docker-env`.

##### **KVM Machine Driver**

```bash
## ref. https://github.com/dhiltgen/docker-machine-kvm
## REF. https://blog.scottlowe.org/2017/11/24/using-docker-machine-kvm-libvirt/
LATEST=$(curl curl -s https://api.github.com/repos/dhiltgen/docker-machine-kvm/releases | jq '[.[].tag_name][0]' -r)
curl -sOL https://github.com/dhiltgen/docker-machine-kvm/releases/download/$LATEST/docker-machine-driver-kvm-ubuntu16.04
sudo mv docker-machine-driver-kvm-ubuntu16.04 /usr/local/bin/docker-machine-driver-kvm &&
  chmod +x /usr/local/bin/docker-machine-driver-kvm

## configure shell environment for kvm
printf "export export MACHINE_DRIVER=kvm\n" > ~/.toolbox.d/docker-machine.sh
```


##### **Test Docker Machine**

```bash
docker-machine create default
eval $(docker-machine env)
docker run hello-world
```

### **Docker Compose**

* **Requirements**: [python3](#python-pyenv)

Create virtualenv for docker-compose (optional):

```bash
PYTHON_VERSION="3.9.0"
pyenv virtualenv $PYTHON_VERSION docker-compose-$PYTHON_VERSION
pyenv shell docker-compose-$PYTHON_VERSIONcomponents
```

```bash
pip install docker-compose
## This installs these python modules for docker-compose 1.27.4:
##  attrs==20.2.0
##  bcrypt==3.2.0
##  cached-property==1.5.2
##  certifi==2020.6.20
##  cffi==1.14.3
##  chardet==3.0.4
##  cryptography==3.2
##  distro==1.5.0
##  docker==4.3.1
##  docker-compose==1.27.4
##  dockerpty==0.4.1
##  docopt==0.6.2
##  idna==2.10
##  jsonschema==3.2.0
##  paramiko==2.7.2
##  pycparser==2.20
##  PyNaCl==1.4.0
##  pyrsistent==0.17.3
##  python-dotenv==0.14.0
##  PyYAML==5.3.1
##  requests==2.24.0
##  six==1.15.0
##  texttable==1.6.3
##  urllib3==1.25.11
##  websocket-client==0.57.0
```

## **Kubernetes**

### **Kubernetes Client Tools**

#### **kubectl**

```bash
brew install kubectl
```

#### **Helm**

```bash
brew install helm
```
**Source**: https://helm.sh/docs/intro/install/

##### **Helm Diff Plugin**

```bash
helm plugin install https://github.com/databus23/helm-diff
```

##### **Helm Secrets Plugin**

```bash
helm plugin install https://github.com/zendesk/helm-secrets
```

#### **Helmfile**

```bash
brew install helmfile
```

### **Kubernetes Platforms**

#### **EKS (eksctl)**

**Requirements**: [AWS CLI](#aws-tools), [kubectl](#kubectl)

Amazon EKS clusters can be easily created with `eksctl`.

```bash
brew install eksctl
```

Afterward, you can create a cluster:

```bash
eksctl create cluster --name <YOUR_CLUSTER_NAME> --region "us-west-2"
```

**Source:** https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

#### **AKS (az)**

**Requirements**: [Azure CLI](#azure-cli), [kubectl](#kubectl)

```bash
export RESOURCE_GROUP=<YOUR_RESOURCE_GROUP>
export CLUSTER_NAME=<YOUR_CLUSTER_NAME>

az group create \
 --name=${RESOURCE_GROUP} \
 --location="centralus" \

az aks create --resource-group ${RESOURCE_GROUP} \
  --name ${CLUSTER_NAME} \
  --node-count 3 \
  --enable-addons monitoring \
  --generate-ssh-keys

az aks get-credentials --resource-group ${RESOURCE_GROUP} --name ${CLUSTER_NAME}
```

**Sources**
  * https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough
  * https://zero-to-jupyterhub.readthedocs.io/en/latest/microsoft/step-zero-azure.html

#### **GKE (gcloud)**

**Requirements**: [Google Cloud SDK](#google-cloud-sdk), [kubectl](#kubectl)

```bash
## Create Cluster
gcloud container clusters create <YOUR_CLUSTER_NAME> \
  --num-nodes 1 \
  --machine-type "n1-standard-2" \
  --zone "us-central1-b"
## Add K8S administrative privileges to an account
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=<your-email-address>
```

**Sources**
* https://zero-to-jupyterhub.readthedocs.io/en/latest/google/step-zero-gcp.html
* https://cloud.google.com/kubernetes-engine/docs/quickstart

#### **MiniKube**

**Requirements**: [kubectl](#kubectl)

```bash
pushd ~/Downloads
curl -sOL https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo apt install ./minikube_latest_amd64.deb
popd
```

##### **Virtualbox Driver**

```bash
minikube config set driver virtualbox
minikube start --nodes 3 --profile <YOUR_CLUSTER_NAME>
```

##### **KVM2 Driver**

```bash
## ref. https://minikube.sigs.k8s.io/docs/drivers/kvm2/
## ref. https://fedoraproject.org/wiki/How_to_debug_Virtualization_problems
minikube config set driver kvm2
minikube start --nodes 3 --profile <YOUR_CLUSTER_NAME>
```

## **Text Editors**

## **VS Code**

```bash
wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
sudo add-apt-repository \
 "deb [arch=amd64] https://packages.microsoft.com/repos/vscode \
 stable \
 main"
sudo apt update && sudo apt -y install code

# delete duplicate entries (not sure why this happens)
sudo rm /etc/apt/sources.list.d/vscode.list*
```

## **Atom**

```bash
wget -qO - https://packagecloud.io/AtomEditor/atom/gpgkey | sudo apt-key add -
sudo add-apt-repository \
 "deb [arch=amd64] https://packagecloud.io/AtomEditor/atom/any/ \
 any \
 main"
sudo apt-get update
sudo apt-get install atom
```

## **Language Platforms**

## **Common Libraries**

These are common set to tools and libraries needed for both Python and Ruby.

```bash
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get -y dist-upgrade
sudo apt -y autoremove

sudo apt-get install -y \
 build-essential \
 libffi-dev \
 libncurses5-dev \
 libncursesw5-dev \
 libreadline-dev \
 libsqlite3-dev \
 libssl-dev \
 software-properties-common \
 zlib1g-dev
```

## **Python (pyenv)**

```bash
# install pyenv
brew install pyenv pyenv-virtualenv

# setup current environment
cat <<-'PYENV' > ~/.toolbox.d/pyenv.sh
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
PYENV

. ~/.toolbox.d/pyenv.sh
```

### **Pyenv Doctor**

A useful in debugging missing libraries before installing python with pyenv is pyenv-doctor plugin:

```bash
git clone git://github.com/pyenv/pyenv-doctor.git $(pyenv root)/plugins/pyenv-doctor
pyenv doctor
```

Also see:
* https://github.com/pyenv/pyenv/wiki/Common-build-problems
* https://github.com/pyenv/pyenv-doctor

## **Installing Python**

Now you can install a desired version of Python.

```bash
## Fetch Latest Version of Python3
LATEST_PYTHON3=$(
 pyenv install --list | tr -d ' ' | grep -oP '^3\.*\d+\.\d+' | sort -V | tail -1
)

## Configure Libraries needed to compile Python
CPPFLAGS="-I$(xcrun --show-sdk-path)/usr/include"
LDFLAGS="-L$(brew --prefix openssl)/lib"
CPPFLAGS="-I$(brew --prefix openssl)/include $CPPFLAGS"
export LDFLAGS="-L$(brew --prefix readline)/lib $LDFLAGS"
export CPPFLAGS="-I$(brew --prefix readline)/include $CPPFLAGS"

## install and set desired ruby version (~8 minutes)
pyenv install $LATEST_PYTHON3
pyenv global $LATEST_PYTHON3

# upgrade pip
pip install --upgrade pip
```

## **Ruby (rbenv)**

```bash
## install rbenv
brew install rbenv ruby-build

## create rbenv environment script
cat <<-'RBENV' > ~/.toolbox.d/rbenv.sh
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@1.1)"
RBENV

## source environment script to start using rbenv
. ~/.toolbox.d/rbenv.sh
```

Install desired ruby version:

```bash
## install recent ruby
LATEST_STABLE_RUBY=$(
 rbenv install --list 2> /dev/null | grep -P '^\d\.\d+\.\d+' | sort -V | tail -1
)

## install and set desired ruby version (~12 minutes)
rbenv install $LATEST_STABLE_RUBY
rbenv global $LATEST_STABLE_RUBY

## upgrade gems and install bundler
gem update && gem install bundler
```

### **NodeJS (nvm)**

```bash
LATEST_NODE=$(curl -s https://nodejs.org/dist/ | html2 | grep -oP '(?<=/html/body/pre/a/@href=)v[0-9.]+' | sort -V | tail -1)
LATEST_NVM=$(curl -s https://api.github.com/repos/nvm-sh/nvm/releases/latest | jq -r '.name')

# From http://nvm.sh
NVM_INSTALL_SCRIPT="https://raw.githubusercontent.com/nvm-sh/nvm/${LATEST_NVM}/install.sh"
curl -so- ${NVM_INSTALL_SCRIPT} | bash
# Use NVM immediately
export NVM_DIR="${HOME}/.nvm"
[[ -s "$NVM_DIR/nvm.sh" ]] && source "$NVM_DIR/nvm.sh"

cat <<-'NVM' >> ~/.toolbox.d/nvm.sh
export NVM_DIR="${HOME}/.nvm"
[[ -s "$NVM_DIR/nvm.sh" ]] && source "$NVM_DIR/nvm.sh"
NVM
```

Install a version of Node:

```bash
LATEST_NODE=$(curl -s https://nodejs.org/dist/ | html2 | grep -oP '(?<=/html/body/pre/a/@href=)v[0-9.]+' | sort -V | tail -1)
command -v nvm &> /dev/null && nvm install $LATEST_NODE
```

Afterward, install Node modules:

```bash
npm -g install typescript coffee-script
```

### **Go Language**

```bash
LATEST_GO=$(curl -s https://golang.org/dl/ | grep -oP '(?<=go).*(?=.linux-amd64.tar.gz">)' | sort -V | uniq | tail -1)
pushd ~/Downloads
curl -sOL https://golang.org/dl/go${LATEST_GO}.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go${LATEST_GO}.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
popd

## configure shell environment
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.toolbox.d/gobin.sh
exec -l bash
```

#### **Setup GOPATH**

```bash
## configure shell environment
cat <<-'GOPATH' >> ~/.toolbox.d/gopath.sh
export PATH=$PATH:$(go env GOPATH)/bin
export GOPATH=$(go env GOPATH)
GOPATH
exec -l bash
```


## **Cloud Provider Tools**

### **Google Cloud SDK**

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
 sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | \
  sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
sudo apt-get update && sudo apt-get install google-cloud-sdk
```


#### **Google Cloud SDK Compatible Python**

* **Updated**: 2020年10月13日

Currently GCloud SDK is not compatible with latest stable version of Python with Python 3.9.0:

* https://issuetracker.google.com/issues/170716950
* https://stackoverflow.com/questions/64010263/gcloud-not-working-with-fedora33-and-python3-9

In order you use `gcloud` command, you need to use a version earlier than Python 3.9.  With `pyenv` installed, you can run:

```bash
PYTHON_VERSION=$(pyenv install --list | tr -d ' ' | grep -P '^3\.7\.\d+'| tail -1)
pyenv install $PYTHON_VERSION
```

Afterward, before using `gcloud`, you can run:

```bash
pyenv shell $PYTHON_VERSION
```

#### **Google Cloud SDK Setup**

```bash
gcloud auth login
# Gtk-Message: 21:08:27.282: Failed to load module "appmenu-gtk-module"
gcloud config set project $PROJECT
```

#### **Testing Google Cloud SDK**

```bash
gcloud compute instances list
```

### **AWS Tools**

AWS Command Line Tools and

#### **AWS CLI**

```bash
pip install awscli
## aws-cli/1.18.157 installs
##  awscli==1.18.157
##  botocore==1.18.16
##  colorama==0.4.3
##  docutils==0.15.2
##  jmespath==0.10.0
##  pyasn1==0.4.8
##  python-dateutil==2.8.1
##  PyYAML==5.3.1
##  rsa==4.5
##  s3transfer==0.3.3
##  six==1.15.0
##  urllib3==1.25.10
```

#### AWS Vault

AWS Vault can secure AWS credentials in local environment.

```bash
## ref. https://github.com/99designs/aws-vault
## article: https://99designs.com.au/tech-blog/blog/2015/10/26/aws-vault/
LATEST_AWSVAULT=$(curl -s https://api.github.com/repos/99designs/aws-vault/releases/latest | jq -r '.tag_name')
curl -sOL https://github.com/99designs/aws-vault/releases/download/$LATEST_AWSVAULT/aws-vault-linux-amd64
chmod +x aws-vault-linux-amd64 && sudo mv aws-vault-linux-amd64 /usr/local/bin/aws-vault
```

#### **AWS Configuration**

You can run your `aws configure` to setup the initial configuration, or set this up manually.

```bash
## credentials = aws access key and aws secret key
## config = default region and default output format
mkdir -p ~/.aws && touch ~/.aws/{credentials,config}

## replace REDACTED with your keys
cat <<-CREDEOF > ~/.aws/credentials
[default]
aws_access_key_id = REDACTED
aws_secret_access_key = REDACTED
CREDEOF

## replace region/output with desired values
cat <<-CONFEOF > ~/.aws/credentials
[default]
region = us-west-2
output = json
CONFEOF
```

#### **AWS Named Profile Examples**

You can have multiple profiles, AWS calls these [named profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html), for different users and levels of access.  Some use cases could include accounts for different customers, different environments (e.g. `test`, `stage`, `prod`), or different levels of access, such as S3 bucket vs. full administrator vs. kubernetes operator.

Below are some examples, using a ACME company.  Note that the `config` prepends the profile with the word `profile `, with exception to the default profile, while `credentials` does not. And yes, this is inconsistent and confusing.

**$HOME/.aws/config**:

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

**$HOME/.aws/credentials**:


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

#### **AWS Profile Helper Functions**

You can configure the current working profile with `AWS_PROFILE` (previously it was `AWS_DEFAULT_PROFILE`).

```bash
cat <<-'LISTAWS' > ~/.toolbox.d/list_aws.sh

setaws() { [[ $# -ne 0 ]] && export AWS_PROFILE=$1; }
getaws() { echo AWS_PROFILE=$AWS_PROFILE; }

listaws() {
  local PROFILES=$(grep profile ${HOME}/.aws/config | sed 's/profile //' | tr -d '[]')
  for PROFILE in ${PROFILES}; do
     if [[ ${PROFILE} == ${AWS_PROFILE} ]]; then
       SEL="=>"
     else
       SEL="  "
     fi
     printf " ${SEL} %s\n" ${PROFILE};
  done
}
LISTAWS
```

### **Azure CLI**

```bash
sudo apt remove azure-cli -y && sudo apt autoremove -y
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Once installed you can log in and do Azure stuffs:

```bash
az login
```


## **Change Configuration Tools**

Alright, I am not endorsing any particular tool, install the ones you like.

#### **Ansible**

Create virtualenv for ansible (optional):

```bash
PYTHON_VERSION="3.9.0"
pyenv virtualenv $PYTHON_VERSION ansible-$PYTHON_VERSION
pyenv shell ansible-$PYTHON_VERSION
```

Install ansible with the current python version:

```bash
pip install ansible
## This installs these components:
##  ansible==2.10.1
##  ansible-base==2.10.2
##  cffi==1.14.3
##  cryptography==3.1.1
##  Jinja2==2.11.2
##  MarkupSafe==1.1.1
##  packaging==20.4
##  pycparser==2.20
##  pyparsing==2.4.7
##  PyYAML==5.3.1
##  six==1.15.0

```

#### **Salt Stack**

Create virtualenv for ansible (optional):

```bash
PYTHON_VERSION="3.9.0"
pyenv virtualenv $PYTHON_VERSION saltstack-$PYTHON_VERSION
pyenv shell saltstack-$PYTHON_VERSION
```

Install ansible with the current python version:


```bash
pip install salt
## This installs these components:
##  certifi==2020.6.20
##  chardet==3.0.4
##  distro==1.5.0
##  idna==2.10
##  Jinja2==2.11.2
##  MarkupSafe==1.1.1
##  msgpack==1.0.0
##  pycryptodomex==3.9.8
##  PyYAML==5.3.1
##  pyzmq==19.0.2
##  requests==2.24.0
##  salt==3001.1
##  urllib3==1.25.10
```

#### **CFEngine 3**

You can fetch and install CFEngine 3.16 with:

```bash
wget -O- http://cfengine.package-repos.s3.amazonaws.com/quickinstall/quick-install-cfengine-community.sh | sudo bash
```

For a more formal installation, you can run this instead:

```bash
curl -fsSL https://cfengine-repotest.s3.amazonaws.com/pub/gpg.key | sudo apt-key add -
sudo add-apt-repository \
 "deb https://cfengine-package-repos.s3.amazonaws.com/pub/apt/packages \
 stable \
 main"
```

#### **Puppet**

* **Updated:** 2020年10月15日

If you have puppet installed already, you should remove it: `sudo apt-get purge puppet`

The instructions from Puppet are a bit unorthodox in that there's a deb package used to install requirements needed to install a deb package.  These instructions cover best practices for adding external repository (gpg key + `/etc/apt/sources.list.d/` list) and then installing the package, as well as making the tools accessible to run from the command line, something the puppet debian packages do not do.

The following components will be installed:

* `puppet-agent` (version `6.15.0-1focal`) - contains all the components to run `puppet` including embedded ruby, `facter`, `hiera`
* `puppet-bolt` (version `2.30.0-1focal`) - task runner tool that can remote execute tasks
* `pdk` (version: `1.18.1.0-1focal`) - puppet development kit used to build puppet modules

```bash
sudo apt-key adv \
  --keyserver "hkp://pgp.mit.edu" \
  --recv-keys "EF8D349F"
This system is production ready but un
# Add entry for Puppet Repository
echo "deb http://apt.puppetlabs.com $(lsb_release -c -s) puppet" \
  | sudo tee -a /etc/apt/sources.list.d/puppet.list

# Install Puppet from Puppet Repository
sudo apt-get update && sudo apt-get install -y puppet-agent puppet-bolt pdk

# Make tools accessible
for ITEM in /opt/puppetlabs/bin/*; do
  [[ -f /usr/local/bin/${ITEM##*/} ]] || \
   sudo ln -sf /opt/puppetlabs/puppet/bin/wrapper.sh /usr/local/bin/${ITEM##*/}
done
```

**NOTE**: The Marionette Collective, a parallel execution and orchestration solution, has been deprecated by Puppet.  The author had made this available as well as new project to cover the same functionality.

More information on legacy MCollective:
* [The Marionette Collective Deprecation](https://choria.io/docs/about/mcollective/)
* [Marionette Collective](https://puppet.com/docs/mcollective/current/index.html)
* [Orchestration with MCollective](https://www.linuxjournal.com/content/orchestration-mcollective)
* [puppet-mcollective](https://github.com/voxpupuli/puppet-mcollective)
* [Choria Legacy](https://github.com/choria-legacy)

#### **ChefDK**

* **Updated:** 2020年10月14日

Chef is a popular change configuration management solution that uses `ruby` to configure servers by converging to a desired state specified in the Chef scripts called recipes.   The [ChefDK](https://downloads.chef.io/chefdk) bundles several tools used with Chef, so that they do not have to be installed individually.  It comes with its own embedded Ruby.

```bash
## extract part of JavaScript to get version list and get latest version
VERS=$(
  echo "{$(curl -s https://downloads.chef.io/products/chefdk/ | \
   grep -oP '"versionList.*\](?=,)')}" | \
  jq -r '.versionList[]' | sort -V | tail -1
)

pushd ~/Downloads
curl -sOL https://packages.chef.io/files/stable/chefdk/$VERS/ubuntu/20.04/chefdk_$VERS-1_amd64.deb
sudo apt install ./chefdk_$VERS-1_amd64.deb
popd
```

If you use Knife-Zero, you can install it into Chef's embedded ruby environment:

```bash
chef exec gem install knife-zero
```


### **HashiCorp Tools: Howbow Dah**

* **Verified:** 2020年10月13日

```bash
get_latest() {
  local TYPE=${1}
  local LATEST=$(curl -s https://github.com/hashicorp/$TYPE/releases.atom | xml2 | \
   grep -oP '(?<=feed/entry/title=)v\d+.\d+.\d+$' | sort -V | tail -1
  )
  echo ${LATEST##v}
}

install_latest() {
  local TYPE=${1}
  local VERSION=$(get_latest $TYPE)
  curl -sOL https://releases.hashicorp.com/$TYPE/$VERSION/${TYPE}_${VERSION}_linux_amd64.zip
  unzip ${TYPE}_${VERSION}_linux_amd64.zip
  sudo mv $TYPE /usr/local/bin
  rm ${TYPE}_${VERSION}_linux_amd64.zip
}

pushd ~/Downloads
## Adjust list of desired tools to the ones you need
TOOLS="consul consul-template nomad packer terraform vault boundary waypoint"
for TOOL in $TOOLS; do install_latest $TOOL; done
popd
```

## **Applications**

Some open source applications that you may want, edit as desired.

### **Spotify**

* **Updated:** 2020年10月13日

```bash
# add credentials for remote repository
curl -sS https://download.spotify.com/debian/pubkey_0D811D58.gpg | sudo apt-key add -
echo "deb http://repository.spotify.com stable non-free" | \
  sudo tee /etc/apt/sources.list.d/spotify.list

# update package list and install spotify
sudo apt-get update && sudo apt-get install spotify-client
```

### **Slack**

* **Updated:** 2020年10月15日

```bash
## scrape URL from website
DOWNLOAD_URL=$(
  curl -s https://slack.com/downloads/instructions/ubuntu | \
   grep -o 'https://downloads.slack-edge.com/linux_releases/slack-desktop-.*-amd64.deb'
)

## download and isntall
pushd ~/Downloads
curl -sOL $DOWNLOAD_URL && sudo apt install ./${DOWNLOAD_URL##*/}
popd
```

# **Pedantic ADHD Guide to DevOps ToolBox: Mac OS X Version**

This is my guide to getting essential tools for DevOps on Mac OS X.

## **XCode Command Line Tools**

You need XCode command line tools.  Type `git` or something and run through instructions.

If you have all day to download the full Xcode IDE compiler, point Xcode command line tools to Xcode (don't ask):

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

## **Homebrew**

With Xcode command tools available, install [Homebrew](https://brew.sh/)

```bash
BREWRL=https://raw.githubusercontent.com/Homebrew/install/master/install
/usr/bin/ruby -e "$(curl -fsSL ${BREWRL})"
```

Optionally, you can install [Cakebrew](https://www.cakebrew.com/), a graphical front-end to Homebrew using [Cask](https://caskroom.github.io/).  Cask is a add-on to Homebrew that installs full applications, not just tools.

```bash
brew tap caskroom/cask
brew cask install cakebrew
```

## **Profiles: Oh Gawd, why, why**

* **Updated:** 2019年4月3日

Mac OS X is confusing as it doesn't follow normal pattern in regards to `.bashrc` and `.bash_profile`.  More information, see: http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html.  Generally, use `~/.bash_profile` on Mac OS X (or macOS) as `Terminal.app` uses this (does login envents each time).  One workaround is to can put everything in `~/.bashrc` just like in Linux, and then have `~/.bash_profile` source the `/.bashrc`.  This guide will instruct you to use `~/.bash_profile` as this is what is expected on Mac OS X (macOS), so you'll have to make adjustments accodringly if you want to use this workaround.

**IMPORTANT**: For `zsh` users, use `~/.zshenv` anywhere you see `~/.bash_profile` used in this guide.

This is what I do:

```bash
# Setup Environment (login script for Terminal.app)
cat <<-BASHPROFILE_EOF > ${HOME}/.bash_profile
[[ -r ~/.profile ]] && . ~/.profile  # personally use for cross shell functions
[[ -r ~/.bashrc ]] && . ~/.bashrc    # personally put configurations here
BASHPROFILE_EOF

# Shared Functions
cat <<-PROFILE_EOF >> ${HOME}/.profile
mac_colors() {
  # Prompt Dressing
  export CLICOLOR=1
  export LSCOLORS=GxFxCxDxBxegedabagaced
}
PROFILE_EOF

# enable colors, yeah baby!
cat <<-BASHRC_EOF >> ${HOME}/.bashrc
mac_colors
BASHRC_EOF

source ~/.bashrc
```

## **Path Munging**

Apple macOS (aka Mac OS X) fetches the path from `/etc/paths` and files within `/etc/paths.d`.  To access these paths in a clean way, Apple adds this to initial shell bootstrap files (`/etc/zprofile` and `/etc/profile`):

```shell
if [ -x /usr/libexec/path_helper ]; then
	eval `/usr/libexec/path_helper -s`
fi
```

If for some reason this is not available, you can add these to your environment.

As a general method to add an item, if it is not already exists in the path, you can use this snippet.


For bash:

```shell
function add_to_path() {
  for path in ${2//:/ }; do
    if ! [[ "${!1}" =~ "${path%/}" ]]; then # ignore last /
      new_path="$path:${!1#:}"
      export "$1"="${new_path%:}" # remove trailing :
    fi
  done
}
```

For zsh:

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

## **Command Line Tools**

For zsh environment, use `~/.zshenv`, wherever `~/.bash_profile` is mentioned, and use `~/.zshrc` wherever `/.bashrc` is mentioned.

### **Essential GNU Tools**

* **Updated:** 2018年5月7日

The BSD Unix tools suck compared to GNU Sed and GNU Awk.  Use GNU tools like a champ.

```bash
cat <<-BREWFILE_EOF > Brewfile
# Got GNU?
brew 'coreutils'
brew 'binutils'
brew 'diffutils'
brew 'gnu-sed', args: ['with-default-names']    # GNU Sed (BSD Sed sucks)
brew 'gnu-tar', args: ['with-default-names']    # GNU Tar
brew 'grep', args: ['with-default-names']       # GNU Grep (BSD Grep sucks)
brew 'gnu-which', args: ['with-default-names']  # GNU Which
brew 'gawk'                                     # GNU Awk (BSD Awk sucks)
brew 'findutils', args: ['with-default-names']
brew 'gnutls'
brew 'bash'                                     # Bash v5 (Bash v3 = old)
brew 'bash-completion'
brew 'zsh'
brew 'gzip'
brew 'screen'
brew 'watch'
brew 'make'
brew 'less'
BREWFILE_EOF
brew bundle --verbose
```

### **Some Other Tools**

* **Updated:** 2019年4月3日

Edit as desired:

```bash
cat <<-BREWFILE_EOF > Brewfile
# Why are these not there?
brew 'tree'             
brew 'wget'
brew 'gpg'
brew 'rsync'            # 2.x to 3.x

# Version Bumps
brew 'git'
brew 'svn'
brew 'unzip'

# Useful Tools
brew 'xml2'             # XML query tool
brew 'jq'               # JSON parse/querty/pretty tool
brew 'sqlite3'
brew 'ngrep'            # net sniffer tool
brew 'mas'              # Installer for AppStore packages
brew 'dnsmasq'          # local cache dns server
brew 'nmap'             # port scanner
brew 'dockviz'
brew 'pstree'           # process tree
brew 'the_silver_searcher'
BREWFILE_EOF
brew bundle --verbose
cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
# edit /usr/local/etc/dnsmasq.conf to add servers
#   e.g. docker-machine vm ip as docker.dev
sudo brew services start dnsmasq
```

## **Github Setup**

### **SSH Prerequisite**

Assuming you do not already have SSH keys generated.  ***NOTE***: If you need to use your private SSH key for automation, you may not want to add a passphrase to your key.  But if you do not need that, then it is best practices to add a passphrase.

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add -K ~/.ssh/id_rsa # Mac only -K for KeyChain
```

Full Instructions:
  * [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

### **Add SSH Public Key to GitHub**

First create an account on GitHub, then add your key public key to your accound:

```bash
# copy public key into Mac's clipboard
pbcopy < ~/.ssh/id_rsa.pub
# paste in appropriate area in GitHub GUI.
```

Full Instructions:
  * [Adding a new SSH key to your GitHub account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

## **Vagrant and Docker**

Developers need to test in Linux, as production systems are not on Macbooks.

```bash
# create package manifest
cat <<-BREWFILE_EOF > Brewfile
cask_args appdir: '/Applications'
cask 'virtualbox'
cask 'virtualbox-extension-pack'
cask 'vagrant'
cask 'vagrant-manager'
cask 'docker-toolbox'

brew 'docker-machine-completion'
brew 'docker-completion'
brew 'docker-compose-completion'
brew 'vagrant-completion'
BREWFILE_EOF
# install the manifest - requires privilege escalation
brew bundle --verbose
# create default docker-machine
docker-machine create default --driver virtualbox
# example download vagrant boxes
vagrant box add ubuntu/trusty64
```

## **Text Editors**

### **MacVim**

```bash
# requires Xcode IDE
# repoint Xcode Command Tools to Xcode IDE command line tools
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
# install latest MacVim
brew install macvim --env-std --with-override-system-vim
```

### **Sublime Text 3**

* **Updated:** 2018年7月9日

Sublime can be installed and accessed by the command line.  

Unfortunately, there's no command line package manager capability, so this will have to be done in the GUI.

```bash
# Sublime Text 3
brew cask install sublime-text
# Command Line Support
ln -sf "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl
```

### **Atom**

```bash
brew cask install atom
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
synced-sidebar
APMFILE_EOF
apm install --packages-file apm_packages.txt
```

### **VisualStudio Code**

Microsoft has a new open source text editor that many like as it is faster than Atom.

```bash
brew cask install visual-studio-code
cat <<-CODEFILE_EOF > code_packages.txt
bbenoist.vagrant
marcostazi.VS-code-vagrantfile
PeterJausovec.vscode-docker
haaaad.ansible
Pendrica.Chef
azaugg.vscode-cfengine
shanoor.vscode-nginx
mrmlnc.vscode-apache
luggage66.AWK
rebornix.Ruby
donjayamanne.python
luggage66.VBScript
ms-vscode.PowerShell
hdg.live-html-previewer
wcwhitehead.bootstrap-3-snippets
stevejpurves.cucumber
avli.clojure
kalitaalexey.vscode-rust
Kasik96.swift
lukehoban.Go
mjmcloug.vscode-elixir
keyring.Lua
robertohuertasm.vscode-icons
jakeboone02.cypher-query-language
eamodio.gitlens
idleberg.nsis
idleberg.nsis-plugins
idleberg.pynsist
CODEFILE_EOF
for PKG in $(cat code_packages.txt); do code --install-extension ${PKG}; done
```

### **Brackets**

[Brackets](http://brackets.io/) is a popular text editor for front-end web client programming.  With Google Chrome installed, you can do live debugging.

```bash
brew cask install brackets
# for live debugging support
brew cask install google-chrome
```

Installing modules is done from the GUI within the application.


## **Language Platforms**

### **Java Platform**

* **Updated:** 2018年5月7日

Homebrew Cask will install the latest stable version of Java, so if you need an earlier version, you have to explicitly install it.  

Below is an example of installing Java 8 (`jdk1.8.0`) and Java 10 (`jdk-10.0.1`).

```bash
cat <<-BREWFILE_EOF > Brewfile
# Java Platform Tools
cask 'java' unless system '/usr/libexec/java_home --failfast'
brew 'ant'    # classic build tools for java
brew 'groovy' # cli and alternative language for jvm
brew 'gradle' # build/doc/package tool
brew 'maven'  # build/doc/package tool
brew 'rhino'  # javascript cli using java
BREWFILE_EOF
brew bundle --verbose

# Install JDK 1.8 (Java8)
brew install "caskroom/versions/java8"

cat <<-'PROFILE_EOF' >> ${HOME}/.profile
env_groovy() {
  export GROOVY_HOME=/usr/local/opt/groovy/libexec
}

# Prereq: Maven should be installed 'brew install maven' (2016-01)
env_maven() {
  export M2_HOME=/usr/local/Cellar/maven/$(ls /usr/local/Cellar/maven | sort | head -1)
  export MAVEN_OPTS="-Xms256m -Xmx512m"
  export M2=$M2_HOME/bin
  export PATH=$M2:$PATH
}

# Prereq: Java 7 and/or Java 8 must be installed (2016-01)
setjdk() {
  if [ $# -ne 0 ]; then
   removeFromPath '/System/Library/Frameworks/JavaVM.framework/Home/bin'
   if [ -n "${JAVA_HOME+x}" ]; then
    removeFromPath $JAVA_HOME
   fi
   export JAVA_HOME=`/usr/libexec/java_home -v $@`
   export PATH=$JAVA_HOME/bin:$PATH
   export CLASSPATH=$CLASSPATH:$HOME/Classes
  fi
}

removeFromPath() {
  export PATH=$(echo $PATH | sed -E -e "s;:$1;;" -e "s;$1:?;;")
}
PROFILE_EOF

cat <<-BASHRC_EOF >> ${HOME}/.bashrc
setjdk 1.8 # or setjdk 10
env_groovy
env_maven
BASHRC_EOF
```

### **Go Lang**

Setup a Go environment with the following:

```bash
# Install Go Language
brew install go
# Setup Workspace
mkdir ~/go
echo 'export GOPATH=${HOME}/go' >> ~/.bashrc
echo 'export PATH=${GOPATH}/bin:${PATH}' >> ~/.bashrc
export GOPATH=${HOME}/go
export PATH=${GOPATH}/bin:${PATH}
```

From here, you can compile and configure code:

```bash
# Build & Run a sample program using workspace
GITHUB_USER=your_github_acccount
mkdir -p ${GOPATH}/src/github.com/${GITHUB_USER}
mkdir -p ${GOPATH}/src/github.com/${GITHUB_USER}/hello
cat <<-'HELLO_EOF' > ${GOPATH}/src/github.com/${GITHUB_USER}/hello/hello.go
package main

import "fmt"

func main() {
  fmt.Printf("Hello, world. I'm running Go!\n")
}
HELLO_EOF
go install github.com/${GITHUB_USER}/hello
hello
# Install a package using workspace
go get -u github.com/ramya-rao-a/go-outline
```

### **Rust Languge**

The [Rust](https://www.rust-lang.org/en-US/) project describes _Rust is a systems programming language that runs blazingly fast, prevents segfaults, and guarantees thread safety_.  Here's how to kick off a basic project using Rust:

```bash
# Install Rust-Lang
brew install rust
# Build & Run a sample program
cargo new hello_world --bin
cd hello_world
cat <<-'HELLO_EOF' > src/main.rs
fn main() {
    println!("Hello, world! Rust is Awesome!");
}
HELLO_EOF
cargo build
cargo run
# Install a Package
cargo install sccache # This one could take a while
```

### **Elixir Language**

Wikipedia describes [Elixir](https://elixir-lang.org/) as _a functional, concurrent, general-purpose programming language that runs on the Erlang virtual machine (BEAM). Elixir builds on top of Erlang and shares the same abstractions for building distributed, fault-tolerant applications_.

```bash
# Install Elixir
brew install elixir
# Run a sample program
cat <<-'HELLO_EOF' > hello.exs
IO.puts "Hello world from Elixir"
HELLO_EOF
elixir hello.exs
# Install a Package
mix local.hex
```


### **Perl Environment**

```bash
# Install Perl latest stable
brew install perl
# Setup Environment
PERL_MM_OPT="INSTALL_BASE=$HOME/perl5" cpan local::lib
echo 'eval "$(perl -I$HOME/perl5/lib/perl5 -Mlocal::lib)"' >> ~/.bash_profile
# Install Modules in current environment
eval "$(perl -I$HOME/perl5/lib/perl5 -Mlocal::lib)"
# Install Package Manager Wrapper
brew install cpanminus
# Install Perl Modules
cpanm YAML::XS
cpanm HTTP::Tiny
cpanm Log::Log4perl
```

### **Python**

Homebrew has had some breaking changes on at least two occasions in the last three years, so from this experience, recommend using a python manager like `pyenv`.  This may have something to do with the community at large, where some environments have `python` pointing to either `python2` or `python3`.  For this reason, I would also recommend not relying on `python` as the executable in the shebang line, and refer to the `python2` or `python3` explicitly, adding symlinks if required:

  * `!#/usr/bin/env python3`
  * `!#/usr/bin/env python2`

### **Python with pyenv**

* **Updated:** 2018年5月7日

```bash
brew install pyenv
##### Install Python2 and Python3 pythons
# https://github.com/pyenv/pyenv
pyenv install 2.7.15
pyenv install 3.6.2
##### Initialize Environment
$ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
eval "$(pyenv init -)"
##### Set python3 and python3 (first is default python)
pyenv global 3.7.2 2.7.15 # use both versions
##### Update Both Envirnoments
pip2 install --upgrade pip setuptools
pip3 install --upgrade pip setuptools
pip3 install yq     # https://yq.readthedocs.io/en/latest/
pip3 install fabric # http://www.fabfile.org/
##### VirtualEnv
# https://github.com/pyenv/pyenv-virtualenv
brew install pyenv-virtualenv
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
eval "$(pyenv virtualenv-init -)"
pyenv virtualenv 2.7.15 'my-virtual-env-2.7.15'
pyenv virtualenv 3.7.2 'my-virtual-env-3.7.2'
pyenv activate 'my-virtual-env-2.7.15'
pip install configparser xmltodict PyYAML
pyenv deactivate
pyenv activate 'my-virtual-env-3.7.2'
pip install configparser xmltodict PyYAML
pyenv deactivate
```

#### **Python 2 using Homebrew**

* **Updated:** 2018年5月7日

```bash
brew install python2

# Deprecated: `brew linkapps 'python'``
# Update SetupTools (easy_install) and PIP
#  Python 2 `pip` changed to `pip2`
pip2 install --upgrade pip setuptools
# VirtualEnv setup
pip2 install virtualenv virtualenvwrapper
mkdir ${HOME}/.virtualenvs
VIRTUALENVWRAPPER_PYTHON=$(which python2)
source /usr/local/bin/virtualenvwrapper.sh
# Install Some Libraries
sudo -H pip2 install configparser xmltodict PyYAML yq fabric
sudo -H chown -R ${USER}:admin "$(pip2 --version | awk '{print $4}')" # fix permission
```

### **Python 3 using Homebrew**

* **Updated:** 2018年5月7日

```bash
brew install python3
pip3 install --upgrade pip setuptools wheel
pip3 install virtualenv virtualenvwrapper
mkdir ${HOME}/.virtualenvs
VIRTUALENVWRAPPER_PYTHON=$(which python3)
source /usr/local/bin/virtualenvwrapper.sh
sudo -H pip3 install configparser xmltodict PyYAML yq fabric
sudo -H chown -R ${USER}:admin "$(pip3 --version | awk '{print $4}')" # fix permission
```

### **NodeJS using NVM**

* **Updated:**
  * 2018年5月7日  - nvm download
  * 2018年9月29日 - update using brew to install nvm


```bash
brew install nvm
# Source nvm script library
source $(brew --prefix nvm)/nvm.sh
# Install Noe
nvm install 11.13.0
nvm use 11.13.0
# Example of Installing Modules
npm -g install grunt mocha bower
npm -g install typescript coffee-script
```

Setup environment support for NVM

```bash
cat <<-'PROFILE_EOF' >> ${HOME}/.profile
nvm_environment() {
  source $(brew --prefix nvm)/nvm.sh
}
PROFILE_EOF

cat <<-'BASHRC_EOF' >> ${HOME}/.bashrc
nvm_environment
nvm use 11.13.0
BASHRC_EOF
```

Afterward, install Node modules:

```bash
npm -g install grunt mocha bower
npm -g install typescript coffeescript
```

### **Ruby**

The ruby installed on Mac OS X is ancient ruby from 2 years ago.  If you only need the latest stable version of ruby for ALL projects, you can use Homebrew to install it:

```bash
brew install ruby
```

You may need multiple versions of ruby as different projects will require different versions of ruby and corresponding gems installed for that version of ruby.  Two popular ruby managers are [RVM](https://rvm.io/) and [RBEnv](https://github.com/rbenv/rbenv).  Only install ONLY one of these environments. (See Below)


#### **Ruby using RVM**

Install Ruby with RVM.  RVM is robust manages the environment by manipulating the path and scanning for commands that change the directory, e.g. `pushd`, `cd`, etc.

It will automatically change the ruby environment when you switch into the directory that has an `.ruby-version` file in the directory.  

RVM also supports Gemsets with `.ruby-gemset` so that you can work on two projects that use different versions of ruby simultaneously.

```bash
# pre-req
brew install gpg
# install rvm
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
source ${HOME}/.rvm/scripts/rvm
# ruby installs take long time - download/compile
command -v rvm &> /dev/null && rvm install ruby
command -v rvm &> /dev/null && rvm install 2.3
```

Setup environment support for RVM

```bash
# environment support
cat <<-PROFILE_EOF >> ${HOME}/.profile
rvm_environment() {
  export PATH="${HOME}/.rvm/bin:${PATH}" # Add RVM to PATH for scripting
  [[ -s "${HOME}/.rvm/scripts/rvm" ]] && source "${HOME}/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*
}
PROFILE_EOF

cat <<-BASHRC_EOF >> ${HOME}/.bashrc
rvm_environment  # boostrap rvm environment
rvm use 2.3        # chose version 2.3.x
BASHRC_EOF
```

Update gems and install some gems:

```bash
sudo gem update --system
sudo -H gem install bundler rake nori inifile sqlite3
```

#### **Ruby using RBENV**

Install Ruby using RBEnv tool.  RBEnv is manages ruby versions by using aliases.  Note RBEnv is not compatible with RVM, so if you installed RVM previously, you would want to remove it first and reload your shell: `rvm implode; exec $SHELL -l`

```bash
# install rbenv
brew install rbenv
eval "$(rbenv init -)"
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
# get list of ruby versions that can be installed
rbenv install --list
# install ruby versions (whatever makes sense for your environment)
rbenv install 2.6.2
rbenv install 2.3.5
# set desired ruby version
rbenv global 2.6.2
# install bundler for depenedency management
gem update --system
gem install bundler
```

## **Server Test Tools**

These are tools that can be used to test a server.

### **Serverspec**

[ServerSpec](http://serverspec.org/) is a framework that allows utilization of *RSpec tests for checking your servers are configured correctly*.  The can execute commands locally, through SSH, WinRM, or Docker API.  

First install `ruby` on Windows (see above section), then in either Command Shell (CMD.EXE) or PowerShell run this:

```bash
gem install serverspec
# Initialize framework
serverspec-init
```

This will create a directory structure similar below, where `$HOSTNAME` is either localhost, a remote host name, or vagrant virtual guest machine name depending on what options were selected with `serverspec-init`.  Tests are organized by the hostname of the target hosts, with an example tests in `sample_spec.rb`:

```bash
.
├── Rakefile
└─── spec
    ├── $HOSTNAME
    │   └── sample_spec.rb
    ├── README.md
    └── spec_helper.rb
```

As an example, a test would look something like this:

```ruby
# Test Vagrant Image
describe 'Vagrant Image Tests' do
  describe user('vagrant') do
    it { should exist }
    it { should belong_to_group 'vagrant' }
  end
end
```

After crafting some tests, you can run them using `rake spec` command, or manually using `TARGET_HOST=$HOSTNAME rspec` command.

### **InSpec**


[InSpec](https://www.inspec.io/) is a framework to create infrastructure tests for integration or compliance testing and is marketed as *compliance as code*.  The tests are organized in a test profile, which can be initially created with `inspec init profile NAME`.  

Inspec uses similar syntax to ServerSpec, but is not feature parity with ServerSpec.  Execution of test scripts is done manually using `inspec` command.  There is no further automation to orchestrate tests directly [Vagrant](https://www.vagrantup.com/) or [Docker](https://www.docker.com/), so this will have to be created manually.  There is integration through [TestKitchen](http://kitchen.ci/) tool.

First install `ruby` on Windows (see above section), then in either Command Shell (CMD.EXE) or PowerShell run this:

```bash
gem install inspec
# Initialize profile
inspec init profile $PROFILENAME
```

This will create the following structure, with some example tests in `example.rb`
```bash
.
└─── $PROFILENAME
    ├── controls
    │   └── example.rb
    ├── inspec.yml
    ├── libraries
    └── README.md
```

As an example, a control could look something like this:

```ruby
# Test Vagrant Image
control 'vagrant-image' do
  title 'Vagrant Image Tests'
  desc 'Test the vagrant image requirements'
  describe user('vagrant') do
    it { should exist }
    its('groups') { should include('vagrant') }
  end
end
```

After crafting some tests, you can run them using `inspec exec` command, for example:

```bash
inspec exec $PROFILENAME/ \
        -t ssh://$TARGET_USER@$TARGET_HOST \
        -p $TARGET_PORT \
        -i $TARGET_IDENTITYFILE
```

## **Advanced Tools**

### **Change Configuration Tools**

* **Updated:** 2018年9月22日


Alright, I am not endorsing any particular tool, install the ones you like.

```bash
# Ansible
sudo -H pip install ansible
sudo -H chown -R ${USER}:admin "$(pip --version | awk '{print $4}')"

# Salt Stack
sudo -H pip install salt
sudo -H chown -R ${USER}:admin "$(pip --version | awk '{print $4}')"

# CFEngine
brew install cfengine

# Puppet 5.x
brew tap puppetlabs/puppet
brew cask install pdk
brew cask install puppet-bolt
brew cask install --appdir='/Applications' puppet-agent
```


#### **ChefDK**

* **Updated:** 2018年6月7日 Homebrew deprecated ChefDK out of their system.


```bash
# Chef DK
brew tap chef/chef
brew cask install chefdk
```

If you use default ruby or really do not care about ruby versions and conflicts, you can use these helper scripts in your environment to setup ChefDK.  The `  eval "$(chef shell-init bash)"` is not compatible with RBenv or RVM.

```bash
# Note: This conflicts w/ RVM -
#  suggest commenting out one or other depending on usage
cat <<-PROFILE_EOF >> ${HOME}/.profile
env_chefdk() {
  eval "$(chef shell-init bash)"
}
PROFILE_EOF

cat <<-BASHRC_EOF >> ${HOME}/.bashrc
env_chefdk
BASHRC_EOF
```


##### **ChefDK with RBEnv**

* **Updated:** 2018年6月7日 fixes


If you use RBEnv, you can use [rbenv-chefdk](https://github.com/docwhat/rbenv-chefdk), as per instructions:

```bash
brew install rbenv-chefdk
mkdir "$(rbenv root)/versions/chefdk"
rbenv shell chefdk
rbenv rehash
```

If you have a code repo Chef recipes that are compatible with ChefDK, you can put in `.ruby-version` with the version of `ext-chefdk-ruby`.


##### **ChefDK with RVM**

This snippet is from: https://gist.github.com/asnodgrass/36d88ffcb76b4068c62c.  This will have to be carefully updated with newer versions of ChefDK, as the embedded ruby version will change.

```bash
CHEFDK="/opt/chefdk/embedded"
CHEFDK_USER="$HOME/.chefdk/gem/ruby/2.4.2"
RVM_GEMS="$HOME/.rvm/gems"
RVM_RUBIES="$HOME/.rvm/rubies"
RUBY_NAME="ext-chefdk-ruby"

mkdir -p $RVM_RUBIES/$RUBY_NAME
ln -s $CHEFDK/bin $RVM_RUBIES/$RUBY_NAME
ln -s $CHEFDK_USER $RVM_GEMS/$RUBY_NAME
ln -s $CHEFDK/lib/ruby/gems/2.4.2 $RVM_GEMS/$RUBY_NAME\@global
rvm alias create chefdk $RVM_NAME
rvm use chefdk
```

If you have a code repo Chef recipes that are compatible with ChefDK, you can put in `.ruby-version` with the version of `ext-chefdk-ruby`.

### **Google Cloud Tools**

Google's command line tools require that you have access to a web browser and a google account with billing enabled.  You will need to log into your account, which will enable access using OAuth.  

You'll need to create credentials to access the account, using `gcloud init`, which will access Chrome in a graphical environment, otherwise for a console only environment, you'll have to use `gcloud init --console-only` and following the instructions using a Chrome on another system.

Google Cloud organizes and segregates access to resources into [projects](https://cloud.google.com/storage/docs/projects).  A default one is autogenerated, but you can create others such as `mycompany-staging`, `mycompany-devtest`, `mycompany-production`.

```bash
# Install GCloud Tools
brew cask install google-cloud-sdk

# Create/Set A Project
MY_PROJECT="zerg-rush-210739"
# Use `gcloud init`, which will access the web browser, otherwise, in console:
gcloud init --console-only
gcloud projects create "${MY_PROJECT}"
gcloud config set project "${MY_PROJECT}"
```

#### **Multiple Account Configurations**

You can create and access multiple accounts.

```bash
ACCOUNT="other-account"
cat <<-CONFIG_EOF >> ${HOME}/.config/gcloud/configurations/config_${ACCOUNT}
[core]
account = taldarim@gmail.com
project = protoss-stalker-568143
CONFIG_EOF
gcloud config configurations activate ${ACCOUNT}
gcloud auth login
```

### **AWS Tools**


#### **AWS CLI**

AWS Command Line Tools and [AWS Vault](https://github.com/99designs/aws-vault).  

Updated this to explicitly use `pip3` on Mac OS X.

```bash
sudo -H pip3 install awscli
sudo -H chown -R ${USER}:admin "$(pip3 --version | awk '{print $4}')"
```

#### **AWS CDK (Cloud Dev Kit) for TypeScript, Java, C#**

AWS added [AWS CDK](https://github.com/awslabs/aws-cdk) for TypeScript, Java, and C# languages.  

**Prerequisite**: [NodeJS](#nodejs-using-nvm).

```shell
npm i -g 'aws-cdk'
mkdir hello-cdk
cd hello-cdk
cdk init app --language=typescript # (or --language=java, ...)
cdk deploy
```

#### **AWS SDK for JavaScript**

**Prerequisite**: [NodeJS](#nodejs-using-nvm).


```shell
npm i -g 'aws-sdk'
```

#### **AWS SDK for Ruby**

**Prerequisite**: [Ruby](#ruby).

```shell
gem install 'aws-sdk'
```

#### **AWS SDK for Go Language**

**Prerequisite**: [Go Lang](#go-lang).


```shell
go get -u 'github.com/aws/aws-sdk-go'
```

#### **AWS Profile Management**

```bash
mkdir -p ${HOME}/.aws
# configure environment(s)
touch ${HOME}/.aws/credentials
touch ${HOME}/.aws/config
# OPTIONAL: manage secrets w/ keychain
brew cask install aws-vault
# OPTIONAL: AWS Profile Functions
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
cat <<-BREWFILE_EOF > Brewfile
brew 'consul'
brew 'consul-template'
brew 'terraform'
brew 'nomad'
brew 'packer'
brew 'packer-completion'
brew 'vault'
BREWFILE_EOF
brew bundle --verbose
```

### **Orchestration Tools**

Play with orchestration tools on Mac OS X with [MiniMesos](https://minimesos.org/) and [MiniKube](https://github.com/kubernetes/minikube)

```bash
brew cask install minikube
brew install minimesos
```

## **GitOps**

### **Jenkins/X Client**

* **Added:** 2018年9月22日

```bash
cat <<-BREWFILE_EOF > Brewfile
tap  'jenkins-x/jx'
brew 'jx'
BREWFILE_EOF
brew bundle --verbose
```


## **Applications**

Some open source applications that you may want, edit as desired.

```bash
cat <<-BREWFILE_EOF > Brewfile
cask_args appdir: '/Applications'
cask 'firefox'        # browser
cask 'google-chrome'  # browser
cask 'spotify'        # music
cask 'slack'          # communication
cask 'vlc'            # media files

cask 'pycharm'        # JetBrain's PyCharm
cask 'rubymine'       # JetBrain's RubyMine/NodeJS IDE
cask 'intellij-idea'  # JetBrain's Java IDE
BREWFILE_EOF
brew bundle --verbose
```

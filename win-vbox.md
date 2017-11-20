# **Pedantic ADHD Guide to DevOps ToolBox (Windows with Virtualbox)**

This is my guide to getting essential tools on Windows.  This guide uses [Virtualbox](https://www.virtualbox.org/) for virtualization.  For essential GNU tools like `bash`, `grep`, `sed`, and `awk`, as well as compilers needed for Ruby on Windows, [MSYS2](http://www.msys2.org/) is used.

## **DevOps**

DevOps can be described as *"the emerging professional movement that advocates a collaborative working relationship between Development and IT Operation"* (Gene Kim).  I personally see DevOps as a way to break down the previous silos between Development and Operations related to the nature that once desktop installed applications are now in cloud services.  Developing applications independently without collaboration leads to a lot of misalignment, inefficiencies, increased costs, and failures.  

From this movement, there's a whole new set of tools that help in automating creation of cloud resources (systems, network, persistence, etc.) and developing, testing, deploying applications into systems. The systems themselves can be physical, virtual, containers, orchestration platforms, or serverless functions.  Below are some tools that may help in supporting these activities.

- Joaquin Menchaca (July 2017)

## **Testing**

These have been tried on Windows 10 Professional (64-bit).

## **Chocolately**

[Chocolately](https://chocolatey.org/) is a package manager install tool that can fetch packages from the Internet to install on your system.  It uses [.Net](https://www.microsoft.com/net/) and [NuGet](https://www.nuget.org/) for its magic.

### **Chocolately Install Instructions**

These instructions just parrot [Chocolately Install instructions](https://chocolatey.org/install).

1. Open PowerShell in Administrative mode.
2. Run The following:

```PowerShell
Set-ExecutionPolicy AllSigned
$InstallScript = 'https://chocolatey.org/install.ps1'
iex ((New-Object System.Net.WebClient).DownloadString($InstallScript))
```

## **Essential GNU Environment**

This gives you recent versions of bash, gawk, gnu sed, and perl.

```PowerShell
choco install msys2
```

### **Essential Tools**

This will give you an ssh client, git, rsync (useful for vagrant), and a few other commonly used tools.

Open the MSYS2 application: `C:\tools\msys64\msys2_shell.cmd`

```bash
# Essential Tools
pacman --noconfirm -S openssh git rsync
# Other Common Tools
pacman --noconfirm -S wget tree sqlite3 bc
```

### **Create NTFS Junctions**

For easy access between both MSYS2 and Windows environments, you can create NTFS junctions to each other, which behave like symbolic links when in bash.  To do this, in Windows Command Shell (in Administrator mode):

```batch
:: Create a Junction to MSYS2 HOME directory
mklink /J msyshome C:\tools\msys64\home\%USERNAME%
:: Junction to openssh keys
mklink /J .ssh msyshome\.ssh
:: Create a Junction back to Windows directory
cd msyshome
mklink /J winhome %USERPROFILE%
```


### **Github Setup**

You must do this in MSYS2 environment.

#### **SSH Prerequisite**

Assuming you do not already have SSH keys generated.  ***NOTE***: If you need to use your private SSH key for automation, you may not want to add a passphrase to your key.  But if you do not need that, then it is best practice to add a passphrase.

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

Full Instructions:
  * [Generating a new SSH key and adding it to the ssh-agent]https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-windows)

#### **Add SSH Public Key to GitHub**

First create an account on GitHub, then add your key public key to your account:

```bash
# copy public key into Windows' clipboard
clip < ~/.ssh/id_rsa.pub
# paste in appropriate area in GitHub GUI.
```

Full Instructions:
  * [Adding a new SSH key to your GitHub account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/#platform-windows)


## **Vagrant and Docker**

1. Open PowerShell in Administrative mode, and run the following
   ```PowerShell
   choco install virtualbox
   choco install vagrant
   choco install docker-toolbox
   ```
2. Navigate to the [Virtualbox Downloads](https://www.virtualbox.org/wiki/Downloads) and download **Virtualbox Extension Pack**.  Make sure that any anti-virus is temporarily shut off for this next step.  Double-click on downloaded file, e.g. `Oracle_VM_VirtualBox_Extension_Pack-5.1.22-115126.vbox-extpack`.
3. With, vagrant should be installed into `C:\HashiCorp\Vagrant\bin`, in the MSYS2 application (`C:\tools\msys64\msys2_shell.cmd`), run:
   ```bash
   echo 'PATH=${PATH}:/c/HashiCorp/Vagrant/bin/' >> ${HOME}/.bash_profile
   export PATH=${PATH}:/c/HashiCorp/Vagrant/bin/
   ```
4. Download Ubuntu image (this will take some time, so do it when convenient), in MSYS2, run:
   ```bash
   vagrant box add ubuntu/trusty64
   ```
5. Create Docker Machine Environment
   ```PowerShell
   docker-machine create default --driver virtualbox
   ```

## **Windows Tools**

```Powershell
choco install -y sysinternals
choco install -y winrar
choco install -y 7zip.install
```

## **Web Browsers**

```Powershell
choco install -y googlechrome
choco install -y firefox
```

## **Text Editors**

In no particular order, here's some of the text editors I use or have used on Windows over the years.

### **Notepad++**

```PowerShell
choco install -y notepadplusplus
```

### **Sublime Text**

Sublime is extremely popular general purpose cross-platform text editor.


```PowerShell
choco install -y sublimetext3
```

### **Brackets**

Brackets is popular for web client or front-end development.  It has a live preview option, which requires GoogleChrome.

```PowerShell
choco install -y brackets
```

Installing modules is done from the GUI within the application.  It can be done at the command, but process may very slightly:

```batch
SET EXTENSIONS=%HOMEDRIVE%%HOMEPATH%\AppData\Roaming\Brackets\extensions\user

cd /D %EXTENSIONS%
git clone git@github.com:ivogabe/Brackets-Icons.git
cd Brackets-Icons
npm install -g gulp
npm install

cd /D %EXTENSIONS%
git clone https://github.com/zaggino/brackets-npm-registry.git
cd brackets-npm-registry
npm install

popd
```

### **VisualStudio Code**

A recent general purpose cross-platform text editor with robust extension support.  The extensions can be installed through the GUI or command line.

```PowerShell
choco install -y visualstudiocode
# Refresh PATH to include `code`
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") `
  + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
# Install Some Extensions
#  Example Extensions (tailor to your preferences)
@'
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
ms-vscode.PowerShell
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
'@ | Out-File -Encoding "UTF8" $home/code_ext_tmp.txt
#####
# Filter out comments (PowerHell Way)
##########
Select-String -NotMatch '^#' $home/code_ext_tmp.txt  | %{ $_.Line } |
  Out-File -Encoding 'UTF8' $home/code_ext_lst.txt
#####
# Install Editor Packages (one at a time)
##########
Get-Content  $home/code_ext_lst.txt |
  Foreach-Object { code --install-extension $_ }
#####
# Cleanup
##########
Remove-Item code_ext_*.txt
```

### **Atom**

Atom is becoming popular general purpose cross-platform text editor.  Atom has command-line plug-in manager tool called `apm`.

In PowerShell, run:

```PowerShell
# Install Atom
choco install -y atom
# Refresh PATH to include `atom` and `apm`
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") `
   + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
# Install Some Modules
@'
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
'@ | Out-File -Encoding "UTF8" $home/apm_pkg_tmp.txt
#####
# Filter out comments (PowerHell Way)
##########
Select-String -NotMatch '^#' $home/apm_pkg_tmp.txt  | %{ $_.Line } |
  Out-File -Encoding 'UTF8' $home/apm_pkg_lst.txt
#####
# Install Editor Packages from file list
##########
apm install --packages-file $home/apm_pkg_lst.txt
#####
# Cleanup
##########
Remove-Item apm_pkg_*.txt
```

## **jEdit**

[jEdit](http://jedit.org/) classic text editor that runs in Java VM.

```PowerShell
choco install -y jedit
```

## **Language Platforms**

### **C/C++ Tool Chain**

These are C/C++ tools used to compile applications.

#### **Visual C++ 2005 Build Tools**

```PowerShell
choco install -y vcbuildtools
```

#### **MSYS2 Mingw ToolChain**

Open MSYS2:

```bash
pacman -S base-devel mingw-w64-x86_64-toolchain
```

### **Go Language**

Fetch go language from https://golang.org/dl/.  In MSYS2 you can do something like this:

```bash
curl -O https://storage.googleapis.com/golang/go1.9.1.windows-amd64.msi
```

**NOTE**: Alternatively you can install with Chocolatey using `choco install -y golang`.  This package uses the zip installer, which is known to be slow, ten minutes in this case, and the version lags behind. To get recent [security fixes](https://github.com/golang/go/issues?q=milestone%3AGo1.9.1), you need to install from the source.

Using **MSYS2**, you can do the following:

```bash
# Current Environment
export GOPATH=/c/Users/${USER}/go
export PATH=${PATH}:/c/Go/bin/:${GOPATH}/bin
# One Time Setup Only
mkdir ${GOPATH}
echo 'export GOPATH=/c/Users/${USER}/go' >> ~/.bash_profile
echo 'export PATH=${PATH}:/c/Go/bin/:${GOPATH}/bin' >> ~/.bash_profile
```

From here, you can do a small test compilation:

```bash
GITHUB_USER=your_github_acccount
mkdir -p ${GOPATH}/src/github.com/${GITHUB_USER}
mkdir -p ${GOPATH}/src/github.com/${GITHUB_USER}/hello
cat <<-HELLO_EOF > ${GOPATH}/src/github.com/${GITHUB_USER}/hello/hello.go
package main

import "fmt"

func main() {
  fmt.Printf("Hello, world. I'm running Go!\n")
}
HELLO_EOF
go install github.com/${GITHUB_USER}/hello
hello
```

**References**: This was adapted from:
  * [How to Write Go Code: Workspaces](https://golang.org/doc/code.html#Workspaces)
  * [Getting Started with Golang - Windows](https://github.com/abourget/getting-started-with-golang/blob/master/Getting_Started_for_Windows.md)


### **Java Platform**

```PowerShell
choco install -y jdk8
choco install -y ant
choco install -y maven
choco install -y groovy gradle
```

### **ActivePerl**

```PowerShell
choco install -y activeperl
# Refresh Path
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
# Install Modules
ppm install Log-Log4perl
ppm install Dancer
```

### **Python 2.7**

```PowerShell
choco install -y python2
# Refresh PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" +  [System.Environment]::GetEnvironmentVariable("Path","User")
# update and essential tools
pip install --upgrade pip setuptools
pip install virtualenv virtualenvwrapper
# other tools
pip install configparser xmltodict PyYaml
```

### **Node**

```PowerShell
choco install -y nodejs-lts
# Refresh PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") `
   + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
# Install Modules
npm -g install grunt mocha bower
npm -g install typescript coffee-script
```

### **Ruby 2.4 via RubyInstaller**

```PowerShell
choco install ruby
# Refresh PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
# Install Gems
gem update
gem install bundler rake nori inifile sqlite3
```

#### **Ruby 2.4 DevKit**

In order to install Ruby gems with native extensions in Ruby 2.4+, you'll need to have MSYS2 install and go through instructions in [Ruby Development Kit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit).

For these tools to work, as per [documentation](https://github.com/oneclick/rubyinstaller2), open up MSYS2 and run:
```bash
pacman -S base-devel mingw-w64-$(uname -i)-toolchain
```

In PowerShell (`powershell.exe`) or Windows Command shell (`cmd.exe`), you can enable MSYS2 tools with this: `ridk enable`

#### **Ruby 2.0-2.3 DevKit**

```PowerShell
choco install -y ruby2.devkit
# Refresh PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
cd C:\tools\DevKit2
ruby dk.rb init
'- C:\tools\ruby23' | Out-File -Append -Encoding "UTF8" config.yml
ruby dk.rb install
```

#### **Test Rails**

Run these below and test http://127.0.0.1:3000/ in your web browser of choice.

```PowerShell
gem install rails
rails init $home/myweb
cd $home/myweb
bundle install
rails server
```

## **Hashicorp Tools**

### **Packer**

Install Packer using Chocolatey:

```PowerShell
choco install -y packer
```

You can make this available in MSYS2 with this:

```bash
PACKER_PATH="/c/ProgramData/chocolatey/lib/packer/tools/packer.exe"
ln -s ${PACKER_PATH} /usr/bin/packer.exe
```

## **Applications**
Some open source applications that you may want, edit as desired.

```PowerShell
# IDEs
choco install -y pycharm-community
choco install -y intellijidea-community
# Apps/Tools
choco install -y slack
choco install -y spotify
choco install -y adobereader
```

### **LDAP**

#### **JXplorer**

This is an classic LDAP explorer that runs on Java VM:

* http://www.jxplorer.org/downloads/users.html

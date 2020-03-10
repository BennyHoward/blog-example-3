# Development Environment Setup

Welcome to the development environment setup guide.  
In here you'll learn how to setup your machine in order to run and develop on this project.  

This development environment setup assumes the developer is using macOS.  
Windows and Linux instructions are currently not available at the moment, however it is a [TODO](./.todo) item.  

## 1. Install Xcode Select

Xcode Select is Apple's development tools bundle.  
One of the needed tools is [git](https://git-scm.com/), which will be installed with this (among other things).  

Install via the following command:  

```sh
xcode-select --install
```

## 2. Install Homebrew

Homebrew is a 3rd-party package manager for macOS.  
It's a command line tool for installing various packages via the command line.  
It's similar to `apt`, `yum`, `apk`, `emerge`, etc. for some distros of Linux.  

Refer to the [Homebrew](https://brew.sh/) site for more information.  

Install via the following command:  

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

**NOTE**:  You may find instruction to install `cask`.  But Homebrew now comes with `cask` without needing to explicitly install it.  
**NOTE**:  There is a Linux (and Windows Subsystem for Linux) version of Homebrew, called [Linuxbrew](https://docs.brew.sh/Homebrew-on-Linux).  However, `cask` packages won't be available.  

## 3. Install ASDF VM via Homebrew

[ASDF VM](https://asdf-vm.com/#/plugins-all?id=plugin-list) is a runtime and developer tools version management tool.  In short, think of it as something like [Node.js Version Manager (NVM)](https://github.com/nvm-sh/nvm) and [Ruby Version Manager (RVM)](https://rvm.io/), but runtime agnostic.  It accomplishes this via plugins.  So after you install ASDF VM you'll need to install plugins for the various runtimes.  

Why use this?  
Because, one of the most frustrating things is getting a consistent development environment across different machines.  
No more (mostly) "it works on my machine"-isms.  
I can't tell you how many hours teams I've been on has wasted over the years just getting someone's code to just run.  

Install via the following command:  

```sh
brew install asdf
```

Then you'll need to update you're system `PATH`.  
Assuming you're using Z-shell, you'll need to update the `~/.zshrc` file.  
Update that file by running the following command:  

```sh
echo 'source $(brew --prefix)/opt/asdf/asdf.sh' >> ${HOME}/.zshrc
```

**NOTE**: The single quote is important here.  
**NOTE**: You may also need to source the `~/.zshrc` file for the system changes to take effect.  

### 3.1 ASDF VM Node.js plugin

Since you'll be installing Node.js via ASDF VM, you'll need to install the Node.js plugin first.  

However, there are prerequisites needed by the plugin to install any given version of Node.js.

Install the prerequisites via the following command:  

```sh
brew install coreutils gpg
```

Install the plugin via the following command:  

```sh
asdf plugin-add nodejs
bash ${HOME}/.asdf/plugins/nodejs/bin/import-release-team-keyring
```

### 3.2 ASDF VM Python plugin

Since you'll be installing Python via ASDF VM, you'll need to install the Python plugin first.  

Install via the following command:  

```sh
asdf plugin-add python
```

### 3.3 Install the Node.js and Python runtimes via ASDF VM

In the base directory of the project there is a file `.tool-versions` file.  
This is the configuration file used by ASDF VM to both install the missing runtime version, but also use the appropriate runtime restricted the the current directory and all subdirectories in this project (assuming there aren't any additional `.tool-versions` files in the subdirectories).  

Also, ensure that the appropriate ASDF VM plugins and prerequisites have been installed before installing the needed runtimes via ASDF VM.  

To install the missing runtimes (if their missing), you'll need to navigate to the base directory of this project and run the following command:

```sh
asdf install
```

### 3.4 Post-install steps for Node.js

Even though you now have the runtimes installed, you'll need to perform some final steps to properly setup Node.js.  

Install `node-gyp` globally.  In the event that some Node.js dependencies require native extensions to be compiled, you'll need `node-gyp` (and Python2).  
Install `node-gyp` globally with the following command:

```sh
npm install --global node-gyp
```

**NOTE**: Notice that `sudo` wasn't used for the global installation.  That is because ASDF VM doesn't install Node.js in system directories which may require permissions.  
**NOTE**: Also, configurations and global installs that are applied to runtime typically don't carry over to another installed version.  This is because ASDF VM treats different installed versions as septate software independent of each other.  The only exception to this would be runtimes that store configurations or plugins or etc. in directories that ASDF VM doesn't control.  For example, local NPM dependencies stored in project `node_module` folders.  

### 3.5 Post-install steps for Python

*No post installation steps.*

**NOTE**: Also, configurations and global installs that are applied to runtime typically don't carry over to another installed version.  This is because ASDF VM treats different installed versions as septate software independent of each other.  The only exception to this would be runtimes that store configurations or plugins or etc. in directories that ASDF VM doesn't control.  For example, local virtual environments created by Python's `virtualenv` or `venv` modules.  

## 4. Globally install the NestJS CLI via NPM

This project uses the NestJS framework.  Therefore, it depends on the NestJS CLI, which needs to be installed globally.  

Ensure that you've installed Node.js via ASDF VM and you're currently using it, such as having you're working directory be in this project's directory (and/or subdirectories).  

To install it, do so via the following command:

```sh
npm install --global @nestjs/cli
```

## 5. Install Virtualbox, Docker, and Docker Compose via Homebrew

Because the Docker engine cannot natively run on macOS, you'll need to install Virtualbox and run Docker through a Linux virtual machine.  Sounds complicated?  That's because it is, but only in terms of how it works, in reality using it is as smooth as if you have it working natively with the system.  

Install Virtualbox via the following command:

```sh
brew cask install virtualbox virtualbox-extension-pack
```

**NOTE**:  You'll be prompted to enter you're password.  

Docker Machine is a tool used to setup the Linux virtual machine so that Docker can be used.  
You'll also need Docker Machine NFS.  Unfortunately, mounting volumes in with Docker tends to have file permission issues, such as non-root users in the Docker container sometimes not being able to write to folders mounted to the Docker container (e.g. `docker ... --volume ...`).  Fortunately, Docker Machine NFS fixes this.
  
Install Docker and Docker Machine via the following command:  

```sh
brew install docker docker-machine docker-machine-nfs
```

**NOTE**:  Docker Compose comes with the Homebrew `docker` package.  

### 5.1 Create and configure the Virtualbox Docker machine

Create the Docker Machine called `vbox-blogexample3` (actually you can name it whatever you want, but we're calling it this for our purposes) via the following command:  

```sh
docker-machine create --driver virtualbox vbox-blogexample3
```

Then you'll need to update you're system's environment variables.  
Assuming you're using Z-shell, you'll need to update the `~/.zshrc` file.  
Update that file by running the following command:  

```sh
cat <<'SHELL' >> ${HOME}/.zshrc
if [[ $(docker-machine status vbox-blogexample3) == 'Running' ]]; then
  eval "$(docker-machine env vbox-blogexample3)"
fi
SHELL
```

**NOTE**: The single quote is important here.  
**NOTE**: You may also need to source the `~/.zshrc` file for the system changes to take effect.  
**NOTE**: With system restarts you may get an error like this: `docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.`.  To fix this just start the Docker machine with this command: `docker-machine start vbox-blogexample3` and source the `~/.zshrc` file again.  

Activate NFS on the project folder so the MongoDB can mount project folders properly with the correct permissions.  
For more information on Docker Machine NFS refer to the [documentation](https://github.com/adlogix/docker-machine-nfs) found on it's README file.  

We're altering this Virtualbox machine to be used to mount volumes in the project directory using Docker Machine NFS.  This will avoid permissions issues with between the Docker container and host folders, however this will mean that mounting folders outside this project won't function properly.  Therefore, it's necessary to create a new Docker machine exclusive to this project.  Don't worry about eating up too much storage on your machine, the Virtualbox Docker machines are relatively cheap on resources, taking up ~1GB of memory, 1 CPU core, and ~70MB per machine.  

```sh
docker-machine-nfs vbox-blogexample3 --shared-folder=$(pwd) --nfs-config="-alldirs -maproot=0"
```

**NOTE**: You wil be asked for you password.  This is because Docker Machine NFS needs root permissions to edit `/etc/exports`.  

### 5.2 Create the Docker network

We'll need to create a Docker network so that the containers can comminate with each other via their hostname, leveraging Docker's builtin DNS.  

Create Docker network `blog-example-3` via the following command:  

```sh
docker network create blog-example-3
```

### 5.3 Download the needed base images

Pull the official Node.js Docker image, `node:10.19.0-buster-slim`, via the following command:  

```sh
docker pull node:10.19.0-buster-slim
```

**NOTE**: Documentation on how to use this container can be found [here](https://github.com/nodejs/docker-node/blob/master/README.md#how-to-use-this-image).  

Pull the official MongoDB Docker image, `mongo:4.2.3-bionic`, via the following command:  

```sh
docker pull mongo:4.2.3-bionic
```

**NOTE**: Documentation on how to use this container can be found [here](https://hub.docker.com/_/mongo/).  Look for the "How to use this image" section.  

Pull the official Elasticsearch Docker image, `elasticsearch:7.6.0`, via the following command:  

```sh
docker pull elasticsearch:7.6.0
```

**NOTE**: This image is based on [CentOS 7](https://hub.docker.com/_/centos).  
**NOTE**: Documentation on how to use this container can be found [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docker.html).  
**NOTE**: Documentation on how to use Elasticsearch can be found [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index.html).  

## Optional Developer Tools

While not strictly required for this project operate properly, they are nice to haves that'll make development much smother.  

### Install Visual Studio Code

It's always good to have an IDE.  [Visual Studio Code](https://code.visualstudio.com/) is an excellent an popular choice.  

```sh
brew cask install visual-studio-code
```

Another nice-to-have feature would be to the ability to launch the IDE from the command line.  
You can do this by updating the `~/.zshrc` file with the following function definition.  
This will allow launch the IDE with the following terminal command: `code`.  

Update that file by running the following command:  

```sh
cat <<'SHELL' >> ${HOME}/.zshrc
function code {
  /Applications/Visual\ Studio\ Code.app/Contents/MacOS/Electron "$@"
}
SHELL
```

**NOTE**: You may also need to source the `~/.zshrc` file for the system changes to take effect.  

### Install Postman

[Postman](https://www.postman.com/) is a REST API client.  

```sh
brew cask install postman
```

**NOTE**: You may be asked to enter your password to install.  

### Install MongoDB Compass

[MongoDB Compass](https://www.mongodb.com/products/compass) is a GUI for interacting with MongoDB and visualizing it's contents.  

If you're familiar with tools like: phpPgAdmin (for Postgres), phpMyAdmin (for MySQL/MariaDB), phpLiteAdmin (for SQLite), etc.; it's the same idea.  

```sh
brew cask install mongodb-compass
```

**NOTE**: You may be asked to enter your password to install.  

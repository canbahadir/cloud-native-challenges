# cloud-native-challenges
Challenges I worked on from various sources.


## BCFM

### TASK 0
Setup a vagrant environment to work with. Then use it with VS Code.

STEPS:

On your preferred OS download Vagrant & Virtual Box.
For this task I chosed to work with centos-8.
Setup a Vagrant Box ( https://app.vagrantup.com/bento/boxes/centos-8 )
Create a folder for vagrant boxes on your machine.
Open terminal

    cd <vagrant-box-folder>
    vagrant init bento/centos-8
    vagrant up


On VS Code install remote - SSH extension ( https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh ) 

Get ssh config from Vagrant
    vagrant ssh-config

Add output to VS Code Remote-SSH config file.
Connect to vagrant box using Remote-SSH.

Open terminal in VS Code and install needed packages:
Upgrade default packages.
    sudo dnf update
    sudo dnf upgrade

Add google cloud sdk repo
    sudo tee -a /etc/yum.repos.d/google-cloud-sdk.repo << EOM
    [google-cloud-sdk]
    name=Google Cloud SDK
    baseurl=https://packages.cloud.google.com/yum/repos/cloud-sdk-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOM

Install google cloud sdk
    sudo dnf install google-cloud-sdk

Initiate gcloud and give access to google account.
    gcloud init

Install git
    sudo dnf install git

Setup brew package manager
    sudo /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/vagrant/.bash_profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
    sudo yum groupinstall 'Development Tools'
    brew install gcc

Install kubectl, terraform, and helm.
    sudo dnf install kubectl
    brew install terraform
    brew install helm

### TASK 1

### TASK 2

### TASK 3
Create an app which serves "/bcfm" page. 
When a request sent it should have your name.

STEPS:

    cd task3/dockerapp
    docker build -t node-app:0.1 .
    docker images
    sudo docker run -p 4000:80 --name my-app node-app:0.1
    curl http://localhost:4000/bcfm

### TASK 4

### TASK 5

### TASK 6
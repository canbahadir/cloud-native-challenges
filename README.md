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
Create a kubernetes cluster. (3 Master, 1 Worker node)

First create a VPC network on gcloud.

    gcloud compute networks create kubernetes-the-kubespray-way --subnet-mode custom

Create a subnet

    gcloud compute networks subnets create kubernetes \
    --network kubernetes-the-kubespray-way \
    --range 10.240.0.0/24

Add internal firewall rules

    gcloud compute firewall-rules create kubernetes-the-kubespray-way-allow-internal \
    --allow tcp,udp,icmp,ipip \
    --network kubernetes-the-kubespray-way \
    --source-ranges 10.240.0.0/24

Add external firewall rules

    gcloud compute firewall-rules create kubernetes-the-kubespray-way-allow-external \
    --allow tcp:80,tcp:6443,tcp:443,tcp:22,icmp \
    --network kubernetes-the-kubespray-way \
    --source-ranges 0.0.0.0/0


Create 4 VMs which will be our kubernetes nodes.

3 VMs for master nodes.

    for i in 0 1 2; do
    gcloud compute instances create master-${i} \
        --async \
        --boot-disk-size 200GB \
        --can-ip-forward \
        --image-family ubuntu-1804-lts \
        --image-project ubuntu-os-cloud \
        --machine-type e2-standard-2 \
        --private-network-ip 10.240.0.1${i} \
        --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
        --subnet kubernetes \
        --tags kubernetes-the-kubespray-way,controller
    done

1 VM for worker node.

    gcloud compute instances create worker-0 \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --private-network-ip 10.240.0.20 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-kubespray-way,worker

Also we need to add a ssh key to communicate with VMs.

    ssh-keygen -t rsa -b 4096 -C username -f /home/vagrant/username

Add public key to a file that gcloud can parse.
Check task1 folder for example.
Upload this file to gcloud. This key can be used to do ssh access to all resources on gcloud.

    gcloud compute project-info add-metadata --metadata-from-file ssh-keys=/home/vagrant/sshpubkeylist

Set-up Kubespray

Create a virtual environment.

    python3 -m venv venv
    source venv/bin/activate

Clone kubespray and switch to stable branch.

    git clone https://github.com/kubernetes-sigs/kubespray.git
    cd kubespray
    git checkout release-2.13

Install dependencies.

    pip install -r requirements.txt

Create cluster config.

    cp -rfp inventory/sample inventory/mycluster

Update ansible inventory file.

    declare -a IPS=($(gcloud compute instances list --filter="tags.items=kubernetes-the-kubespray-way" --format="value(EXTERNAL_IP)"  | tr '\n' ' '))
    CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

Update inventory/mycluster/hosts.yaml as shown on task1/hosts.yaml
In inventory/mycluster/group_vars/k8s_cluster/k8s_cluster.yml change following part:

    supplementary_addresses_in_ssl_keys: [master0_external_ip, master1_external_ip, master2_external_ip]

In inventory/mycluster/group_vars/k8s_cluster/addons.yml set metrics_server_enabled to 'true'


Deploy the configuration. (This will take around 20 min)

    ansible-playbook -i inventory/mycluster/hosts.yaml -u username -b -v --private-key=/home/vagrant/username cluster.yml

Get kubeconfig file

    scp -i /home/vagrant/username $USERNAME@$IP_MASTER_0:/etc/kubernetes/admin.conf kubespray-do.conf

Modify kubespray-do.conf by changing master0_internal_ip with master0_external_ip.

Load the configuration for kubectl:

    export KUBECONFIG=$PWD/kubespray-do.conf

Now kubectl should be able to access nodes.

    [vagrant@localhost kubespray-test]$ kubectl get nodes
    NAME    STATUS   ROLES    AGE   VERSION
    node1   Ready    master   9d    v1.17.12
    node2   Ready    master   9d    v1.17.12
    node3   Ready    master   9d    v1.17.12
    node4   Ready    <none>   9d    v1.17.12

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


#### For kubernetes(GKE):


Create a api key to upload docker image to gcloud so helm can deploy from there

    gcloud iam service-accounts create dockerloader
    gcloud projects add-iam-policy-binding casestudy-307604 --member "serviceAccount:dockerloader@casestudy-307604.iam.gserviceaccount.com" --role "roles/storage.admin"

    gcloud iam service-accounts keys create keyfile.json --iam-account dockerloader@casestudy-307604.iam.gserviceaccount.com
    gcloud auth activate-service-account dockerloader@casestudy-307604.iam.gserviceaccount.com --key-file=keyfile.json

verify access

    gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin https://HOSTNAME
    docker login -u oauth2accesstoken -p "$(gcloud auth print-access-token)" https://HOSTNAME


Retag docker image

    sudo docker tag node-app:0.1 gcr.io/casestudy-307604/node-app


Push docker image

    sudo docker push gcr.io/casestudy-307604/node-app

Create a helm chart

    helm create helm-chart

Update helm-chart/values.yaml and helm-chart/templates/services.yaml according to task4/helmchart.

Deploy helm chart

    helm install nodeapp helm-chart/

Reserve a regional ip

    gcloud compute addresses create endpoints-ip --region us-central1

Get ip

[vagrant@localhost ~]$ gcloud compute addresses list
NAME          ADDRESS/RANGE   TYPE      PURPOSE  NETWORK  REGION       SUBNET  STATUS
endpoints-ip  xxx.xxx.xxx.xxx  EXTERNAL                    us-central1          IN_USE

Deploy ingress resource:

    helm install nginx-ingress ingress-nginx/ingress-nginx --set rbac.create=true --set controller.service.loadBalancerIP="xxx.xxx.xxx.xxx"

Apply ingress resource to sync with app service:

    kubectl apply -f /home/vagrant/cloud-native-challenges/task4/helmchart-basics/ingress.yaml

(Optional) To forward it to domain add new record on your domain panel

    Type: A Record
    Host: <preferred subdomain> // i defined "case"
    Value: <regional ip that we got above>
    TTL: Automatic

Now my app is reachable at

case.bahadircan.com

case.bahadircan.com/bcfm

### TASK 4
Create a CI/CD environment for app created in task3.
    - Choose whatever tool you want
    - CI/CD should start when there is a commit
    - There should a SonarQube Check with QualityGate Check.
    - CI/CD should deploy app to kubernetes with helm chart
    - app should scale up to 10 pods when there is more %40 percent cpu usage





### TASK 5

### TASK 6
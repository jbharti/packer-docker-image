# Create docker image using packer

### **Introduction**

Packer builder builds Docker images using Docker. The builder do the following things

-   Create docker container
-   Runs provisioners within this container
-   Create Image with New Tag
-   Push Image in to repository

### **Pre-Requisite**

The Docker builder must run on a machine that has

-   Install docker engine \[ https://docs.docker.com/engine/install/ \]
-   Install ansible for provision\[ https://docs.ansible.com/ansible/latest/installation\_guide/intro\_installation.html \]
-   Install packer \[ https://learn.hashicorp.com/tutorials/packer/get-started-install-cli \]

### **Installation**

##### **Checkout Repository**

-   Check out the project code from git.

```plaintext
$git clone http://gitserver.digite.in/devops-MS/packer.git
```

##### Create the configuration file of Packer

1.  Let’s create the configuration file of Packer, **ubuntu.json**, with this content:

```plaintext
{
    "builders": [
        {
            "name": "docker",
            "type": "docker",
            "image": "ubuntu:20.04",
            "commit": true
        }
    ],
    "provisioners": [
        {
            "type": "shell",
            "script": "ansible/install_python.sh"
        },
        {
            "type": "ansible",
            "playbook_file": "ansible/playbook.yaml"
        }
    ],
    "post-processors": [
        {
            "type": "docker-tag",
            "repository": "my-image",
            "tag": "1.0"
        },      
        {
            "type": "docker-push",
            // to push into AWS ECR
            //"ecr_login": true,
            //"aws_access_key": "YOUR KEY HERE",
            //"aws_secret_key": "YOUR SECRET KEY HERE",
            //"login_server": "https://12345.dkr.ecr.us-east-1.amazonaws.com/"
            
            // to push into docker hub
            "login": true,
            "login_username": "YOUR USERNAME",
            "login_password": "YOUR PASSWORD",
            "login_server": "https://hub.docker.com/"

      }
    ]
}
```

#####   
2\. Create provisioners

-   Create the script **install\_python**.**sh** to install Python :

```plaintext
#!/bin/bash
apt-get update
apt install -y python
```

-   Create the Ansible playbook **playbook.yaml**:

```plaintext
--- 
- hosts: all
  tasks:
    - name: message
      debug: msg="Container {{ inventory_hostname }}"
    - name: install
      package: name=git state=latest
```

##### **How to Uses**

Launch Packer to generate the image by running below command from terminal

```plaintext
packer build ubuntu.json
```

##### Verify Image

```plaintext
on old image-
docker run -it ubuntu:20.04 sh -c "git"
docker run -it ubuntu:20.04 sh -c "python --version"

on new image-
docker run -it my-image:1.0 -c "git --version"
docker run -it my-image:1.0 -c "python --version"
```
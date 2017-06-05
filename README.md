
# Requirements

You will need a driver machine with ansible installed and a clone of the current repository:

* If you are running on cloud (public/private network)
  * Install ansible on the edge node (with public ip)
* if you are running on private cloud (public network access to all nodes)
  * Install ansible on your laptop and drive the deployment from it

### Installing Ansible on RHEL

```
curl -O https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo rpm -i epel-release-latest-7.noarch.rpm
sudo yum update -y
sudo yum install -y  ansible
```

### Installing Ansible on Mac

* Install Annaconda
* Use pip install ansible

### Supported/Tested Platform

* RHEL 7.x
* Ansible 2.3.1.0
  * Ansible 2.3.0.0 seems to have a bug with conditional when which is used in some roles

# Available Components

* **Common**  Deploys Java
* **HDFS** Deploys HDFS filesystem
* **Spark** Deploys Spark in Standalone mode
* **Anaconda** Deploys Anaconda Python distribution
* **IOP** Deploys IBM Open Platform (IOP) 4.2.5
* **Notebook** Deploys Notebook Platform

# Deploying Components

### Create host inventory file

```
[all:vars]
ansible_connection=ssh
#ansible_user=root
#ansible_ssh_private_key_file=~/.ssh/ibm_rsa
gather_facts=True
gathering=smart
host_key_checking=False


[master]
FQDN   ansible_host=IP

[nodes]
FQDN   ansible_host=IP
FQDN   ansible_host=IP
FQDN   ansible_host=IP
FQDN   ansible_host=IP

```

### Create your playbook

```
- name: Deploy and configure IOP
  hosts: all
  remote_user: root
  roles:
    - role: common
    - role: iop

- name: Deploy and configure Anaconda
  hosts: all
  remote_user: root
  roles:
    - role: anaconda

- name: Deploy and configure Notebook Platform
  hosts: master
  remote_user: root
  roles:
    - role: notebook

```

### Deploy

```
ansible-playbook <deployment playbook.yml> -i <hosts inventory>
```

Example:

```
ansible-playbook setup-spark-cluster.yml -i hosts-fyre
```

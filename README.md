This repository deploys a few different modes of a Spark cluster and Data Science Platform based on Anaconda and Jupyter Notebook stack

Changelog from [https://github.com/lresende/ansible-spark-cluster](https://github.com/lresende/ansible-spark-cluster):
* Uses OpenJDK
* A single `/inventory/hosts/group_vars/all.yml` file that does not depend on previous role variables to function
* Proxy config for http and https proxies (i.e. for those using certificates). Place cert files (\*.crt) in `roles/proxy_config/files/certs/`
* Hadoop compiled separately from Spark, providing greater flexibility
* Preconfigured Jupyter Notebook file with auto generated config to launch sparkContext object appropriately
* H2O Flow configuration (not compatible with loading in files via S3a, so is selectable)
* Auto start spark cluster, jupyter lab instance, and optionally, h2o flow when booting headnode, graceful shutdown. Useful for scheduled boot and shutdown when using cloud (i.e. AWS, Google Compute Cloud, Azure)
 
# Requirements

You will need a driver machine with ansible installed and a clone of the current repository:

* If you are running on cloud (public/private network)
  * Install ansible on the head node

Additionally:
 - The designated headnodes will need a ssh key titled 'cluster_key'
 - All nodes will need public ssh key 'cluster_key.pub'
 - All nodes need user ec2-user with sudo permissions
 - Host inventory is manually specified and located at inventory/hosts
 - Variables for installation are located at `inventory/hosts/group_vars/all.yml`
   - Sparkling water can be turned on or off via `use_sparkling_water` variable 

**Software versions and URLs are specified in the variables file**

### Installing Ansible on RHEL

```
curl -O https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo rpm -i epel-release-latest-7.noarch.rpm
sudo yum update -y
sudo yum install -y  ansible
```


### Updating Ansible configuration

In order to have variable overriding from host inventory, please add the following configuration into your ~/.ansible.cfg file

```
[defaults]
host_key_checking = False
hash_behaviour = merge
```

### Supported/Tested Platform

* RHEL 7.x
* Ansible 2.6.3


# Defining your cluster deployment metadata (host inventory)

Ansible uses 'host inventory' files to define the cluster configuration, nodes, and groups of nodes
that serves a given purpose (e.g. master node).

Below is a host inventory sample definition:

```
[all:vars]
timeout=3600
[master]
Testh0   ansible_host=<IP>   ansible_host_private=<IP>  ansible_host_id=1

[nodes]
Testc0   ansible_host=<IP>   ansible_host_private=<IP>  ansible_host_id=3
Testc1   ansible_host=<IP>   ansible_host_private=<IP>  ansible_host_id=4
Testc2   ansible_host=<IP>   ansible_host_private=<IP>  ansible_host_id=5

```

Some specific configurations are:

* `install_java=True` : install/update java 8
* `install_temp_dir=/tmp/ansible-install` : temporary folder used for install files
* `install_dir=/opt` : where packages are installed (e.g. Spark)

**Note:** ansible_host_id is only used when deploying a "Spark Standalone" cluster.
**Note:** Ambari is currently only supporting Python 2.x

### Related ansible roles

* **Common**  Deploys Java and common dependencies

### Deploying

```
ansible-playbook --verbose <deployment playbook.yml> -i <hosts inventory>
```

Example:

```
ansible-playbook -i inventory/hosts -c paramiko setup-ds-platform.yml
```

# Deploying Spark standalone

In this scenario, a Standalone Spark cluster will be deployed with a few optional components.

### Related ansible roles
* **proxy_config** Configures for proxy network
* **cluster_network** Configures networking for cluster
* **Common**  Deploys Java and common dependencies
* **hadoop** Deploys hadoop in Standalone mode using slave nodes as data nodes
* **Spark** Deploys Spark in Standalone mode using slave nodes as workers
* **Spark-CLuster-Admin** Utility scripts for managing Spark cluster
* **ElasticSearch** Deploy ElasticSearch nodes on all slave nodes
* **Zookeeper** Depoys Zookeeper on all nodes (required by Kafka)
* **Kafka** Deploy Kafka nodes on all slave nodes
* **Anaconda** Deploys Anaconda Python
* **Sparkling Water** Deploys h20 and Sparkling Water for Spark
* **Spark Start Stop** Configures cluster to start Hadoop, Spark, Anaconda, and optionally Sparkling Water


# Legal Disclaimers

* The **Ambari** role will install [MySQL community edition](https://www.mysql.com/products/community/)
which is available under GPL license.

* The **Notebook** role will install [R](https://www.r-project.org/) which is available under [GPL2 | GPL 3](https://www.r-project.org/Licenses/)

By deploying these packages via the ansible utility scripts in this project you are accepting the
license terms for these components.
 

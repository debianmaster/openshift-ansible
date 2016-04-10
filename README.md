## Pre-req
> Setup ssh keys between work machine and servers before you start this.  
> I have One Centos machine with  hostname *cloud.i63.io* and ip address *104.199.140.227*  
> make sure following wild card mapping is setup  *.cloud.i63.io  should go to 104.199.140.227 


## Setup
> On your work machine     

```sh
git clone https://github.com/debianmaster/openshift-ansible.git
cd openshift-ansible
```

> Create following file at  /etc/ansible/hosts      

```yml
[OSEv3:children]
masters
nodes
etcd
lb
registry
router

# Set variables common for all OSEv3 hosts
[OSEv3:vars]

# SSH user, this user should allow ssh based auth without requiring a password
ansible_ssh_user=cjonagam
osm_default_subdomain=cloud.i63.io
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/htpasswd'}]



# If ansible_ssh_user is not root, ansible_sudo must be set to true
ansible_sudo=true

deployment_type=origin

# host group for masters
[masters]
cloud.i63.io openshift_public_hostname=cloud.i63.io  openshift_ip=104.199.140.227  openshift_public_ip=104.199.140.227 openshift_hostname=cloud.i63.io


# host group for nodes
[nodes]
cloud.i63.io openshift_public_hostname=cloud.i63.io  openshift_ip=104.199.140.227  openshift_public_ip=104.199.140.227 openshift_hostname=cloud.i63.io

# host group for etcd
[etcd]
cloud.i63.io

[lb]
cloud.i63.io


[router]
cloud.i63.io
```

```sh
ansible-playbook playbooks/byo/config.yml -i /etc/ansible/hosts
```

> SSH to master and execute following    
`htpasswd -c /etc/origin/htpasswd cjonagam`   #set password  
`oadm registry`   # creates a registry    
`oc edit nodes/cloud.i63.io`  # make the value of   




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
`htpasswd -c /etc/origin/htpasswd admin`   #set password    
`oc edit nodes/cloud.i63.io`  # make the value of   unschedulable  to false  
`oc policy add-role-to-user admin admin -n default`  # give permission on default project to admin   
`oc policy add-role-to-user admin admin -n openshift`  # give permission on openshift project to admin   
`oadm registry`   # creates a registry    
`oadm router`   # creates a openshift-haproxy router     

## Usage
```sh
oc login https://cloud.i63.io:8443 
oc status
```

> Add new secret to openshift for gitlab / github             


```sh
oc secrets new-basicauth gitlab --username=gitusername --password=gitpassword
oc secrets add serviceaccount/builder secrets/gitlab
```
> Create an image stream  *is.json*
```json
{
    "kind": "ImageStream",
    "apiVersion": "v1",
    "metadata": {
        "name": "shop-api",
        "namespace": "development",
        "labels": {
            "app": "shop-api"
        }
    }
}
```
`create -f is.json`    



> Create a build config file as follows with gitlab sercret   *bc.yml*    

```yml
apiVersion: "v1"
kind: "BuildConfig"
metadata:
  name: "sample-build"
spec:
  source:
    git:
      uri: "https://gitlab.com/dynamostack/shop-api.git" 
    sourceSecret:
      name: "gitlab"
    type: "Git"
  strategy:
    type: "Source"
    sourceStrategy:
     from:
      kind: DockerImage
      name: 'openshift/nodejs-010-centos7' 
```

```sh
oc create -f bc.yml   
```



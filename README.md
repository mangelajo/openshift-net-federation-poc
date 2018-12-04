# openshift connect POC

  The ansible playbooks in this repository are designed to spawn two separate
(all in one) openshift kubernetes clusters, and a third "federation" host, from
where you can command them.

  The main script of the playbooks is setup-fed-cluster.yml, this one will
call the other scripts in order.

  The playbooks use openstack to deploy the clusters, the following resources
will be created from create-net.yml:
   - 3 private networks (private1, private2, private3)
   - 3 subnets
   - 3 routers (router1, router2, router3)

  Then create-os-instances.yml will create 3 instances:
   - openshift1  (first openshift "cluster")
   - openshift2  (second openshift "cluster")
   - fedhost (a host for federation)

  create-os-instances.yml uses os_server to create the instances and
add_host to dynamically add those instances to the ansible inventory,
you can modify the static "inventory" file to add those hosts later,
for example if you want to only run parts of ansible with "--start-at-task",
because in this case, the "add_host" part will be skipped, and the
static inventory from "inventory" file will be used.

## pre-setup

  Copy cloud.yml.example to cloud.yml and edit your credentials to your cloud.

## setup

```bash
$ ansible-playbook setup-fed-cluster.yml
```

## the fedhost

  In your fedhost, the config for both clusters is configured on the root user

```bash
$ sudo su -
# grep 8443/system:admin ~/.kube/config* | grep xip
/root/.kube/config1:  name: default/38-145-34-212-xip-io:8443/system:admin
/root/.kube/config1:  name: default/38-145-35-46-xip-io:8443/system:admin
```

  You can sudo to your fedhost and then use the context of one of your clusters to login

```bash
$ sudo su -
# oc config use-context default/38-145-34-212-xip-io:8443/system:admin
# oc login -u system:admin

or

# oc config use-context default/38-145-34-212-xip-io:8443/system:admin
# oc login -u developer
Username: developer
Password: << ANY password
Login successful.

You have one project on this server: "myproject"

Using project "myproject".

#
```



## TO-DO

### Add support for load balancing

Edit /openshift3/kube-apiserver/ to add:

See: https://docs.openshift.com/enterprise/3.1/install_config/configuring_openstack.html#openstack-configuring-masters

then /etc/openstack.conf

[root@openshift1 kube-apiserver]# cat /etc/openstack.conf
[Global]
auth-url = <OS_AUTH_URL>
username = <OS_USERNAME>
password = <password>
tenant-id = <OS_TENANT_ID>
region = <OS_REGION_NAME>

[LoadBalancer]
use-octavia=true
subnet-id=

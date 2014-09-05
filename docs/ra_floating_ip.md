# Floating IP Resource Agent

## Description

`ocf:heartbeat:FloatingIP` is a resource agent for managing floating IP that is associated to a set of instances in OpenStack. The main purpose of this resource agent is to provide a virtual floating IP between instances. So a floating IP is associated to another instance if current associated insntace is fail.

The agent uses nova command to manage floating IP and use meta-data functions inside instance to retrieve information like ``uuid`` or current floating IP of instance.

Due to limitation of OpenStack, there some requirements when using this agent:

* **The instances should not have floating IPs associated before configuration**: since OpenStack do not allow multiple floating IPs being associated with instance, user must make sure there is no floating IP associated before configuring the agent. User can connect to an instance outside the cluster and login to instances inside cluster. (Be sure to add public key in `~/.ssh/authorized_keys`)
* **There should be only one FloatingIP resource in the cluster**
* **The floating IP should be only used for instances inside a cluster**: user inside the same project can associate/disassociate the floating IP. Currently there is no method to restrict ACL of floating IP association, thus a floating IP should be reserved by user in mind.
 

## Software requirements

* python-novaclient: 2.15.0 is tested
* curl
* python


## Parameters

* `ip`: floating IP address

  The floating IP to be used. For example: "172.16.19.99". The IP should be allocated to project before using the agent and user must make sure no one in the same project will use it.

* `os_tenant_id`: Tenant/Project ID
* `os_tenant_name`: Tenant/Project name
* `os_auth_url`: OpenStack Auth URL
* `os_username`: OpenStack Username
* `os_password`: Password of user
* `os_cacert`: Certificate file to use

## Operations

Operations' timeout value **should** be set to advisory minimun:

```
   start         timeout=60s
   stop          timeout=60s
   status        interval=60s timeout=60s
   monitor       interval=60s timeout=60s
```

## Example

The following snippet creates a FloatingIP resource whose floating IP is `172.16.19.135`

```
primitive p_floating_ip ocf:heartbeat:FloatingIP \
    params ip="172.16.19.135" \
    params os_tenant_id="mytenantid" \
    params os_tenant_name="mytenantname" \
    params os_auth_url="http://172.16.16.16:5000/v2.0" \
    params os_username="myusername" \
    params os_password="mypassword" \
    params os_cacert="/path/to/cert_file" \
    op start interval="0" timeout="60s" \
    op stop interval="0" timeout="60s" \
    meta target-role="Started"
```

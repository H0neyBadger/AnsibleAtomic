# AnsibleAtomic

A collection of ansible roles to:
* Creates VMs using virt-install (kvm)
* Extends VMs disk space
* Create a private docker registry
* Installs kubernetes nodes/master
* Sets etcd & flannel network

This playbook is based on steps defined in :
* http://www.projectatomic.io/docs/quickstart/
* http://www.projectatomic.io/docs/gettingstarted/

WARNING: This job is ***insecure***. So do not use if for production purpose !

# Requirements

* virt-install
* a valid DNS resolution
* a valid SSH connection (from your ansible host)

# Create an atomic cluster

Edit the default [inventory](inventory) and verify the path of your atomic image ***virt_setup_template***
```
[hypervisors]
localhost ansible_connection=local ansible_python_interpreter=/usr/bin/python3

[instances]
atomic001 ansible_host=atomic001.home.lab
atomic002 ansible_host=atomic002.home.lab
atomic003 ansible_host=atomic003.home.lab
atomic004 ansible_host=atomic004.home.lab

[docker_registries]
atomic001

[kubernetes_master]
atomic001

[instances:vars]
ansible_user=fedora
virt_setup_ram=768
virt_setup_vcpu=2
virt_setup_template="/var/lib/libvirt/images/Fedora-AtomicHost-28-20180425.0.x86_64.qcow2"
```

Also, edit the [group_vars/all/main.yml](group_vars/all/main.yml) to edit the ***virt_setup_ssh_authorized_keys*** with your ```ssh public key``` 


Run the ***create*** playbook
```bash
ansible-playbook ./create.yml -i inventory -v
# to delete your cluster :
# ansible-playbook ./destroy.yml -i inventory -v
```

## Networking

You may use the embedded libvirt dnsmasq/dhcp service to resolve your DNS names.
To achieve that, you have to edit the libvirt network config and add the localOnly='yes' parameter.
```xml
<domain name='home.lab' localOnly='yes'/>
``` 
Where 'name' is your domain name suffix 


```bash
virsh net-edit default 
virsh net-destroy default
virsh net-start default
virsh net-dumpxml default
```

You should have a similar result :
```xml
<network>
  <name>default</name>
  <uuid>7fadd52e-b8c0-4f11-a254-940a27d950be</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:0b:d3:3d'/>
  <domain name='home.lab' localOnly='yes'/>
  <ip address='192.168.100.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.100.128' end='192.168.100.254'/>
    </dhcp>
  </ip>
</network>
```

then use the libvirt's dnsmasq server as local DNS for your domain 
```
server=/home.lab/192.168.100.1
```

# Test app 
from your atomic master
```
kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080
```


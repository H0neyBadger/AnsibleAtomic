
# Requirements

* virt-install
* a valid DNS resolution

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

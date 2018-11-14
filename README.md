Maas-Control
=========

Do stuf with Maas

Requirements
------------

* Working Maas server and shell access to this server

Limitations
-----------

* Machines are not yet supported
* Creation of seperate dns records (A, CNAME etc) is not yet supported
* Breakdown of network and devices are not yet supported

Role Variables
--------------
```yaml
# maas user
maas_user: maas
# maas api url
maas_url: http://172.16.44.100:5240/MAAS
# an array of public keys set to the maas user
maas_public_keys: []
# default maas fabric to use
default_fabric: 'fabric-2'
# default mtu setting
default_mtu: 1500
# default domain devices and machines are created at
default_domain: 'maas'
# number of ip addresses after the start of a subnet
default_maas_dhcp_start_offset: 25601
# number of ip addresses in the dhcp range
default_maas_dhcp_number_of_addresses: 765
# default dns servers set on a subnet
default_dns: "8.8.8.8, 8.8.4.4"
# default upstream dns servers when maas is used as dns server
maas_upstream_dns: "8.8.8.8, 8.8.4.4"
# the id of the primary rack controller (used for dhcp)
default_maas_primary_rack_controller_id: ""
# id of the secondary rack controller (used for dhcp, leave "" if not used)
default_maas_secondary_rack_controller_id: ""
# default inventory group to query for devices or machines
maas_managed_inventory_group: all
# default type if network_interfaces are defined, can be device, machine or ignore
default_maas_type: device
# frabrics to create (api does not yet support attachting facbrics to interfaces)
fabrics:
  - name: 'fabric-2'
    interface: enp179s0f0
  - name: 'fabric-3'
    interface: enp179s0f1

vlans: {}
# example vlan / dhcp configs
# vlans:
#   220:
#     subnet: 10.220.0.0/16 (required)
#     mtu: 1500 (defaults to default_mtu)
#     dhcp_start_offset: 10 (defaults to default_maas_dhcp_start_offset)
#     dhcp_number_of_addresses: 220 (defaults to default_maas_dhcp_number_of_addresses)
#     dhcp: false (defaults to true)

# to create a maas device (hostname, dns and posible ip to mac reservation)
# set in the host_var of a device
# maas_type: device
# network_interfaces:
#   - mac_address: <the mac address>
#     ip_address: <the ip address>
#   - mac_address: <the mac address>
#     ip_address: <the ip address>
```

Dependencies
------------

* None

Example Playbook
----------------

```
- hosts: maas-master
  gather_facts: false
  roles:
    - role: naturalis.maas-control
```

License
-------

Apache 2.0

Author Information
------------------

Atze de Vries - atze.devries@naturalis.nl

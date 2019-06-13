# maas-control

Ansible playbook to control and configure [MAAS](https://maas.io/) (Metal as a
Service).

## Requirements

* Working MAAS server and SSH access to this server. At Naturalis we use the
  [ansible-maas-deploy](https://github.com/naturalis/ansible-maas-deploy)
  role.
* The rack controller ID for both the primary and secondary rack controller
  (which is the last part of the URL of both machines on the Controller page)
  should be added as variables (see below).
* All VLAN's should be defined on the required interfaces of your
  rack controller(s). You cannot do this from the MAAS API.
* For backup with restic to S3 a bucket with access and secret key should be
  available.

## Limitations

* Creation of separate DNS records (A, CNAME etc) is not yet supported.

## Role Variables

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
# fabrics to create (api does not yet support attachting facbrics to interfaces)
fabrics:
  - name: 'fabric-2'
    interface: enp179s0f0
  - name: 'fabric-3'
    interface: enp179s0f1
# default image
default_machine_distro: 'bionic'

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

# Public SSH keys MAAS will be able to add to machines
maas_public_keys:
  - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDXGplrKtQ30nhhpkKECjb5WGLhPDwGEMI+xqGYYQZdc+/XO77gpF8s9FI8F40dC+n2dIlVQqVQ6AmDSec7ZeWljN9QrWFlf/tcEcItQ20WHNYxuMpewO8KwhLpQpxsGiRBC+t6cXKUpGImiMIZTdjou1iH2m40EFUEhhMpyqZblhXBSU8QaABne5WANM5LNeLMqDKgrEuwmtUAow54b4VfLH92WG4rH35XhvSYH9Ty9xBG1ks3Jg3WkueLmxiWtRq4mzeBos7MXeN8x4WOqmzieqK7IMI9taTZG2atEGSf8DRaDKsSMt9eVV+r1RfRgpokrRgxVHX0KTsLonH1i3+h david.heijkamp@naturalis.nl"

# SSH key for maasbackup user
maas_backup_key:
  public: "ssh-rsa AAAAB3N..."
  private: |
   -----BEGIN RSA PRIVATE KEY-----
   MIIEpQIBAAKCAQEAxJ1KXa1T+ihsgbbTM93RJHV1IuPen54NnTdnIKhSv9+kv5Nj
   ...

# Secrets and URL for backup to S3 with restic
restic_aws_access_key: "AAAAAAAAAAAAAAAAAAAA"
restic_aws_secret_key: "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
restic_password: "zzzzzzzzzzzzzzzz"
restic_aws_repo: "s3:https://s3.amazonaws.com/restic-repo"

# Optional DHCP snippets, for example:
dhcp_snippets:
  global:
    - name: Cumulus provision URL
      snippet: |-
        option www-server code 72 = ip-address;
        option cumulus-provision-url code 239 = text;
  subnets:
    - name: ONIE installer
      snippet: |-
        option default-url = "http://10.225.0.10/onie-installer";
        option cumulus-provision-url "http://10.225.0.10/ztp_oob.sh";
        option www-server 10.225.0.10;
      subnet: "10.225.0.0/16"
  nodes: []

# Optional commisioning scripts
commissioning_scripts:
  - name: Hello world
    script: |
      #!/bin/bash
      echo "Hello world!"
```

## Dependencies

* [restic](https://github.com/naturalis/ansible-restic)

## Example Playbook

```
- hosts: maas-master
  gather_facts: false
  roles:
    - role: naturalis.maas-control
```

## Backup

This role (optionally) configures backups with restic. You can skip backup
configuration by using `--skip-tags backup`.

When backup is configured, a [prebackup
script](./files/backup-maas-cluster.yml) based on Ansible will be
installed. This prebackup script runs from the `maas-master` and runs tasks on
both the `maas-master` and `maas-slave` server. Ansible will be run with the
`maasbackup` user on both machines. Logs of the prebackup script are stored on
on `maas-master` in `/var/log/ansible/maasbackup.log`.

## License

Apache 2.0

## Author Information

Atze de Vries - atze.devries@naturalis.nl

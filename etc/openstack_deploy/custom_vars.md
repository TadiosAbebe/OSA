# STG Environment
> vlan tags
- management vlan: 10
- storage vlan: 20
- tunnel vlan: 30
- ceph vlan: 40

> /etc/openstack_deploy/openstack_user_config.yml
```yaml
_internal_lb_vip_address: &internal_lb_vip_address 172.29.236.8
_external_lb_vip_address: &external_lb_vip_address 172.29.236.9
_infra_hosts: &infra_hosts
  infra01:
    ip: 172.29.236.11
  infra02:
    ip: 172.29.236.12
  infra03:
    ip: 172.29.236.13
#_compute_hosts: &compute_hosts
#  compute01:
#    ip: 172.29.236.21
#  compute02:
#    ip: 172.29.236.22
#  compute03:
#    ip: 172.29.236.23
_storage_hosts: &storage_hosts
  storage01:
    ip: 172.29.236.31
  storage02:
    ip: 172.29.236.32
  storage03:
    ip: 172.29.236.33
_cidr_networks: &cidr_networks
  container: 172.29.236.0/22
  tunnel: 172.29.240.0/22
  storage: 172.29.244.0/22
_used_ips: &used_ips
  - "172.29.236.1,172.29.236.50"
  - "172.29.240.1,172.29.240.50"
  - "172.29.244.1,172.29.244.50"
  - "172.29.248.1,172.29.248.50"
```

> /etc/hosts
```bash
127.0.0.1 localhost
172.29.236.10 node00
172.29.236.11 node11
172.29.236.12 node12
172.29.236.13 node13
172.29.236.21 node21
172.29.236.22 node22
172.29.236.22 node23
172.29.236.31 node31
172.29.236.32 node32
172.29.236.33 node33
```

> /etc/openstack_deploy/user_variables.yml
```yml
ceph_mons:
  - 172.29.244.31
  - 172.29.244.32
  - 172.29.244.33
```

> /etc/neptlan
```yml
network:
  ethernets:
    eno1: {}
    eno2: {}
    eno3: {}
    eno4: {}
  bonds:
      bond0: # Management and storage
          interfaces:
          - eno1
          - eno3
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: eno1
      bond1: # Provider and tenant
          interfaces:
          - eno2
          - eno4
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: eno2
  vlans:
    bond0.10: # Management VLAN
      id: 10
      link: bond0
    bond0.20: # Openstack Storage and CEPH public network VLAN
      id: 20
      link: bond0
    bond1.30: # Tunnel (geneve/vxlan/gre) Network
      id: 30
      link: bond1
    bond1.40: # CEPH cluster network VLAN
      id: 40
      link: bond1
  bridges:
    br-mgmt: # Container/Host management bridge
      addresses:
      - 172.29.236.x/22
      interfaces:
      - bond0.10
      mtu: 9000
      routes:
      - to: 0.0.0.0/0
        via: 172.29.236.1      
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
    br-storage: # Storage bridge
      addresses:
      - 172.29.244.x/22
      interfaces:
      - bond0.20
      mtu: 9000
    br-overlay: # Networking (tunnel/overlay) bridge
      addresses:
      - 172.29.240.x/22
      interfaces:
      - bond1.30
      mtu: 9000
    br-ceph: # Ceph cluster bridge
      addresses:
      - 172.29.248.x/22
      interfaces:
      - bond1.40
      mtu: 9000
  version: 2
```

# TST Environment
> vlan tags
- management vlan: 110
- storage vlan: 120
- tunnel vlan: 130
- ceph vlan: 140

> /etc/openstack_deploy/openstack_user_config.yml
```yaml
_internal_lb_vip_address: &internal_lb_vip_address 172.28.236.8
_external_lb_vip_address: &external_lb_vip_address 172.28.236.9
_infra_hosts: &infra_hosts
  infra01:
    ip: 172.28.236.11
  infra02:
    ip: 172.28.236.12
  infra03:
    ip: 172.28.236.13
#_compute_hosts: &compute_hosts
#  compute01:
#    ip: 172.28.236.21
#  compute02:
#    ip: 172.28.236.22
#  compute03:
#    ip: 172.28.236.23
_storage_hosts: &storage_hosts
  storage01:
    ip: 172.28.236.31
  storage02:
    ip: 172.28.236.32
  storage03:
    ip: 172.28.236.33
_cidr_networks: *cidr_networks
  container: 172.28.236.0/22
  tunnel: 172.28.240.0/22
  storage: 172.28.244.0/22
_used_ips: &used_ips
  - "172.28.236.1,172.28.236.50"
  - "172.28.240.1,172.28.240.50"
  - "172.28.244.1,172.28.244.50"
  - "172.28.248.1,172.28.248.50"
```

> /etc/hosts
```bash
127.0.0.1 localhost
172.28.236.10 node00
172.28.236.11 node11
172.28.236.12 node12
172.28.236.13 node13
172.28.236.21 node21
172.28.236.22 node22
172.28.236.22 node23
172.28.236.31 node31
172.28.236.32 node32
172.28.236.33 node33
```

> /etc/openstack_deploy/user_variables.yml
```yml
ceph_mons:
  - 172.28.244.31
  - 172.28.244.32
  - 172.28.244.33
```
```yml
network:
  ethernets:
    eno1: {}
    eno2: {}
    eno3: {}
    eno4: {}
  bonds:
      bond0: # Management and storage
          interfaces:
          - eno1
          - eno3
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: eno1
      bond1: # Provider and tenant
          interfaces:
          - eno2
          - eno4
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: eno2
  vlans:
    bond0.110: # Management VLAN
      id: 110
      link: bond0
    bond0.120: # Openstack Storage and CEPH public network VLAN
      id: 120
      link: bond0
    bond1.130: # Tunnel (geneve/vxlan/gre) Network
      id: 130
      link: bond1
    bond1.140: # CEPH cluster network VLAN
      id: 140
      link: bond1
  bridges:
    br-mgmt: # Container/Host management bridge
      addresses:
      - 172.28.236.x/22
      interfaces:
      - bond0.10
      mtu: 9000
      routes:
      - to: 0.0.0.0/0
        via: 172.28.236.1      
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
    br-storage: # Storage bridge
      addresses:
      - 172.28.244.x/22
      interfaces:
      - bond0.20
      mtu: 9000
    br-overlay: # Networking (tunnel/overlay) bridge
      addresses:
      - 172.28.240.x/22
      interfaces:
      - bond1.30
      mtu: 9000
    br-ceph: # Ceph cluster bridge
      addresses:
      - 172.28.248.x/22
      interfaces:
      - bond1.40
      mtu: 9000
  version: 2
```

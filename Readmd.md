# Openstack Internal Documentation
- [Creatting a ceph cluster](#Creating-a-ceph-cluster)
  - [Preparing ceph for openstack deployment](##Preparing-ceph-for-openstack-deployment)
- [Deploying openstack with ceph](#Deploying-openstack-with-ceph)
  - [Preparing the deployment host](##Preparing-the-deployment-host)
  - [Preparing the target host](##Preparing-the-target-host)
  - [Running openstack-ansible deployment scripts](##Running-openstack-ansible-deployment-scripts)
  - [Verifying the openstack cloud](##Verifying-the-openstack-cloud)
- [Openstack post installation configurations](#Openstack-post-installation-configurations)
- [Testing the openstack using rally](#Testing-the-openstack-using-rally)
- [Appendix](#Appendix)
  - [Temporary netplan file](##Temporary-netplan-file)
  - [Installing monitoring packages](##Installing-monitoring-packages)
  - [Installing management packages](##Installing-management-packages)

Before preceding with deployment of openstack, we need to setup an external ceph cluster that we later on connect with our openstack deployment

# Creating a ceph cluster

- The below steps assume that you have already installed operating system on the hosts and it is tested on ubuntu server 22.04
- We need to have additional block devices/disks that we will use as osd later on
- If this disks has been used for ceph deployment before we need to clear the disks using the following commands
```bash
vgremove -f ceph-...
pvremove -ff /dev/sdx
```
- Start by configuring the network on the host with the following network configuration file. and don't forget to change
    - the interface names under ethernets i.e. interface0, interface1, interface2, interface3
    - the addresses section of each bridge i.e. 172.2x.236.x/22, 172.2x.236.x 172.2x.244.x/22, 172.2x.240.x/22
    - the vlan names and ids bond0.x, id: x,
```bash
nano /etc/netplan/...
```
```yaml
network:
  ethernets:
    interface0: {}
    interface1: {}
    interface2: {}
    interface3: {}
  bonds:
      bond0: # Management and storage
          interfaces:
          - interface0
          - interface2
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: interface0
      bond1: # Provider and tenant
          interfaces:
          - interface1
          - interface3
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: interface1
  vlans:
    bond0.x: # Management VLAN
      id: x
      link: bond0
    bond0.x: # Openstack Storage and CEPH public network VLAN
      id: x
      link: bond0
    bond1.x: # Tunnel (geneve/vxlan/gre) Network
      id: x
      link: bond1
    bond1.x: # CEPH cluster network VLAN
      id: x
      link: bond1
  bridges:
    br-mgmt: # Container/Host management bridge
      addresses:
      - 172.2x.236.x/22
      interfaces:
      - bond0.x
      mtu: 9000
      routes:
      - to: 0.0.0.0/0
        via: 172.2x.236.x      
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
    br-storage: # Storage bridge
      addresses:
      - 172.2x.244.x/22
      interfaces:
      - bond0.x
      mtu: 9000
    br-overlay: # Networking (tunnel/overlay) bridge
      addresses:
      - 172.2x.240.x/22
      interfaces:
      - bond1.x
      mtu: 9000
    br-ceph: # Ceph cluster bridge
      addresses:
      - 172.2x.248.x/22
      interfaces:
      - bond1.x
      mtu: 9000
  version: 2
```
- After this apply the network configuration you just configured
```bash
sudo netplan apply
```
- In the event network connection get disconnected after applying the above netplan, restarting the host will resolve the issue
- If we are configuring the deployment host, on a system that has cloud-init, we need to configure the cloud-init to preserve_hostname and to not update_hostname and update_etc_hosts to do so we need
  - change preserve_hostname to true
  - comment update_hostname and update_etc_hosts
```bash
nano /etc/cloud/cloud.cfg
```
```bash
systemctl restart cloud-init
```
- Update and upgrade your system
```bash
sudo apt update && sudo apt upgrade -y
```

> Optional: Monitoring packages could be installed and setup if you want to monitor this server with zabbix, refer to the appendix section on how to install and setup zabbix monitoring [Installing monitoring packages](##Installing-monitoring-packages)

> Optional: Management packages could be installed and configured if you want to manage this server with cockpit, refer to the appendix section on how to install and setup cockpit [Installing management packages](##Installing-management-packages)

- Set the ceph version for your deployment using the following command
```bash
CEPH_RELEASE=18.2.0
```
- Get the cephadm installer
```bash
curl --silent --remote-name --location https://download.ceph.com/rpm-${CEPH_RELEASE}/el9/noarch/cephadm
```
- Make the installer executable
```bash
chmod +x cephadm
```
- Add the correct version of the repo
```bash
./cephadm add-repo --release reef
```
- Start the installation
```bash
./cephadm install
```
> The below step will only be performed on the bootstrap node and not on the other nodes, skip this for all nodes except the bootstrap node
>> - Start the cluster bootstrap, don't forget to substitute the --mon-ip and --cluster-network parameters 172.2x.244.x, 172.2x.248.x22
```bash
cephadm bootstrap --mon-ip 172.2x.244.x --initial-dashboard-user "admin" --initial-dashboard-password "password" --dashboard-password-noupdate --cluster-network 172.2x.248.0/22
```
>> - Copy the ssh keys from /etc/ceph/ceph.pub on this bootstrap node to the rest of the cluster node of /root/.ssh/authorized_key
- Install the ceph-common package
```bash
apt install ceph-common
```
> The below steps will only be performed on the bootstrap node and not on the other nodes, skip this for all nodes except the bootstrap node
>> - Go into the cephadm shell with `cephadm shell` if you don't have the ceph-common package installed
>> - Add the rest of the ceph nodes
```bash
ceph orch host add nodexx 172.2x.244.x
```
>> - Add the admin label to the rest of the storage node for easier management
```bash
ceph orch host label add nodexx _admin
```
>> - Make some services to be deployed on all hosts
```bash
ceph orch apply prometheus '*'
ceph orch apply alertmanager '*'
ceph orch apply mgr '*'
```
>> - Add osd to the ceph cluster
```bash
ceph orch daemon add osd nodexx:/dev/sdx
```

## Preparing ceph for openstack deployment

- Create pools that are used by openstack service
```bash
ceph osd pool create volumes
ceph osd pool create images
ceph osd pool create backups
ceph osd pool create vms
```
- Initialize the pools
```bash
rbd pool init volumes
rbd pool init images
rbd pool init backups
rbd pool init vms
```
- Setup ceph authentication for clients
```bash
ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images' mgr 'profile rbd pool=images'
```
```bash
ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=volumes, profile rbd pool=vms'
```
```bash
ceph auth get-or-create client.cinder-backup mon 'profile rbd' osd 'profile rbd pool=backups' mgr 'profile rbd pool=backups'
```

# Deploying openstack with ceph

## Preparing the deployment host

- The below steps assume that you have already installed operating system on the hosts and it is tested on ubuntu server 22.04
- The deployment host is the host which we will be using as our ansible controller, and install all ansible and openstack ansible related packages
- We will start by configuring the network, for configuring temporary network configuration you can refer the following netplan [Temporary netplan file](##Temporary-netplan-file)
```yaml
network:
  ethernets:
    interface0: {}
  bonds:
      bond0: # Management and storage
          interfaces:
          - interface0
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: interface0
  vlans:
    bond0.x: # Management VLAN
      id: x
      link: bond0
    bond0.x: # Openstack Storage and CEPH public network VLAN
      id: x
      link: bond0
    bond0.x: # Tunnel (geneve/vxlan/gre) Network
      id: x
      link: bond0
    bond0.x: # CEPH cluster network VLAN
      id: x
      link: bond0
  bridges:
    br-mgmt: # Container/Host management bridge
      addresses:
      - 172.2x.236.x/22
      interfaces:
      - bond0.x
      mtu: 9000
      routes:
      - to: 0.0.0.0/0
        via: 172.2x.236.1
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
    br-storage: # Storage bridge
      addresses:
      - 172.2x.244.x/22
      interfaces:
      - bond0.x
      mtu: 9000
    br-overlay: # Networking (tunnel/overlay) bridge
      addresses:
      - 172.2x.240.x/22
      interfaces:
      - bond0.x
      mtu: 9000
    br-ceph: # Ceph cluster bridge
      addresses:
      - 172.2x.248.x/22
      interfaces:
      - bond0.x
      mtu: 9000
  version: 2
```
- We don't necessary need the bridge br-overly, br-ceph vlans bond.30, bond.40 to be configured on the deployment host but it doesn't heart to have them
- After this apply the network configuration you just configured
```bash
sudo netplan apply
```
- In the event network connection get disconnected after applying the above netplan, restarting the host will resolve the issue
- If we are configuring the deployment host, on a system that has cloud-init, we need to configure the cloud-init to preserve_hostname and to not update_hostname and update_etc_hosts to do so we need
  - change preserve_hostname to true
  - comment update_hostname and update_etc_hosts
```bash
nano /etc/cloud/cloud.cfg
```
```bash
systemctl restart cloud-init
```
- Update and upgrade your system
```bash
sudo apt update && sudo apt upgrade -y
```
- After update is completed, restart the host
```bash
sudo reboot now
```

> Optional: Monitoring packages could be installed and setup if you want to monitor this server with zabbix, refer to the appendix section on how to install and setup zabbix monitoring [Installing monitoring packages](##Installing-monitoring-packages)

> Optional: Management packages could be installed and configured if you want to manage this server with cockpit, refer to the appendix section on how to install and setup cockpit [Installing management packages](##Installing-management-packages)

- Update the hosts file with the openstack cluster hosts ip and name, you can find the list inside [etc/openstack_deploy/custom_vars.md]()
```bash
sudo nano /etc/hosts
```
```bash
127.0.0.1 localhost
172.2x.236.x xxx
```
- Configure timezone
```bash
timedatectl set-timezone Africa/Addis_Ababa
```
- Generate ssh-keys
```bash
ssh-keygen
```
- Install additional packages as per the openstack-ansible deployment
> Refer to the official documentation to get up to dated packages list to install for the openstack version you are deploying
```bash
sudo apt install build-essential git chrony openssh-server python3-dev sudo -y
```
- Clone and bootstrap openstack-ansible, refer the branch you want to clone by going to the official openstack-ansible documentation and navigating to the latest stable branch, so don't forget to replace -b xxx in the git clone command
```bash
tmux
```
```bash
git clone -b xxx https://github.com/openstack/openstack-ansible.git /opt/openstack-ansible
cd /opt/openstack-ansible
./scripts/bootstrap-ansible.sh
```
- Configure openstack-ansible, don't forget to replace the xxx on the git check out line to the suitable branch(staging, master)
```bash
cp -r /opt/openstack-ansible/etc/openstack_deploy /etc/openstack_deploy
cd ~
git clone https://github.com/TadiosAbebe/OSA.git
cd OSA
git checkout xxx
```
```bash
cp etc/openstack_deploy/user_variables.yml /etc/openstack_deploy
cp etc/openstack_deploy/openstack_user_config.yml /etc/openstack_deploy
```
- Edit the contents of /etc/openstack_deploy/openstack_user_config.yml and replace with variables that reside under etc/openstack_deploy/custom_vars.yml
- Edit the ceph_mons: variable inside /etc/openstack_deploy/user_variables to reflect your current environment
- Generate openstack service credentials
```bash
cd /opt/openstack-ansible
./scripts/pw-token-gen.py --file /etc/openstack_deploy/user_secrets.yml
```

## Preparing the target host

- The below steps assume that you have already installed operating system on the hosts and it is tested on ubuntu server 22.04
- We will start by configuring the network
```yaml
network:
  ethernets:
    interface0: {}
    interface1: {}
    interface2: {}
    interface3: {}
  bonds:
      bond0: # Management and storage
          interfaces:
          - interface0
          - interface2
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: interface0
      bond1: # Provider and tenant
          interfaces:
          - interface1
          - interface3
          mtu: 9000
          parameters:
            lacp-rate: fast
            mii-monitor-interval: 100
            transmit-hash-policy: layer3+4
            mode: active-backup
            primary: interface1
  vlans:
    bond0.x: # Management VLAN
      id: x
      link: bond0
    bond0.x: # Openstack Storage and CEPH public network VLAN
      id: x
      link: bond0
    bond1.x: # Tunnel (geneve/vxlan/gre) Network
      id: x
      link: bond1
    bond1.x: # CEPH cluster network VLAN
      id: x
      link: bond1
  bridges:
    br-mgmt: # Container/Host management bridge
      addresses:
      - 172.2x.236.x/22
      interfaces:
      - bond0.x
      mtu: 9000
      routes:
      - to: 0.0.0.0/0
        via: 172.2x.236.x      
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
    br-storage: # Storage bridge
      addresses:
      - 172.2x.244.x/22
      interfaces:
      - bond0.x
      mtu: 9000
    br-overlay: # Networking (tunnel/overlay) bridge
      addresses:
      - 172.2x.240.x/22
      interfaces:
      - bond1.x
      mtu: 9000
    br-ceph: # Ceph cluster bridge
      addresses:
      - 172.2x.248.x/22
      interfaces:
      - bond1.x
      mtu: 9000
  version: 2
```
- After this apply the network configuration you just configured
```bash
sudo netplan apply
```
- In the event network connection get disconnected after applying the above netplan, restarting the host will resolve the issue
- If we are configuring the deployment host, on a system that has cloud-init, we need to configure the cloud-init to preserve_hostname and to not update_hostname and update_etc_hosts to do so we need
  - change preserve_hostname to true
  - comment update_hostname and update_etc_hosts
```bash
nano /etc/cloud/cloud.cfg
```
```bash
systemctl restart cloud-init
```
- Update and upgrade your system
```bash
sudo apt update && sudo apt upgrade -y
```
> Optional: Monitoring packages could be installed and setup if you want to monitor this server with zabbix, refer to the appendix section on how to install and setup zabbix monitoring [Installing monitoring packages](##Installing-monitoring-packages)

> Optional: Management packages could be installed and configured if you want to manage this server with cockpit, refer to the appendix section on how to install and setup cockpit [Installing management packages](##Installing-management-packages)

> At this point it is a good idea to go to the deployment node's cockpit interface and add the rest of the node inside cockpit for easier management

- Update the hosts file with the openstack cluster hosts ip and name, you can find the list inside ...
```bash
sudo nano /etc/hosts
```
```bash
127.0.0.1 localhost
172.2x.236.x xxx
```
- Configure timezone
```bash
timedatectl set-timezone Africa/Addis_Ababa
```
- At this point make sure the ssh-keys of the deployment host is present under the /root/.ssh/authorized_host
```bash
nano /root/.ssh/authorized_keys
```
- Install additional packages as per the openstack-ansible deployment
> Refer to the official documentation to get up to dated packages list to install for the openstack version you are deploying
```bash
apt install bridge-utils debootstrap openssh-server tcpdump vlan python3 openvswitch-switch -y && apt install linux-modules-extra-$(uname -r) -y
```

## Running openstack-ansible deployment scripts

- Run the setup hosts script first
```bash
tmux
```
```bash
cd /opt/openstack-ansible/playbooks
openstack-ansible setup-hosts.yml
```
- Record the output of setup-hosts script with the below command
```bash
mv /openstack/log/ansible-logging/ansible.log  /openstack/log/ansible-logging/setup-hosts.log
```
- Run the setup infrastructure script
```bash
openstack-ansible setup-infrastructure.yml
```
- Record the output of setup-infrastructure script
```bash
mv /openstack/log/ansible-logging/ansible.log  /openstack/log/ansible-logging/setup-infrastructure.log
```
- Finally run the setup-openstack script
```bash
openstack-ansible setup-openstack.yml
```
- Record the output of setup-openstack script
```bash
mv /openstack/log/ansible-logging/ansible.log  /openstack/log/ansible-logging/setup-openstack.log
```

## Verifying the openstack cloud

- To get the horizon dashboard login password run the following command
```bash
cat /etc/openstack_deploy/user_secrets.yml | grep keystone_auth
```
- ssh to the utility container of the deployment from the deployment node
```bash
ssh infra01_utility...
```
- Source the openrc file
```bash
source openrc
```
- Run the below commands and review their output
```bash
openstack endpoint list
openstack compute service list
openstack network agent list
openstack volume service list
openstack image list
```
- You can also ssh into the infrastructure nodes and run the below command to verify that all service are up
```bash
hatop -s /run/haproxy.stat
```

# Openstack post installation configurations

- ssh into the utility container
```bash
ssh infra01_utility...
```
- Create external network
  - for test environment run the following
  ```bash
  EXT_NET_VLAN_ID='102'
  EXT_NET_CIDR='10.20.30.0/24'
  EXT_NET_RANGE='start=10.20.30.50,end=10.20.30.100'
  EXT_NET_GATEWAY='10.20.30.1'

  openstack network create --external --provider-physical-network vlan --provider-network-type vlan --provider-segment ${EXT_NET_VLAN_ID} public
  openstack subnet create --no-dhcp --allocation-pool ${EXT_NET_RANGE} --network public --subnet-range ${EXT_NET_CIDR} --gateway ${EXT_NET_GATEWAY} public-subnet
  ```
  - for staging environment run the following
  ```bash
  EXT_NET_VLAN_ID='102'
  EXT_NET_CIDR='10.20.30.0/24'
  EXT_NET_RANGE='start=10.20.30.101,end=10.20.30.200'
  EXT_NET_GATEWAY='10.20.30.1'

  openstack network create --external --provider-physical-network vlan --provider-network-type vlan --provider-segment ${EXT_NET_VLAN_ID} public
  openstack subnet create --no-dhcp --allocation-pool ${EXT_NET_RANGE} --network public --subnet-range ${EXT_NET_CIDR} --gateway ${EXT_NET_GATEWAY} public-subnet
  ```
- Create private network
```bash
PRV_NET_CIDR='192.168.1.0/24'
PRV_NET_GATEWAY='192.168.1.1'

openstack network create --provider-network-type geneve private
openstack subnet create --subnet-range ${PRV_NET_CIDR} --network private --gateway ${PRV_NET_GATEWAY} --dns-nameserver 8.8.8.8 private-subnet
```
- Create router and connect the public and private network
```bash
openstack router create router
openstack router add subnet router private-subnet
openstack router set --external-gateway public router
```
- Create resource flavors
```bash
openstack flavor create --id 1 --ram 512 --disk 1 --vcpus 1 m1.tiny
openstack flavor create --id 2 --ram 2048 --disk 20 --vcpus 1 m1.small
openstack flavor create --id 3 --ram 4096 --disk 40 --vcpus 2 m1.medium
openstack flavor create --id 4 --ram 8192 --disk 80 --vcpus 4 m1.large
openstack flavor create --id 5 --ram 16384 --disk 160 --vcpus 8 m1.xlarge
```
- Configure security group to open ssh and ping
```bash
ADMIN_USER_ID=$(openstack user list | awk '/ admin / {print $2}')
ADMIN_PROJECT_ID=$(openstack project list | awk '/ admin / {print $2}')
ADMIN_SEC_GROUP=$(openstack security group list --project ${ADMIN_PROJECT_ID} | awk '/ default / {print $2}')

openstack security group rule create --ingress --ethertype IPv4 --protocol icmp ${ADMIN_SEC_GROUP}
openstack security group rule create --ingress --ethertype IPv4 --protocol tcp --dst-port 22 ${ADMIN_SEC_GROUP}
```
- Create instance images
  - for cirros image
  ```bash
  apt install wget -y
  wget http://download.cirros-cloud.net/0.6.2/cirros-0.6.2-x86_64-disk.img
  IMAGE=cirros-0.6.2-x86_64-disk.img
  GLANCE_NAME=cirros-0.6.2-x86_64-disk
  IMAGE_TYPE=linux

  openstack image create --disk-format qcow2 --container-format bare --public --property hw_scsi_model=virtio-scsi --property hw_disk_bus=scsi --property hw_qemu_guest_agent=yes --property os_require_quiesce=yes --property os_type=${IMAGE_TYPE} --file ${IMAGE} ${GLANCE_NAME}
  ```
  - for ubuntu server 22.04 image
  ```bash
  apt install wget -y
  wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
  IMAGE=jammy-server-cloudimg-amd64.img
  GLANCE_NAME=jammy-server-cloudimg-amd64
  IMAGE_TYPE=linux

  openstack image create --disk-format qcow2 --container-format bare --public --property hw_scsi_model=virtio-scsi --property hw_disk_bus=scsi --property hw_qemu_guest_agent=yes --property os_require_quiesce=yes --property os_type=${IMAGE_TYPE} --file ${IMAGE} ${GLANCE_NAME}
  ```
  - for ubuntu server 20.04 image
  ```bash
  apt install wget -y
  wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
  IMAGE=focal-server-cloudimg-amd64.img
  GLANCE_NAME=focal-server-cloudimg-amd64
  IMAGE_TYPE=linux

  openstack image create --disk-format qcow2 --container-format bare --public --property hw_scsi_model=virtio-scsi --property hw_disk_bus=scsi --property hw_qemu_guest_agent=yes --property os_require_quiesce=yes --property os_type=${IMAGE_TYPE} --file ${IMAGE} ${GLANCE_NAME}
  ```
  - for ubuntu server 23.04 image
  ```bash
  apt install wget -y
  wget https://cloud-images.ubuntu.com/lunar/current/lunar-server-cloudimg-amd64.img
  IMAGE=lunar-server-cloudimg-amd64.img
  GLANCE_NAME=lunar-server-cloudimg-amd64
  IMAGE_TYPE=linux

  openstack image create --disk-format qcow2 --container-format bare --public --property hw_scsi_model=virtio-scsi --property hw_disk_bus=scsi --property hw_qemu_guest_agent=yes --property os_require_quiesce=yes --property os_type=${IMAGE_TYPE} --file ${IMAGE} ${GLANCE_NAME}
  ```
  - for centos image
  ```bash
  apt install wget -y
  wget https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2
  IMAGE=CentOS-7-x86_64-GenericCloud.qcow2
  GLANCE_NAME=CentOS-7-x86_64-GenericCloud
  IMAGE_TYPE=linux

  openstack image create --disk-format qcow2 --container-format bare --public --property hw_scsi_model=virtio-scsi --property hw_disk_bus=scsi --property hw_qemu_guest_agent=yes --property os_require_quiesce=yes --property os_type=${IMAGE_TYPE} --file ${IMAGE} ${GLANCE_NAME}
  ```
# Testing the openstack using rally

Openstack cloud can be tested from deployment node by install rally
- Install rally using pip, you may need to install pip if it is not installed on your system first
```bash
sudo apt install python3-pip
pip install rally-openstack
```
- The you need to create rally database before running any test, this will create sqlite db to store the test result of the tests
```bash
rally db create
```
- Then you need to get the openrc file of the openstack cloud you want to test, and place it in a suitable location
- Source the openrc file
```bash
source openrc
```
- Create a rally deployment based on the sourced openrc file
```bash
rally deployment create --fromenv --name=deployment_1
```
- Run a deployment check
```bash
rally deployment check
```
- Clone the rally test cases repository to the deployment machine
```bash
git clone https://github.com/openstack/rally-openstack.git
```
- Copy the test cases to a suitable location
```bash
mkdir rally-tests
cp rally-openstack/samples/tasks/scenarios/* rally-tests
```
- Verify, edit and run task
```bash
nano rally-tests/nova/boot-and-delete.yaml
rally task start rally-tasts/nova/boot-and-delete.yaml
```
- When editing a rally task, either to change the flavor , image ..., you can use the openstack cli right from the deployment node since it will be install with rally and you already have access to the openstack cloud with the sourcing of the openrc file
- Once the task run is completed you can export html report using the following command
```bash
rally task report --out=report1.html
```

- ref: https://rally.readthedocs.io/en/latest/index.html

# Appendix

## Temporary netplan file
- After installation of operating system, initially we may not have internet on the system and we may need to manually enter the netplan configuration file instead of copy pasting, in that case below is a minimal netplan configuration file
```yaml
network:
  ethernets:
    interface0: {}
  vlans:
    interface0.x:
      id: x
      link: interface0
      addresses:
      - 172.2x.236.x/22
      routes:
      - to: 0.0.0.0/0
        via: 172.2x.236.x      
  version: 2
```

## Installing monitoring packages

- Download and install zabbix monitoring packages
```bash
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
```
```bash
dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
```
```bash
apt update
```
```bash
apt install zabbix-agent2 zabbix-agent2-plugin-*
```
- Edit the zabbix agent configuration file to point it to the monitoring server ip, and also set the hostname variable
```bash
nano /etc/zabbix/zabbix_agent2.conf
```
- Enable and start the service
```bash
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2
```
## Installing management packages

- Download, install and start cockpit packages
```bash
. /etc/os-release
sudo apt install -t ${VERSION_CODENAME}-backports cockpit -y
systemctl start cockpit
```
- Edit the cockpit configuration file to allow root login, and comment out the root line
```bash
nano /etc/cockpit/disallowed-users
```
- restart cockpit service
```bash
systemctl restart cockpit
```


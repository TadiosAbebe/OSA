# Errors and Solutions
---
### Error:
Horizon dashboard sometimes having an issue retriving instance details, launching console,... with an error TOO MANY CONNECTIONS, SQLALCHEMY.EXC.OPERATIONALERROR: (PYMYSQL.ERR.OPERATIONALERROR) (1040, U’TOO MANY CONNECTIONS’)
### Solution:
Connect to the galera container of your openstack deployment `lxc-attach infra01_galaera_xxxx`, login to your mariadb database `mysql`, the view what is the currently set value for max_connection
- `show variables like '%connection%'`
```
MariaDB [(none)]> show variables like '%connection%';
+---------------------------+--------------------+
| Variable_name             | Value              |
+---------------------------+--------------------+
| character_set_connection  | utf8mb3            |
| collation_connection      | utf8mb3_general_ci |
| default_master_connection |                    |
| extra_max_connections     | 10                 |
| max_connections           | 800                |
| max_user_connections      | 0                  |
+---------------------------+--------------------+
6 rows in set (0.001 sec)
```
- Check max used connections and connection error: `show global status like '%connections%'`
```
MariaDB [(none)]> show global status like '%connections%';
+-----------------------------------+-------+
| Variable_name                     | Value |
+-----------------------------------+-------+
| Connection_errors_max_connections | 0     |
| Connections                       | 77702 |
| Max_used_connections              | 802   |
| Slave_connections                 | 0     |
| wsrep_open_connections            | 0     |
+-----------------------------------+-------+
5 rows in set (0.002 sec)
```
- Set the max_connections to a higher number, `set global max_connections=1024;`
```
MariaDB [(none)]> set global max_connections=1024;
Query OK, 0 rows affected (0.000 sec)
```
---
### Error:
Openstack volume stuck on reserved state and unable to delete the volume. Trying to delete the volume will result in the error "Error: You are not allowed to delete volume: 1d30fe4c-c329-431d-8062-4deedf276ff5" on horizon
### Solution:
Setting the openstack volume to available state will allow you to delete the volume. Use `openstack volume list` to find the volume id, and set it's state available using the following command `openstack volume set --state available volume_id`
---
### Error:
- Instance creation failed with a log error message on the compute node `journalctl -u nova-compute`  Failed to start libvirt guest: libvirt.libvirtError: XML error: invalid secret uuid 'openstack'
### Solution:
- It seems like the secret uuid 'openstack' is read from the hardcoded sercret openstack inside the user_secret.yml file which is now deperciated
---
### Error:
- os-nova playbook fail on TASK [ceph_client : Define libvirt nova secret] with the following error
	fatal: [compute01]: FAILED! => {"changed": true, "cmd": ["virsh", "secret-define", "--file", "/tmp/nova-secret.xml"], "delta": "0:00:00.019320", "end": "2023-11
-05 15:12:25.649258", "msg": "non-zero return code", "rc": 1, "start": "2023-11-05 15:12:25.629938", "stderr": "error: Failed to set attributes from /tmp/nova-s
ecret.xml\nerror: internal error: malformed uuid element", "stderr_lines": ["error: Failed to set attributes from /tmp/nova-secret.xml", "error: internal error:
 malformed uuid element"], "stdout": "", "stdout_lines": []}
	fatal: [compute02]: FAILED! => {"changed": true, "cmd": ["virsh", "secret-define", "--file", "/tmp/nova-secret.xml"], "delta": "0:00:00.018461", "end": "2023-11
-05 15:12:25.766331", "msg": "non-zero return code", "rc": 1, "start": "2023-11-05 15:12:25.747870", "stderr": "error: Failed to set attributes from /tmp/nova-s
ecret.xml\nerror: internal error: malformed uuid element", "stderr_lines": ["error: Failed to set attributes from /tmp/nova-secret.xml", "error: internal error:
 malformed uuid element"], "stdout": "", "stdout_lines": []}
### Solution:
- generate a uuid using uuidgen and place that uuid in user_variables.yml for the 'cinder_ceph_client_uuid' entry
---
### Error:
- setup-infrastructure.yml failing on TASK [repo_server : Retrieve upper constraints using https] with error
	fatal: [infra01_repo_container-54a7b385 -> localhost]: FAILED! => {"changed": false, "dest": "/etc/openstack_deploy/upper-constraints/upper_cons
traints_8a9f69d09be03926242bfc2a04168a166f23209f.txt", "elapsed": 0, "msg": "Request failed: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] cer
tificate verify failed: certificate is not yet valid (_ssl.c:1131)>", "url": "https://releases.openstack.org/constraints/upper/8a9f69d09be039262
42bfc2a04168a166f23209f"}
### Solution:
- update and upgrade system, if you encounter temporary failure resolving 'archive.unbuntu.com' error `echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null`
- if you enconter is not valid yet(invalid for another 13h) error `sudo hwclock --hctosys`
---
### Error:
- setup-hosts.ymal failing on TASK: check how kernel modules are implemented (statically builtin, dynamic, not set) with error
	fatal: [infra01]: FAILED! => {"changed": false, "msg": "file not found: /boot/config-5.4.0-165-generic"}
	fatal: [storage01]: FAILED! => {"changed": false, "msg": "file not found: /boot/config-5.4.0-165-generic"}
	fatal: [compute01]: FAILED! => {"changed": false, "msg": "file not found: /boot/config-5.4.0-165-generic"}
### Solution:
- This is maily caused by different kernel version on the deployment and target host, starting from a clean installation or an earlier snapshot will solve the probelm
---
### Error:
- setup-infrastructure.yml failing on TASK: Fail if galera_cluster_name doesnt match provided value with the error The galera_cluster_name variable does not match what is set in mysql
### Solution:
- This error was happening when deploying openstack in hyperconverge mode with the ceph cluster, separating the ceph cluster onto its own machine has solved the issue
---
### Error:
1. Unable to login into openstack horizon dashboard. "Something went wrong" error afer putting in username and password
2. When verifing openstack services after deployment, issuing "openstack network agent list" on inside the utility contianer on the infrastructure nodes, it results in 503 error
	- HttpException: 503: Server Error for url: http://10.10.36.9:9696/v2.0/agents, 503 Service Unavailable: No server is available to handle this request.
3. When investigating the neutron service, by navigating to the neutron-server container on the infrastructure nodes ans issuing systemctl status neturon-server, the service is flactuating by comming up and down

### Solution:
The reason neutron-server is not starting was due to incompatable coniguration set in place on the openstack user conifg file and else where, on the zed release OSA switched from linux bridges to ovn and from vxlan to geneve.
- changed network type from vxlan to geneve
- changed group_binds from neutron_linuxbridge_agent to neutron_ovn_controller
- added additional configurations on user_variables.yml
- created files and configurations in /etc/openstack_deploy/env.d/neutron.yml, /etc/openstack_deploy/env.d/nova.yml, /etc/openstack_deploy/group_vars/network_hosts

ref: https://bugs.launchpad.net/openstack-ansible/+bug/2002897

---
### Error:
After reboot the mariadb service inside the galera containers of the infrastructure nodes is not starting
Attaching the lxc container of galera on the infrastructure node and running systemctl status mariadb shows the process is failed and looking further in the logs it shows
```
WSREP: gcs connect failed: Connection timed out
WSREP: wsrep::connect(gcomm://172.29.239.234,172.29.239.9,172.29.237.176) failed: 7
Aborting
mariadb.service: Main process exited, code=exited, status=1/FAILURE
mariadb.service: Failed with result 'exit-code'.
Failed to start MariaDB 10.6.10 database server.
```

### Solution:
Inside the galera containers of the infrastructure nodes
- naviage to /var/lib/mysql/grastate.dat file
- change the value of safe_to_bootstrap to 1
- sed -i "/safe_to_bootstrap/s/0/1/" /var/lib/mysql/grastate.dat
- now restart the mariadb service or running galera_new_cluster seems to fix the problem

ref: https://stackoverflow.com/questions/37212127/mariadb-gcomm-backend-connection-failed-110 https://galeracluster.com/library/documentation/crash-recovery.html

---
### Error:
openstack-ansible setup-openstack.yml script keeps failing on setting up gnocchi on "python_venv_build : Build wheels for the packages to be installed into the venv" Task to be specific. I was trying to deploy the Zed release of OpenStack, but the issue still persists when trying to deploy the Antelope release of OpenStack. When I look at the python log file generated from the venv build, I see a bunch of "# Package would be ignored # warning" about 18 of them; some of those are for the packages gnoochi.cli, gnoochi.incoming, gnoochi.indexer, gnoochi.rest, gnoochi.storage, gnoochi.test, and so on... and it says "AttributeError: module 'setuptools.command.easy_install' has no attribute 'get_script_header'"


### Solution:
I need to apply both of the aforementioned patches for it to work for me.
- adding "gnocchi_git_install_branch: 6f35ea5413a9f78551d8193b8d2a6d77c49b6372" to /etc/openstack_deploy/user_variables.yml
- adding "setuptools==67.8.0" to /opt/openstack-ansible/global-requirement-pins.txt

ref: https://bugs.launchpad.net/openstack-ansible/+bug/2026828

---

### Error:
can't create an admin network from horizon dashboard Admin > Network > Networks > Create Network provides "Danger: An error occured. Please try again later." and it is returning 500 Internal Server Error. from devtools. journalctl -f -u apache2 says "Undefined provider network types are found: ['v', 'l', 'a', 'n', ',', 'l', 'o', 'c', 'a', 'l', ',', 'g', 'e', 'n', 'e', 'v', 'e']" when trying to create a network
### Solution: 
Given that you've defined neutron_ml2_drivers_type to their defaults, removing this variable and re-running os-horizon-install.yml should fix the issue

ref: https://review.opendev.org/c/openstack/openstack-ansible-os_horizon/+/888917

---

### Error:
After successful deployment of openstack. instance can not be created with the following cinder error "did not finish being created even after we waited 3 seconds or 2 attempts. And its status is error"
### Solution: 
This happens because of the instance image you are using to create the virtual machine, so first verify that cinder is working by creating a cinder volume under volume > create volume, if this completed successfully the remove you openstack image and try to upload it properly.

---

# Chat with OSA-team
1. Neutron troubleshooting
2. Openstack_user_config troubleshooting

- <Tad> mgariepy Here is my neutron and connectivity issue, instances can communicate with each other on their private addresses, i.e. 192.168.10.144 is able to ping 192.168.10.160 and vice versa, but they are not able to go out to the internet, i.e. ping 8.8.8.8
- <Tad> My configuration files are located at https://github.com/TadiosAbebe/OSA/blob/master/etc/openstack_deploy and  I have modified the openstack_user_config.yml, user_variables.yml, ./group_vars/network_hosts, /env.d/neutron.yml and /env.d/nova.yml files.
- <Tad> Here is a detail description of what i tried and how my enviroment is setup https://pastebin.com/W9xy0EWC
	- <noonedeadpunk> Tad: first thing I can tell - `is_container_address` should be defined only once. And that supposed to be br-mgmt
	- <noonedeadpunk> one small nit - br-vxlan is super confusing given that this is geneve - 2 different overlay protocols. 
	- Also - you have all bridges except br-storage as linux bridges, and br-storage as ovs one?  I would either have all bridges on controllers as OVS or all as just linux bridges, not mix of them. But there is technically nothing wrong in mixing them if there is a reason for that. in your case it should be likely easier to drop `container_bridge_type: "openvswitch"` and re-configure br-storage as simple bridge in your netplan by removing the openvswitch.
	- log_hosts is likely will not have any effect - we've dropped rsyslog roles as logs are managed with journald
	- I think you're missing `network-gateway_hosts` and `network-northd_hosts` https://docs.openstack.org/openstack-ansible-os_neutron/latest/app-ovn.html#deployment-scenarios
	- ovn gateway is exactly the thing that is repsonsible for external connectivity
	- controller gateway is implicitly included in network-hosts, but ovn gateway is totally not as there're multiple scenarios where to place them, and usually that's not control plane either compute nodes or standalone network nodes
	- defined network-gateway_hosts and network-northd_hosts
	- once you will change env.d/conf.d/openstack_user_config you will likely need to re-run lxc-containers-create.yml playbook
	- the host_bind_override: "bond1" is this necessary? idea behind that, is that there might be no bridge as br-vlan - as it's absolutely fine to have just interface instead of the bridge on network/compute hosts and it is not needed on storage/controller hosts at all. so to avoid creating br-vlan bridge, you can just have an interface and mark that with `host_bind_override`
	- if you have br-vlan bridge on compute/network hosts - you don't need host_bind_override then. basically what will happen with that bridge - it will be added as "interface" to another bridge, while br-vlan would have only 1 interface in it. It was named as "bridge" only for consistency and naming things in the same way across all docs. same with br-vxlan actually
	- When specifying network-northd_hosts network-gateway_hosts in openstack_user_config do i need to remove the network_hosts: portion or should i keep it? no, keep that, it is needed for neutron-server.
	- what i have been doing so far is running all seutp-host, infrastructure and openstack after changing any configuration, is that the proper way. well. it's long way, it would work though. short path would be to run lxc-containers-create and then affected roles. For example, if you're changing neutron configuration - run os-neutron-install.yml afterwards. and lxc-containers-create is needed only when you expect changes in inventory, that would result in creating new containers. and what you can do - run `openstack-ansible lxc-containers-destroy.yml --limit neutron_server` and re-create these with `openstack-ansible lxc-containers-create.yml --limit neutron_server,localhost`
	- if there is a power interruption and all my 3 servers losses power, galera cluster won’t start up and I have to go and manually do galera_new_cluster on the node where safe_to_bootstrap is 1 inside the /var/lib/mysql/grastate.dat am I doing something wrong or is there a more permanent solution.  nah, I guess it's "relatively" fair recovery process of galera. Same actually happens in split-brain, when it did not record last sequence number (due to unexpected shutdown) - it doesn't know where latest data is so yes, you need to tell it which one should act as "master"
	- well ,actually, I dunno if you knew that or not, but you can create multiple containers of same type on a single host: https://docs.openstack.org/openstack-ansible/latest/reference/inventory/configure-inventory.html#deploying-0-or-more-than-one-of-component-type-per-host so you can have 3 galera containers on the 1 host to play with clustering, for instance
	- Well, power loss of all DC can happen, ofc, but mysql would be your least problem in case of this happening. like storage! instances that are runnning using buffers, so they can easily get broken FS inside them
-  Having this openstack_user_config now https://pastebin.com/XLduZ9uK setup-openstack fails on TASK [os_neutron : Setup Network Provider Bridges] with the error "ovs-vsctl: cannot create a bridge named br-vlan because a port named br-vlan already exists on bridge br-provider"
	- <noonedeadpunk> Um... honestly I'm not sure here. And you have pre-created the br-vlan with netplan config?
	- <Tad> yes i have br-vlan on bond1 of my netplan
	- <Tad> noonedeadpunk: when i put back host_bind_override: "bond1" on container_bridge: "br-vlan" os-neutron-install.yml completed without error
	- <noonedeadpunk> Tad: so from VM you can't reach router gateway, right?
	- <Tad> noonedeadpunk yes they cant reach the gateway
	- <jamesdenton> sure, so the first problem, then is getting them to ping the tenant gateway
	- <jamesdenton> there's only 1 compute?
	- <Tad> yes
	- <jamesdenton> can you do me a favor and provide the output of "ovs-vsctl list open_vswitch" from all 3 nodes?
	```
	root@infra01:~# ovs-vsctl list open_vswitch
	_uuid               : 39a9c6a4-d941-4f3e-88b6-ff6a5e8c8a93
	bridges             : []
	cur_cfg             : 74
	datapath_types      : [netdev, system]
	datapaths           : {}
	db_version          : "8.2.0"
	dpdk_initialized    : false
	dpdk_version        : none
	external_ids        : {hostname=infra01, rundir="/var/run/openvswitch", system-id="83f0db97-6759-4986-90e6-e67bffcfa466"}
	iface_types         : [erspan, geneve, gre, internal, ip6erspan, ip6gre, lisp, patch, stt, system, tap, vxlan]
	manager_options     : []
	next_cfg            : 74
	other_config        : {}
	ovs_version         : "2.13.8"
	ssl                 : []
	statistics          : {}
	system_type         : ubuntu
	system_version      : "20.04"

	root@compute01:~# ovs-vsctl list open_vswitch
	_uuid               : 26903f03-32e7-4f03-bc60-4e57507524dc
	bridges             : [aa4a80da-2869-45c9-8e77-f60e01923ee7, ba275680-512b-41fc-aca0-26cf86b6dc7f, dcb14645-e397-438a-815a-f1ce1f68f2ec]
	cur_cfg             : 112
	datapath_types      : [netdev, system]
	datapaths           : {system=19a46066-ef6c-4a43-931c-81f3703219a5}
	db_version          : "8.3.0"
	dpdk_initialized    : false
	dpdk_version        : none
	external_ids        : {hostname=compute01, ovn-bridge-mappings="vlan:bond1", ovn-cms-options=enable-chassis-as-gw, ovn-encap-ip="172.29.240.21", ovn-encap-type=geneve, ovn-remote="ssl:172.29.237.159:6642,ssl:172.29.236.111:6642,ssl:172.29.239.169:6642", rundir="/var/run/openvswitch", system-id="2eecff48-2b90-4af7-9e00-4a957eeba15b"}
	iface_types         : [bareudp, erspan, geneve, gre, gtpu, internal, ip6erspan, ip6gre, lisp, patch, stt, system, tap, vxlan]
	manager_options     : [20dfe4f3-ba38-4197-8a4b-3e432a9ad212]
	next_cfg            : 112
	other_config        : {vlan-limit="0"}
	ovs_version         : "2.17.5"
	ssl                 : []
	statistics          : {}
	system_type         : ubuntu
	system_version      : "20.04"

	root@storage01:~# ovs-vsctl list open_vswitch
	_uuid               : 8ea535b8-f48c-448d-930d-25af774da575
	bridges             : []
	cur_cfg             : 74
	datapath_types      : [netdev, system]
	datapaths           : {}
	db_version          : "8.2.0"
	dpdk_initialized    : false
	dpdk_version        : none
	external_ids        : {hostname=storage01, rundir="/var/run/openvswitch", system-id="33b16e55-778f-4f48-8335-75507b7f9ea1"}
	iface_types         : [erspan, geneve, gre, internal, ip6erspan, ip6gre, lisp, patch, stt, system, tap, vxlan]
	manager_options     : []
	next_cfg            : 74
	other_config        : {}
	ovs_version         : "2.13.8"
	ssl                 : []
	statistics          : {}
	system_type         : ubuntu
	system_version      : "20.04"
	```
	- <jamesdenton> on the compute, can you show me the 'ovs-vsctl show' output?
	```
	root@compute01:~# ovs-vsctl show
	26903f03-32e7-4f03-bc60-4e57507524dc
		Manager "ptcp:6640:127.0.0.1"
			is_connected: true
		Bridge bond1
			fail_mode: standalone
			Port patch-provnet-0339a086-dd6e-442e-bc76-ecad6aa8f194-to-br-int
				Interface patch-provnet-0339a086-dd6e-442e-bc76-ecad6aa8f194-to-br-int
					type: patch
					options: {peer=patch-br-int-to-provnet-0339a086-dd6e-442e-bc76-ecad6aa8f194}
			Port bond1
				Interface bond1
					type: internal
					error: "could not add network device bond1 to ofproto (File exists)"
		Bridge br-int
			fail_mode: secure
			datapath_type: system
			Port tap22eb0bf4-19
				Interface tap22eb0bf4-19
			Port br-int
				Interface br-int
					type: internal
			Port tap2355f4d0-76
				Interface tap2355f4d0-76
			Port tapdc914ac5-af
				Interface tapdc914ac5-af
			Port tape4df664c-68
				Interface tape4df664c-68
			Port patch-br-int-to-provnet-0339a086-dd6e-442e-bc76-ecad6aa8f194
				Interface patch-br-int-to-provnet-0339a086-dd6e-442e-bc76-ecad6aa8f194
					type: patch
					options: {peer=patch-provnet-0339a086-dd6e-442e-bc76-ecad6aa8f194-to-br-int}
			Port tap77086df1-a0
				Interface tap77086df1-a0
		Bridge br-provider
			fail_mode: standalone
			Port br-vlan
				Interface br-vlan
			Port br-provider
				Interface br-provider
					type: internal
		ovs_version: "2.17.5"
	```
	- <jamesdenton> ok, so i lied. I see ovn-bridge-mappings="vlan:bond1", which implies bond1 is a bridge, and that vsctl show output confirms that. I suspect you mean for bond1 to be the interface used, and that should be connected to a bridge (likely br-provider)
	- <jamesdenton> the openstack_user_config.yml shows br-vlan, though, so maybe br-provider was rolled by hand later?
	- <anskiy> there is bridge called br-provider, which contanins port br-vlan (like in the error you mentioned before), and in your netplan config, br-vlan is the linux bridge with bond1. At the same time, bond1 is a port in OVS bridge bond1...
	- <anskiy> You might need to delete OVS bridge br-provider, maybe?.. As I don't really see, where it's been used
	- <jamesdenton> Mine looks like this: https://paste.opendev.org/show/bLkYnCApAH4vXAULykQk/. Playbooks would create br-provider (if it doesn't exist) and connect bond1 to br-provider (bond1 must be an existing interface)
	```
	- network:
		group_binds:
			- neutron_ovn_controller
		container_bridge: br-provider
		network_interface: "bond1"
		type: "vlan"
		net_name: "vlan"
    - network:
        group_binds:
          - neutron_ovn_controller
        container_bridge: "br-overlay"
        ip_from_q: "tunnel"
        type: "geneve"
        range: "9900:9999"
        net_name: "geneve"
	```
	- <Tad> so there is a bunch of "failed to add bond1 as port: File exists" inside ovs-vswitchd.log and i think this is happing because i added back the host_bind_override: "bond1" on my config. but without that the playbook fails
	- <jamesdenton> host_bind_override should only be for linuxbridge
	- <jamesdenton> try using "network_interface: bond1" instead. br-vlan is probably also unnecessary
	- <jamesdenton> for OVS/OVN, you want an OVS provider bridge connected directly to a physical interface. The snippet i sent would result in the playbooks creating br-provider and connecting bond1.
	- <jamesdenton> wanna ship that config over first so we can glance at it?
	- <Tad> netplan: https://pastebin.com/6erRikBP  user_variable https://pastebin.com/7qxF9rBG https://pastebin.com/XLduZ9uK
	- <Tad> so the above configs seems okay?
	- <jamesdenton> ok, so my suggestion would be to remove br-vlan from netplan
	- <jamesdenton> then, in openstack_user_config, in the br-vlan network block, add 'network_interface: "bond1"' under 'container_bridge: "br-vlan"'. br-vlan will end up being created as an OVS bridge
	- <Tadios> noonedeadpunk jamesdenton anskiy, After yesterday's suggestions and recommendations, I was able to fix my networking issues. and thank you very much. Now my instances are able to communicate with the outside, but I have a few more questions.
	- <Tadios> 1. I still can't create an admin network from horizon dashboard Admin > Network > Networks > Create Network provides "Danger: An error occured. Please try again later." and it is returning 500 Internal Server Error. from devtools. I dont know where to look the log files for this error? I tried /var/apache2/error.log but notting.
	- <noonedeadpunk> Tadios: it should be in journald for apache2 unit. like `journalctl -f -u apache2`
	- <Tadios> i should try this from inside horizon container right?
	- <noonedeadpunk> yup. you can do outside as well, but you'd need to select correct path with db
	- <noonedeadpunk> inside /var/log/journal
	- <Tadios> it says "Undefined provider network types are found: ['v', 'l', 'a', 'n', ',', 'l', 'o', 'c', 'a', 'l', ',', 'g', 'e', 'n', 'e', 'v', 'e']" when trying to create a network
	- <noonedeadpunk> ah
	- <Tadios> does it have something to do with this specification in my user_variables.yml "neutron_ml2_drivers_type: "vlan,local,geneve""
	- <noonedeadpunk> Tadios: that looks like quite valid bug actually
	- <opendevreview> Dmitriy Rabotyagov proposed openstack/openstack-ansible-os_horizon master: Fix wrong neutron_ml2_drivers_type  https://review.opendev.org/c/openstack/openstack-ansible-os_horizon/+/888917
	- <noonedeadpunk> Tadios: this should fix it ^
	- <noonedeadpunk> but yes, given that you've defined neutron_ml2_drivers_type to their defaults, removing this variable and re-running os-horizon-install.yml should fix the issue
	- <noonedeadpunk> (you should be able to add --tags horizon-config to reduce time of running the playbook)
	- <Tadios> and my second question was my vms can't still ping the gateway and i don't know what fixed the issue but it is working now, could it be the horizon fix or something else?
	- <Tadios> oh my bad, it was Security Groups. and last question
	- <Tadios> Do we need to specify the container_type: "veth" in the provider_network: section of the openstack_user_config or is it optional? It is listed as required in the documentation. Also, what about container_interface? I asked this because I don't see these options on the configuration jamesdenton shared https://paste.opendev.org/show/bLkYnCApAH4vXAULykQk/
	- <noonedeadpunk> I'm not 100% sure, but I'd say yes
	- <NeilHanlon> would guess veth is required for plumbing the pseudowires
	- <noonedeadpunk> I think that `container_type` would be passed to lxc config and then it's up to lxc defaults 
	- <noonedeadpunk> `container_interface` is needed only for groups that have containers in fact. Like it is needed for neutron-server, but it is not for nova-compute, for instance

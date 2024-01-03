# Errors and Solutions

---
### Error:
After a successful deployment of openstack 27.3.0, `openstack network agent list` command on the utilitiy container will only list one OVN Controller Gateway agent, while there should be 3, going into either the neutron-server container and viewing the journal logs or going into one of the nodes where the neutron-ovn-metadata-agent service is running and viewing the journal logs will show the following error
```
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn [-] OVS database connection to OVN_Southbound failed with error: 'non-zero flags not allowed in calls to send() on <class 'eventlet.green.ssl.GreenSSLSocket'>'. Verify that the OVS and OVN services are available and that the 'ovn_nb_connection' and 'ovn_sb_connection' configuration options are correct.: ValueError: non-zero flags not allowed in calls to send() on <class 'eventlet.green.ssl.GreenSSLSocket'>
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn Traceback (most recent call last):
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/neutron/plugins/ml2/drivers/ovn/mech_driver/ovsdb/impl_idl_ovn.py", line 126, in start_connection
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     self.ovsdb_connection.start()
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovsdbapp/backend/ovs_idl/connection.py", line 83, in start
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     idlutils.wait_for_change(self.idl, self.timeout)
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/neutron/plugins/ml2/drivers/ovn/mech_driver/ovsdb/impl_idl_ovn.py", line 52, in wait_for_change
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     while idl_.change_seqno == seqno and not idl_.run():
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/db/idl.py", line 433, in run
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     self._session.run()
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/jsonrpc.py", line 519, in run
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     error = self.stream.connect()
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/stream.py", line 817, in connect
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     retval = super(SSLStream, self).connect()
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/stream.py", line 300, in connect
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     self.__scs_connecting()
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/stream.py", line 268, in __scs_connecting
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     retval = self.check_connection_completion(self.socket)
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/stream.py", line 777, in check_connection_completion
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     return Stream.check_connection_completion(sock)
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/stream.py", line 137, in check_connection_completion
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     return ovs.socket_util.check_connection_completion(sock)
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/ovs/socket_util.py", line 181, in check_connection_completion
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     sock.send("\0".encode(), socket.MSG_DONTWAIT)
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/eventlet/green/ssl.py", line 194, in send
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     return self._call_trampolining(
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/openstack/venvs/neutron-27.3.0/lib/python3.10/site-packages/eventlet/green/ssl.py", line 158, in _call_trampolining
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     return func(*a, **kw)
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn   File "/usr/lib/python3.10/ssl.py", line 1232, in send
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn     raise ValueError(
ERROR neutron.plugins.ml2.drivers.ovn.mech_driver.ovsdb.impl_idl_ovn ValueError: non-zero flags not allowed in calls to send() on <class 'eventlet.green.ssl.GreenSSLSocket'>
```
### Solution:
Restarting the whole infrastructure seems to solve the problem and ovn metadata agent is appearing on all the compute hosts when issuing `openstack network agent list`
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

Instance creation failed with a log error message on the compute node `journalctl -u nova-compute`  Failed to start libvirt guest: libvirt.libvirtError: XML error: invalid secret uuid 'openstack'

### Solution:

- It seems like the secret uuid 'openstack' is read from the hardcoded sercret openstack inside the user_secret.yml file which is now deperciated

---

### Error:

os-nova playbook fail on TASK [ceph_client : Define libvirt nova secret] with the following error
	fatal: [compute01]: FAILED! => {"changed": true, "cmd": ["virsh", "secret-define", "--file", "/tmp/nova-secret.xml"], "delta": "0:00:00.019320", "end": "2023-11
-05 15:12:25.649258", "msg": "non-zero return code", "rc": 1, "start": "2023-11-05 15:12:25.629938", "stderr": "error: Failed to set attributes from /tmp/nova-s
ecret.xml\nerror: internal error: malformed uuid element", "stderr_lines": ["error: Failed to set attributes from /tmp/nova-secret.xml", "error: internal error:
 malformed uuid element"], "stdout": "", "stdout_lines": []}
	fatal: [compute02]: FAILED! => {"changed": true, "cmd": ["virsh", "secret-define", "--file", "/tmp/nova-secret.xml"], "delta": "0:00:00.018461", "end": "2023-11
-05 15:12:25.766331", "msg": "non-zero return code", "rc": 1, "start": "2023-11-05 15:12:25.747870", "stderr": "error: Failed to set attributes from /tmp/nova-s
ecret.xml\nerror: internal error: malformed uuid element", "stderr_lines": ["error: Failed to set attributes from /tmp/nova-secret.xml", "error: internal error:
 malformed uuid element"], "stdout": "", "stdout_lines": []}

### Solution:

generate a uuid using uuidgen and place that uuid in user_variables.yml for the 'cinder_ceph_client_uuid' entry

---

### Error:
setup-infrastructure.yml failing on TASK [repo_server : Retrieve upper constraints using https] with error
	fatal: [infra01_repo_container-54a7b385 -> localhost]: FAILED! => {"changed": false, "dest": "/etc/openstack_deploy/upper-constraints/upper_cons
traints_8a9f69d09be03926242bfc2a04168a166f23209f.txt", "elapsed": 0, "msg": "Request failed: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] cer
tificate verify failed: certificate is not yet valid (_ssl.c:1131)>", "url": "https://releases.openstack.org/constraints/upper/8a9f69d09be039262
42bfc2a04168a166f23209f"}

### Solution:

- update and upgrade system, if you encounter temporary failure resolving 'archive.unbuntu.com' error `echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null`
- if you enconter is not valid yet(invalid for another 13h) error `sudo hwclock --hctosys`

---

### Error:

setup-hosts.yml failing on TASK: check how kernel modules are implemented (statically builtin, dynamic, not set) with error
	fatal: [infra01]: FAILED! => {"changed": false, "msg": "file not found: /boot/config-5.4.0-165-generic"}
	fatal: [storage01]: FAILED! => {"changed": false, "msg": "file not found: /boot/config-5.4.0-165-generic"}
	fatal: [compute01]: FAILED! => {"changed": false, "msg": "file not found: /boot/config-5.4.0-165-generic"}

### Solution:
- This is maily caused by different kernel version on the deployment and target host, starting from a clean installation or an earlier snapshot will solve the probelm

---

### Error:

setup-infrastructure.yml failing on TASK: Fail if galera_cluster_name doesnt match provided value with the error The galera_cluster_name variable does not match what is set in mysql

### Solution:

This error was happening when deploying openstack in hyperconverge mode with the ceph cluster, separating the ceph cluster onto its own machine has solved the issue

---

### Error:

- Unable to login into openstack horizon dashboard. "Something went wrong" error afer putting in username and password
- When verifing openstack services after deployment, issuing "openstack network agent list" on inside the utility contianer on the infrastructure nodes, it results in 503 error
	- HttpException: 503: Server Error for url: http://10.10.36.9:9696/v2.0/agents, 503 Service Unavailable: No server is available to handle this request.
- When investigating the neutron service, by navigating to the neutron-server container on the infrastructure nodes ans issuing systemctl status neturon-server, the service is flactuating by comming up and down

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

can't create an admin network from horizon dashboard Admin > Network > 
Networks > Create Network provides "Danger: An error occured. Please try again later." and it is returning 500 Internal Server Error. from devtools. journalctl -f -u apache2 says "Undefined provider network types are found: ['v', 'l', 'a', 'n', ',', 'l', 'o', 'c', 'a', 'l', ',', 'g', 'e', 'n', 'e', 'v', 'e']" when trying to create a network

### Solution: 

Given that you've defined neutron_ml2_drivers_type to their defaults, removing this variable and re-running os-horizon-install.yml should fix the issue

ref: https://review.opendev.org/c/openstack/openstack-ansible-os_horizon/+/888917

---

### Error:

After successful deployment of openstack. instance can not be created with the following cinder error "did not finish being created even after we waited 3 seconds or 2 attempts. And its status is error"

### Solution: 

This happens because of the instance image you are using to create the virtual machine, so first verify that cinder is working by creating a cinder volume under volume > create volume, if this completed successfully the remove you openstack image and try to upload it properly.

---

# Galera Problems and Solutions:
## Single node failier(graceful shutdown)
- During a shutdown of the entire galera_cluster, we need to verify from which galera to bootstrap a new cluster for this we view the status of the cluster members using the folloiwng command `ansible galera_container -m shell -a "cat /var/lib/mysql/grastate.dat"` which has the following output. you need to run this from the deployment host inside the /opt/openstack folder
```
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
infra01_galera_container-08ddf08d | CHANGED | rc=0 >>
# GALERA saved state
version: 2.1
uuid:    1e3da54e-3051-11ee-bbb6-cb65a0f86cdc
seqno:   16060
safe_to_bootstrap: 0
compute01_galera_container-82ba3a78 | CHANGED | rc=0 >>
# GALERA saved state
version: 2.1
uuid:    1e3da54e-3051-11ee-bbb6-cb65a0f86cdc
seqno:   -1
safe_to_bootstrap: 0

storage01_galera_container-d550e8d8 | CHANGED | rc=0 >>
# GALERA saved state
version: 2.1
uuid:    1e3da54e-3051-11ee-bbb6-cb65a0f86cdc
seqno:   -1
safe_to_bootstrap: 1
```
- In the above case we need to bootstrap from the node that have the safe_to_bootstrap value 1
- Once verifying from which node to bootstrap the cluster set the _WSREP_NEW_CLUSTER variable using the following command `ansible galera_container -m shell -a "systemctl set-environment _WSREP_NEW_CLUSTER='--wsrep-new-cluster'" --limit galera_container[2]` make sure to replace the galera_container[2] with the suitable container index, and the above will have the following output
```
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
storage01_galera_container-d550e8d8 | CHANGED | rc=0 >>
```
- Then restart the mysql service on that container `ansible galera_container -m shell -a "systemctl start mysql" --limit galera_container[2]`
```
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
storage01_galera_container-d550e8d8 | CHANGED | rc=0 >>
```
- Unset the previously set _WSREP_NEW_CLUSTER variable using the following command `ansible galera_container -m shell -a "systemctl set-environment _WSREP_NEW_CLUSTER=''" --limit galera_container[2]`
```
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
storage01_galera_container-d550e8d8 | CHANGED | rc=0 >>
```
- Or alternatively instead of the above three steps we can run the following to perform the task at once `ansible galera_container -m shell -a "galera_new_cluster" --limit galera_container[2]`
- To verify the cluster status you can run the following command `ansible galera_container -m shell -a "mysql -e 'show status like \"%wsrep_cluster_%\";'"`
```
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
infra01_galera_container-08ddf08d | FAILED | rc=1 >>
ERROR 2002 (HY000): Can't connect to local server through socket '/var/run/mysqld/mysqld.sock' (2)non-zero return code
storage01_galera_container-d550e8d8 | CHANGED | rc=0 >>
Variable_name   Value
wsrep_cluster_weight    1
wsrep_cluster_capabilities
wsrep_cluster_conf_id   2
wsrep_cluster_size      1
wsrep_cluster_state_uuid        1e3da54e-3051-11ee-bbb6-cb65a0f86cdc
wsrep_cluster_status    Primary
compute01_galera_container-82ba3a78 | FAILED | rc=1 >>
ERROR 2002 (HY000): Can't connect to local server through socket '/var/run/mysqld/mysqld.sock' (111)non-zero return code
```
- As you can see from the above output storage01 galera cluster is operation now, the only thing left to do now is join the rest of the nodes to the cluster and to do that you can run simple `ansible galera_container -m shell -a "systemctl start mysql" --limit galera_container[0]` changing the galera_container[0] index to the rest of the nodes
```
root@deploy /o/openstack-ansible ((26.1.2)) [2]# ansible galera_container -m shell -a "systemctl start mysql" --limit galera_container[0]
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
infra01_galera_container-08ddf08d | CHANGED | rc=0 >>
root@deploy /o/openstack-ansible ((26.1.2))# ansible galera_container -m shell -a "systemctl start mysql" --limit galera_container[1]
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
compute01_galera_container-82ba3a78 | CHANGED | rc=0 >>
```
- At last running `ansible galera_container -m shell -a "mysql -e 'show status like \"%wsrep_cluster_%\";'"` will show that the full cluster is operational
```
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
compute01_galera_container-82ba3a78 | CHANGED | rc=0 >>
Variable_name   Value
wsrep_cluster_weight    3
wsrep_cluster_capabilities
wsrep_cluster_conf_id   4
wsrep_cluster_size      3
wsrep_cluster_state_uuid        1e3da54e-3051-11ee-bbb6-cb65a0f86cdc
wsrep_cluster_status    Primary
infra01_galera_container-08ddf08d | CHANGED | rc=0 >>
Variable_name   Value
wsrep_cluster_weight    3
wsrep_cluster_capabilities
wsrep_cluster_conf_id   4
wsrep_cluster_size      3
wsrep_cluster_state_uuid        1e3da54e-3051-11ee-bbb6-cb65a0f86cdc
wsrep_cluster_status    Primary
storage01_galera_container-d550e8d8 | CHANGED | rc=0 >>
Variable_name   Value
wsrep_cluster_weight    3
wsrep_cluster_capabilities
wsrep_cluster_conf_id   4
wsrep_cluster_size      3
wsrep_cluster_state_uuid        1e3da54e-3051-11ee-bbb6-cb65a0f86cdc
wsrep_cluster_status    Primary
```
- ref: https://docs.openstack.org/openstack-ansible/zed/admin/maintenance-tasks.html

## All node failier(non gracefull shutdown)

- To start the cluster from a non gracefull shutdown we need to know which node has the latest data so we issue the following command `ansible galera_container -m shell -a "mysqld --wsrep-recover"`
```
Variable files: "-e @/etc/openstack_deploy/user_secrets.yml -e @/etc/openstack_deploy/user_variables.yml "
[WARNING]: Unable to parse /etc/openstack_deploy/inventory.ini as an inventory source
infra01_galera_container-08ddf08d | CHANGED | rc=0 >>
2023-08-02 10:45:05 0 [Note] mysqld (server 10.6.10-MariaDB-1:10.6.10+maria~ubu2004-log) starting as process 67468 ...
2023-08-02 10:45:05 0 [ERROR] mysqld: Can't lock aria control file '/var/lib/mysql/aria_log_control' for exclusive use, error: 11. Will retry for 30 seconds
2023-08-02 10:45:06 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2023-08-02 10:45:06 0 [Note] InnoDB: Using transactional memory
2023-08-02 10:45:06 0 [Note] InnoDB: Number of pools: 1
2023-08-02 10:45:06 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2023-08-02 10:45:06 0 [Note] InnoDB: Using Linux native AIO
2023-08-02 10:45:06 0 [Note] InnoDB: Initializing buffer pool, total size = 4294967296, chunk size = 134217728
2023-08-02 10:45:06 0 [Note] InnoDB: Completed initialization of buffer pool
2023-08-02 10:45:06 0 [Note] InnoDB: 128 rollback segments are active.
2023-08-02 10:45:06 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2023-08-02 10:45:06 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2023-08-02 10:45:06 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2023-08-02 10:45:06 0 [Note] InnoDB: 10.6.10 started; log sequence number 18464431; transaction id 105924
2023-08-02 10:45:06 0 [Warning] InnoDB: Skipping buffer pool dump/restore during wsrep recovery.
2023-08-02 10:45:06 0 [Note] Plugin 'FEEDBACK' is disabled.
2023-08-02 10:45:06 0 [Note] Server socket created on IP: '172.29.238.154'.
2023-08-02 10:45:06 0 [Note] Server socket created on IP: '172.29.238.154'.
2023-08-02 10:45:06 0 [Note] WSREP: Recovered position: 1e3da54e-3051-11ee-bbb6-cb65a0f86cdc:20639
Warning: Memory not freed: 280
storage01_galera_container-d550e8d8 | CHANGED | rc=0 >>
2023-08-02  7:45:06 0 [Note] mysqld (server 10.6.10-MariaDB-1:10.6.10+maria~ubu2004-log) starting as process 3081 ...
2023-08-02  7:45:07 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2023-08-02  7:45:07 0 [Note] InnoDB: Using transactional memory
2023-08-02  7:45:07 0 [Note] InnoDB: Number of pools: 1
2023-08-02  7:45:07 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2023-08-02  7:45:07 0 [Note] InnoDB: Using Linux native AIO
2023-08-02  7:45:07 0 [Note] InnoDB: Initializing buffer pool, total size = 4294967296, chunk size = 134217728
2023-08-02  7:45:07 0 [Note] InnoDB: Completed initialization of buffer pool
2023-08-02  7:45:07 0 [Note] InnoDB: 128 rollback segments are active.
2023-08-02  7:45:07 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2023-08-02  7:45:07 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2023-08-02  7:45:07 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2023-08-02  7:45:07 0 [Note] InnoDB: 10.6.10 started; log sequence number 16801401; transaction id 56154
2023-08-02  7:45:07 0 [Warning] InnoDB: Skipping buffer pool dump/restore during wsrep recovery.
2023-08-02  7:45:07 0 [Note] Plugin 'FEEDBACK' is disabled.
2023-08-02  7:45:07 0 [Note] Server socket created on IP: '172.29.238.71'.
2023-08-02  7:45:07 0 [Note] Server socket created on IP: '172.29.238.71'.
2023-08-02  7:45:07 0 [Note] WSREP: Recovered position: 1e3da54e-3051-11ee-bbb6-cb65a0f86cdc:20639
Warning: Memory not freed: 280
compute01_galera_container-82ba3a78 | CHANGED | rc=0 >>
2023-08-02  7:45:07 0 [Note] mysqld (server 10.6.10-MariaDB-1:10.6.10+maria~ubu2004-log) starting as process 65416 ...
2023-08-02  7:45:07 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2023-08-02  7:45:07 0 [Note] InnoDB: Using transactional memory
2023-08-02  7:45:07 0 [Note] InnoDB: Number of pools: 1
2023-08-02  7:45:07 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2023-08-02  7:45:07 0 [Note] InnoDB: Using Linux native AIO
2023-08-02  7:45:07 0 [Note] InnoDB: Initializing buffer pool, total size = 4294967296, chunk size = 134217728
2023-08-02  7:45:07 0 [Note] InnoDB: Completed initialization of buffer pool
2023-08-02  7:45:07 0 [Note] InnoDB: 128 rollback segments are active.
2023-08-02  7:45:07 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2023-08-02  7:45:07 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2023-08-02  7:45:07 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2023-08-02  7:45:07 0 [Note] InnoDB: 10.6.10 started; log sequence number 16732851; transaction id 48153
2023-08-02  7:45:07 0 [Warning] InnoDB: Skipping buffer pool dump/restore during wsrep recovery.
2023-08-02  7:45:07 0 [Note] Plugin 'FEEDBACK' is disabled.
2023-08-02  7:45:07 0 [Note] Server socket created on IP: '172.29.237.95'.
2023-08-02  7:45:07 0 [Note] Server socket created on IP: '172.29.237.95'.
2023-08-02  7:45:07 0 [Note] WSREP: Recovered position: 1e3da54e-3051-11ee-bbb6-cb65a0f86cdc:20639
Warning: Memory not freed: 280
```
- From the above output we can see the WSREP: Recovered position: for each member node, and we will figure out which node has the highest number we issue `ansible galera_container -m shell -a "sed -i "/safe_to_bootstrap/s/0/1/" /var/lib/mysql/grastate.dat" --limit galera_container[0]` command on that member node.
- Once discoring which node had the latest data and change it's safe_bootstrap_value then performing the following `ansible galera_container -m shell -a "galera_new_cluster" --limit galera_container[2]` on that node will bootstrap the cluster
- When performing systemctl status mariadb, the serivce might be stuck on "deactivating (stop-sigterm)" in this case we need to find the pid of mariadb from top command and kill that service using "kill -9 460" the check if the service is stopped using systemctl status mariadb, once the service is in a failed stopped state we can once again start the service using systemctl start mysql.
- Sometimes in the event of none gracefull shutdown after starting the initial cluster and tring to join the rest of the nodes using systemctl start mariadb might fail with a journalctl -u mariadb error of
```
Aug 03 12:05:23 compute01-galera-container-82ba3a78 mariadbd[174335]: 2023-08-03 12:05:23 0 [ERROR] WSREP: Corrupt buffer header: addr: 0x7fc8ebfff6e0, seqno: 3543537071533338624, size: 808660325, ctx: 0x55cfde33f968, flags: 12597. store: 45, type: 49
Aug 03 12:05:23 compute01-galera-container-82ba3a78 mariadbd[174335]: 230803 12:05:23 [ERROR] mysqld got signal 6 ;
```
- This case indicate a gcache corrput and we need to clear and force a fresh SST sync using the following command on the node with the problem `truncate /var/lib/mysql/galera.cache -s 0`
- So the procedure might look like: from the deployment node to the node with the issue
```
ansible galera_container -m shell -a "systemctl stop mysql" --limit galera_container[1]
ansible galera_container -m shell -a "truncate /var/lib/mysql/galera.cache -s 0" --limit galera_container[1]
ansible galera_container -m shell -a "systemctl start mysql" --limit galera_container[1]
```
- If you get an error when restarting mariadb with the following error message
```Unknown/unsupported storage engine: InnoDB
Aug 09 15:34:35 infra01-galera-container-08ddf08d sh[326]: 2023-08-09 15:34:35 0 [Note] InnoDB: Starting shutdown...
Aug 09 15:34:35 infra01-galera-container-08ddf08d sh[326]: 2023-08-09 15:34:35 0 [ERROR] Plugin 'InnoDB' init function returned error.
Aug 09 15:34:35 infra01-galera-container-08ddf08d sh[326]: 2023-08-09 15:34:35 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed
.
Aug 09 15:34:35 infra01-galera-container-08ddf08d sh[326]: 2023-08-09 15:34:35 0 [Note] Plugin 'FEEDBACK' is disabled.
Aug 09 15:34:35 infra01-galera-container-08ddf08d sh[326]: 2023-08-09 15:34:35 0 [ERROR] Unknown/unsupported storage engine: InnoDB
Aug 09 15:34:35 infra01-galera-container-08ddf08d sh[326]: 2023-08-09 15:34:35 0 [ERROR] Aborting'
Aug 09 15:34:35 infra01-galera-container-08ddf08d systemd[1]: mariadb.service: Control process exited, code=exited, stat
us=1/FAILURE
Aug 09 15:34:35 infra01-galera-container-08ddf08d systemd[1]: mariadb.service: Failed with result 'exit-code
'.
Aug 09 15:34:35 infra01-galera-container-08ddf08d systemd[1]: Failed to start MariaDB 10.6.10 database server.
```
- Remove the following files inside the galera contianer
```
rm /var/lib/mysql/ib_logfile0
rm /var/lib/mysql/ib_logfile1
```
- Then restart the service `systemctl restart mariadb``

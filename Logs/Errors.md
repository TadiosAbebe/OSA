---
Errors:
1. Unable to login into openstack horizon dashboard. "Something went wrong" error afer putting in username and password
2. When verifing openstack services after deployment, issuing "openstack network agent list" on inside the utility contianer on the infrastructure nodes, it results in 503 error
	- HttpException: 503: Server Error for url: http://10.10.36.9:9696/v2.0/agents, 503 Service Unavailable: No server is available to handle this request.
3. When investigating the neutron service, by navigating to the neutron-server container on the infrastructure nodes ans issuing systemctl status neturon-server, the service is flactuating by comming up and down

Solution:
The reason neutron-server is not starting was due to incompatable coniguration set in place on the openstack user conifg file and else where, on the zed release OSA switched from linux bridges to ovn and from vxlan to geneve.
- changed network type from vxlan to geneve
- changed group_binds from neutron_linuxbridge_agent to neutron_ovn_controller
- added additional configurations on user_variables.yml
- created files and configurations in /etc/openstack_deploy/env.d/neutron.yml, /etc/openstack_deploy/env.d/nova.yml, /etc/openstack_deploy/group_vars/network_hosts

ref: https://bugs.launchpad.net/openstack-ansible/+bug/2002897
---
Error:
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

Solution:
Inside the galera containers of the infrastructure nodes
- naviage to /var/lib/mysql/grastate.dat file
- change the value of safe_to_bootstrap to 1
- sed -i "/safe_to_bootstrap/s/0/1/" /var/lib/mysql/grastate.dat
- now restart the mariadb service or running galera_new_cluster seems to fix the problem

ref: https://stackoverflow.com/questions/37212127/mariadb-gcomm-backend-connection-failed-110

---
Error:
openstack-ansible setup-openstack.yml script keeps failing on setting up gnocchi on "python_venv_build : Build wheels for the packages to be installed into the venv" Task to be specific. I was trying to deploy the Zed release of OpenStack, but the issue still persists when trying to deploy the Antelope release of OpenStack. When I look at the python log file generated from the venv build, I see a bunch of "# Package would be ignored # warning" about 18 of them; some of those are for the packages gnoochi.cli, gnoochi.incoming, gnoochi.indexer, gnoochi.rest, gnoochi.storage, gnoochi.test, and so on... and it says "AttributeError: module 'setuptools.command.easy_install' has no attribute 'get_script_header'"


Solution:
I need to apply both of the aforementioned patches for it to work for me.
- adding "gnocchi_git_install_branch: 6f35ea5413a9f78551d8193b8d2a6d77c49b6372" to /etc/openstack_deploy/user_variables.yml
- adding "setuptools==67.8.0" to /opt/openstack-ansible/global-requirement-pins.txt

Errors:
1. Unable to login into openstack horizon dashboard. "Something went wrong" error afer putting in username and password
2. When verifing openstack services after deployment, issuing "openstack network agent list" on inside the utility contianer on the infrastructure nodes, it results in 503 error
	- HttpException: 503: Server Error for url: http://10.10.36.9:9696/v2.0/agents, 503 Service Unavailable: No server is available to handle this request.
3. When investigating the neutron service, by navigating to the neutron-server container on the infrastructure nodes ans issuing systemctl status neturon-server, the service is flactuating by comming up and down
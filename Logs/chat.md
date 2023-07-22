<Tad> mgariepy Here is my neutron and connectivity issue, instances can communicate with each other on their private addresses, i.e. 192.168.10.144 is able to ping 192.168.10.160 and vice versa, but they are not able to go out to the internet, i.e. ping 8.8.8.8
<Tad> My configuration files are located at https://github.com/TadiosAbebe/OSA/blob/master/etc/openstack_deploy and  I have modified the openstack_user_config.yml, user_variables.yml, ./group_vars/network_hosts, /env.d/neutron.yml and /env.d/nova.yml files.
<Tad> Here is a detail description of what i tried and how my enviroment is setup https://pastebin.com/W9xy0EWC
<noonedeadpunk> Tad: first thing I can tell - `is_container_address` should be defined only once. And that supposed to be br-mgmt
<Tad> okay great i'll fix that.
<noonedeadpunk> one small nit - br-vxlan is super confusing given that this is geneve - 2 different overlay protocols. 
<noonedeadpunk> That all is unrelated (likely) to your issue though
<noonedeadpunk> Also - you have all bridges except br-storage as linux bridges, and br-storage as ovs one?
<Tad> how so? do i need to specifiy openvswitch on my netplan for all bridges?
<noonedeadpunk> then, log_hosts is likely will not have any effect - we've dropped rsyslog roles as logs are managed with journald
<noonedeadpunk> Tad: well, I don't know, I'm asking you :D You have `container_bridge_type: "openvswitch"` only for 1 bridge https://github.com/TadiosAbebe/OSA/blob/master/etc/openstack_deploy/openstack_user_config.yml#L51C9-L51C45
<Tad> nice, i'll remove the log_hosts too
<noonedeadpunk> so was kinda wondering why not to align that to same tech :) But that's not critical as well
<noonedeadpunk> What is critical, is that I don't see some required definitions of groups for OVN
<Tad> like what definitions?
<noonedeadpunk> Tad: I think you're missing `network-gateway_hosts` and `network-northd_hosts`
<noonedeadpunk> https://docs.openstack.org/openstack-ansible-os_neutron/latest/app-ovn.html#deployment-scenarios
<noonedeadpunk> and ovn gateway is exactly the thing that is repsonsible for external connectivity
<Tad> yes, i have seen that from the docs but i supposed it will create them automatically. and i can see OVN Controller Gateway agent on my compute node when issuing openstack network agent list
<noonedeadpunk> I think that controller gateway is implicitly included in network-hosts
<noonedeadpunk> but ovn gateway is totally not
<noonedeadpunk> as there're multiple scenarios where to place them, and usually that's not control plane
<noonedeadpunk> either compute nodes or standalone network nodes
<Tad> Ohh good to know, I’ll specify network-gateway_hosts on my compute node, what else do you see that is off right away?
<noonedeadpunk> actually, that's what made northd available https://github.com/TadiosAbebe/OSA/blob/master/etc/openstack_deploy/env.d/neutron.yml
<noonedeadpunk> that actually looks like beingf taken from Yoga, as it should not be needed since zed
<Tad> i did that from the following suggestion https://bugs.launchpad.net/openstack-ansible/+bug/2002897 so should i remove the neutron.yml config and just place my network-gateway_hosts on openstack_user_config.yml?
* spotz has quit (Ping timeout: 480 seconds)
<noonedeadpunk> Well, I'd drop both that and nova.yml as well
<noonedeadpunk> and then defined network-gateway_hosts and network-northd_hosts
<Tad> great, what else?
<Tad> this "Also - you have all bridges except br-storage as linux bridges, and br-storage as ovs one?" is a good point out but i don't know what i should do? 
<noonedeadpunk> Well, I would either have all bridges on controllers as OVS or all as just linux bridges, not mix of them. But there is technically nothing wrong in mixing them if there is a reason for that
<noonedeadpunk> in your case it should be likely easier to drop `container_bridge_type: "openvswitch"` and re-configure br-storage as simple bridge
<noonedeadpunk> also - once you will change env.d/conf.d/openstack_user_config you will likely need to re-run lxc-containers-create.yml playbook
<Tad> i don't have any reason for mixing that. "We opted to move over to the new OVN provider. This solved our issues and left the deprecated LinuxBrdige driver outside of the equation. Also, VXLAN was replaced with Geneve. Relevant configuration files were adjusted as follows:" is taken from https://bugs.launchpad.net/openstack-ansible/+bug/2002897 that is why i opted for ovs
<noonedeadpunk> Tad: well, this is in the context of neutron driver. Linux bridges are indeed deprecated as a neutron drivers. But they are still part of the Linux :D
<noonedeadpunk> and OSA is quite agnostic of tech - you can even passthrough physical interfaces inside LXC containers and not having bridges at all
<noonedeadpunk> so it's kinda matter of prefference and taste
<Tad> ohh okay, then i'll drop the container_bridge_type: "openvswitch"
<Tad> what about the host_bind_override: "bond1" is this necessary?
<noonedeadpunk> Tad: to be frank - I don't remember :D But idea behind that, is that there might be no bridge as br-vlan - as it's absolutely fine to have just interface instead of the bridge on network/compute hosts
<noonedeadpunk> and it is not needed on storage/controller hosts at all
<noonedeadpunk> so to avoid creating br-vlan bridge, you can just have an interface and mark that with `host_bind_override`
<noonedeadpunk> I can't recall if you can simply use interface in `container_bridge` or not...
<opendevreview> Merged openstack/openstack-ansible-galera_server master: Do not use notify inside handlers  https://review.opendev.org/c/openstack/openstack-ansible-galera_server/+/887520
<opendevreview> Dmitriy Rabotyagov proposed openstack/openstack-ansible-haproxy_server master: Do not use notify inside handlers  https://review.opendev.org/c/openstack/openstack-ansible-haproxy_server/+/888762
<Tad> oh okay.
<noonedeadpunk> but if you have br-vlan bridge on compute/network hosts - you don't need host_bind_override then
<noonedeadpunk> it's just a bit weird, as basically what will happen with that bridge - it will be added as "interface" to another bridge, while br-vlan would have only 1 interface in it. It was named as "bridge" only for consistency and naming things in the same way across all docs
<noonedeadpunk> same with br-vxlan actually
<noonedeadpunk> but it would work both ways :)
<opendevreview> Dmitriy Rabotyagov proposed openstack/openstack-ansible-plugins master: Do not use notify inside handlers  https://review.opendev.org/c/openstack/openstack-ansible-plugins/+/888766
* spotz (~spotz@2603-8081-0400-6800-e1e9-c89c-835d-64f4.res6.spectrum.com) has joined
<Tad> oh okay. When specifying network-northd_hosts network-gateway_hosts in openstack_user_config do i need to remove the network_hosts: portion or should i keep it?
<noonedeadpunk> no, keep that
<noonedeadpunk> it is needed for neutron-server
<Tad> oh okay, I'm now running the playbooks after changing what you suggested
<Tad> and when running the playbook what i have been doing so far is running all seutp-host, infrastructure and openstack after changing any configuration, is that the proper way.
<noonedeadpunk> well. it's long way :)
<noonedeadpunk> it would work though
<noonedeadpunk> (with that pace you could run setup-everything.yml as well)
<noonedeadpunk> short path would be to run lxc-containers-create and then affected roles. For example, if you're changing neutron configuration - run os-neutron-install.yml afterwards
<noonedeadpunk> and lxc-containers-create is needed only when you expect changes in inventory, that would result in creating new containers
<Tad> ya it takes me about more that 4 hours, to deploy openstack on 3 nodes every time i make a change :) i'll try the lxc-containers-create route next time
<noonedeadpunk> in current case, as you've dropped some env.d I would likely also re-create neutron-server containers, as some ovn services ended up there, while it shouldn't
<noonedeadpunk> wow, that's too long kinda...
<Tad> ya image doing it when you have a lot to learn and experiment
<noonedeadpunk> I wouldn't expect setup-everything to run more then 2h to be frank, but it's still really long
<Tad> i think it took that long for me because it is installing the control plain on all three hosts
<noonedeadpunk> and what you can do - run `openstack-ansible lxc-containers-destroy.yml --limit neutron_server`
<noonedeadpunk> and re-create these with `openstack-ansible lxc-containers-create.yml --limit neutron_server,localhost`
<Tad> okay great, that would be handy, for now since i have changed my netplan and reapplied it i am running setup-everything to be on the safe side
* damnthem has quit (Remote host closed the connection)
* damnthem (~Thunderbi@2a01:a282:13:14:6820:87be:ce0a:2bdd) has joined
* jlabarre-rh (~jlabarre@2601:184:c300:1b83::3dd) has joined
<Tad> noonedeadpunk: on another note when experimenting with OSA with three nodes and after successful deployment, if there is a power interruption and all my 3 servers losses power, galera cluster won’t start up and I have to go and manually do galera_new_cluster on the node where safe_to_bootstrap is 1 inside the /var/lib/mysql/grastate.dat am I doing something wrong or is there a more permanent solution.
<anskiy> Tad: yeah, it did. I would suggest revisit your openstack_user_config, as, for example this bit: https://github.com/TadiosAbebe/OSA/blob/master/etc/openstack_deploy/openstack_user_config.yml#L68-L74 -- you're installing infrastructure services for control plane (eg galera cluster) on you compute and storage nodes.
<noonedeadpunk> I think it's intentional POC deployment ;)
<anskiy> could be it, but that's an oportunity to speed thing up a little bit :)
<noonedeadpunk> Tad: nah, I guess it's "relatively" fair recovery process of galera. Same actually happens in split-brain, when it did not record last sequence number (due to unexpected shutdown) - it doesn't know where latest data is
<noonedeadpunk> so yes, you need to tell it which one should act as "master"
<noonedeadpunk> Tad: well ,actually, I dunno if you knew that or not, but you can create multiple containers of same type on a single host: https://docs.openstack.org/openstack-ansible/latest/reference/inventory/configure-inventory.html#deploying-0-or-more-than-one-of-component-type-per-host
<noonedeadpunk> so you can have 3 galera containers on the 1 host to play with clustering, for instance
<Tad> oh okay, I guess you wouldn't encounter power interruption in production environment. The thing is I am experimenting with openstack at my office where all 3 baremetal servers are located here and no UPS. and we often encounter power interruption
<noonedeadpunk> Well, power loss of all DC can happen, ofc, but mysql would be your least problem in case of this happening
<Tad> anskiy as noonedeadpunk pointed out it is a POC thing, but I think I could also leave out the HA thing until i get neutron to work properly as you said it might speed things up a little
<Tad> noonedeadpunk what else should be i concerned with on power interruption
<noonedeadpunk> storage?:)
<Tad> what could go wrong with storage, i'am using cinder with lvm backend and i'm not doing anything serious with the openstack cloud
<noonedeadpunk> well, instances that are runnning using buffers, so they can easily get broken FS inside them
<Tad> oh you are right, i should really get a UPS for the future but currently i am at the stage where i'm just running cirros image
* dviroel (~dviroel@2804:431:cfcd:b68:4c34:524a:cefb:4265) has joined
<Tad> Having this openstack_user_config now https://pastebin.com/XLduZ9uK setup-openstack fails on TASK [os_neutron : Setup Network Provider Bridges] with the error "ovs-vsctl: cannot create a bridge named br-vlan because a port named br-vlan already exists on bridge br-provider"
* spotz has quit (Ping timeout: 480 seconds)
<noonedeadpunk> Um... honestly I'm not sure here. And you have pre-created the br-vlan with netplan config?
<Tad> yes i have br-vlan on bond1 of my netplan
<NeilHanlon> noonedeadpunk: how are rocky jobs looking? i tagged new ovn yesterday
<noonedeadpunk> I did some rechecks today but haven't checked their status yet. but my aio was good :)
<noonedeadpunk> And CI seems way greener today
<mgariepy> hey good morning.
<NeilHanlon> good to hear :) 
<NeilHanlon> i'll follow up with amorlej to find out what I did wrong lol
<NeilHanlon> morning mgariepy
<mgariepy> NeilHanlon, how did your moving all your stuff went ?
<NeilHanlon> went pretty well, all things considered! starting to have some semblance of normalcy now 
<NeilHanlon> movers cost almost 2x what they estimated us... but
<mgariepy> haha dust takes some time to settle :)
<mgariepy> wow.
<NeilHanlon> oh the dust is another thing altogether lol... my asthma hates me
<NeilHanlon> I've always moved in the fall/spring, so I didn't account for that it would take them on the longer end of their time estimate... since it was so hot
<NeilHanlon> plus they charged me $160 for moving my sofa lol
<mgariepy> 160 only for 1 sofa ? 
<mgariepy> must be a huge sofa ;)
<NeilHanlon> supposedly because it was heavy (it's a powered recliner)
<NeilHanlon> another $100 for a fridge they moved downstairs...
<NeilHanlon> i get the feeling they knew it was the last time they were gonna move me and wanted to get their last $$$
<mgariepy> how much does it cost, well it depends, how much do you have, let me get you a personalized quote ? 
<mgariepy> well pretty much like anything else i guess.
<NeilHanlon> they had been good to me in the past, which is why I used them. it was like my 5th move with this same group
<NeilHanlon> but yeah, it felt a bit like a bait and switch
<mgariepy> they offer good service/price for moving between appartement but do charge way more for a house ?
<mgariepy> haha
<Tad> noonedeadpunk: when i put back host_bind_override: "bond1" on container_bridge: "br-vlan" os-neutron-install.yml completed without error
* spotz (~spotz@68.201.196.188) has joined
<mgariepy> can i have some review on : https://review.opendev.org/c/openstack/openstack-ansible-haproxy_server/+/888314 and https://review.opendev.org/c/openstack/openstack-ansible-haproxy_server/+/888498
* NeilHanlon nods
* spotz has quit (Ping timeout: 480 seconds)
* ncuxo (~lessfooba@dynamic-046-114-003-155.46.114.pool.telefonica.de) has joined
* spotz (~spotz@2603-8081-0400-6800-e1e9-c89c-835d-64f4.res6.spectrum.com) has joined
* TadiosAbebe (~TadiosAbe@196.190.62.132) has joined
* spotz has quit (Ping timeout: 480 seconds)
* ncuxo has quit (Remote host closed the connection)
<Tad> mgariepy: when you got time, here is the problem I was referring yesterday: https://pastebin.com/W9xy0EWC and here is my latest openstack_user_config.yml fine after making changes suggested by noonedeadpunk https://pastebin.com/XLduZ9uK but the issue still persist any help would be appreciated
* TadiosAbebe has quit (Ping timeout: 480 seconds)
<jamesdenton> does the haproxy role provide the ability for each endpoint to have a unique fqdn using port 443?
<jamesdenton> @Tad did you create a neutron router and connect it to the external network as well as the tenant network? can you ping the default gateway from the VM?
<jamesdenton> (sorry, looks like i missed that on line 8)
<jamesdenton> If your public network truly is 10.20.30.0/24, do you have an upstream NAT? that network is not routable
<mgariepy> jamesdenton, haproxy does supports SNI
<noonedeadpunk> jamesdenton: you can do that with haproxy maps since Antelope relatively easily
<jamesdenton>  but 2023.1 is the key there, huh?
<Tad> jamesdenton even though instances can ping each other they are not able to ping their gateway.
<Tad> and i don't have any nat on that network it is a vlan 102 network created on the router
<noonedeadpunk> I've heard folks doing that even before, but with maps it should become relatively easily
<jamesdenton> Tad if you expect your tenant network to reach the internet, then the external network needs to either be routable or have some NAT upstream. The provider network itself needs a default gateway that can reach the internet
<jamesdenton> thanks noonedeadpunk mgariepy 
<jamesdenton> Tad did you attach the router to the tenant network? openstack router add subnet <router> <tenant subnet>
<noonedeadpunk> Tad: so from VM you can't reach router gateway, right?
<noonedeadpunk> openstack router I mean
<Tad> jamesdenton yes i, but the vms are'nt able to ping either their gateway or the interface on the other end which is my provider network. 
<Tad> noonedeadpunk yes they cant reach the gateway
<jamesdenton> sure, so the first problem, then is getting them to ping the tenant gateway
<jamesdenton> there's only 1 compute?
<Tad> yes
<jamesdenton> can you do me a favor and provide the output of "ovs-vsctl list open_vswitch" from all 3 nodes?
* TadiosAbebe (~TadiosAbe@196.190.62.121) has joined
<Tad> jamesdenton here you go https://pastebin.com/Hz3UP6u2
<jamesdenton> thanks
<jamesdenton> can you also please provide: openstack network show and openstack subnet show for the 2 networks?
<Tad> jamesdenton here you go https://pastebin.com/70LJggp5
<jamesdenton> and just to confirm, your VM IPs are really 10.0.0.x not 192.169.10.x right?
<Tad> yes they where on 192.168.10.0 but they are on 10.0.0.0 network now
<jamesdenton> DHCP seems to be working?
<Tad> yes it is
* spotz (~spotz@2603-8081-0400-6800-e1e9-c89c-835d-64f4.res6.spectrum.com) has joined
<Tad> jamesdenton can you see any problem on my openstack_user_config file here https://pastebin.com/XLduZ9uK or is it about right
<jamesdenton> Not sure if you intended on spreading out services across all three nodes or not
<jamesdenton> at a glance, the OVN bits seem OK
<jamesdenton> on the compute, can you show me the 'ovs-vsctl show' output?
<Tad> yes i wanted to test a hyperconverged control plane
<Tad> here you go https://pastebin.com/WCzpS4bA
<Tad> but when Having this openstack_user_config https://pastebin.com/XLduZ9uK setup-openstack fails on TASK [os_neutron : Setup Network Provider Bridges] with the error "ovs-vsctl: cannot create a bridge named br-vlan because a port named br-vlan already exists on bridge br-provider" but when i add host_bind_override: "bond1" on container_bridge: "br-vlan" os-neutron-install.yml completed without error
<jamesdenton> ok, so i lied. I see ovn-bridge-mappings="vlan:bond1", which implies bond1 is a bridge, and that vsctl show output confirms that. I suspect you mean for bond1 to be the interface used, and that should be connected to a bridge (likely br-provider)
<jamesdenton> the openstack_user_config.yml shows br-vlan, though, so maybe br-provider was rolled by hand later?
* osn (~osn@ec2-18-188-233-89.us-east-2.compute.amazonaws.com) has joined
<anskiy> there is bridge called br-provider, which contanins port br-vlan (like in the error you mentioned before), and in your netplan config, br-vlan is the linux bridge with bond1. At the same time, bond1 is a port in OVS bridge bond1...
<anskiy> You might need to delete OVS bridge br-provider, maybe?.. As I don't really see, where it's been used
<jamesdenton> Mine looks like this: https://paste.opendev.org/show/bLkYnCApAH4vXAULykQk/. Playbooks would create br-provider (if it doesn't exist) and connect bond1 to br-provider (bond1 must be an existing interface)
<Tad> so there is a bunch of "failed to add bond1 as port: File exists" inside ovs-vswitchd.log and i think this is happing because i added back the host_bind_override: "bond1" on my config. but without that the playbook fails
<jamesdenton> host_bind_override should only be for linuxbridge, IIRC
<jamesdenton> try using "network_interface: bond1" instead. In the meantime, you should be able to delete br-provider bridge. br-vlan is probably also unnecessary
<jamesdenton> also, if you have the playbook error that would be helpful
<Tad> how did the the br-provider get created in the first place though?
<jamesdenton> ¯\_(ツ)_/¯ 
<Tad> and the thing about deleting br-provider of br-vlan is i dont want to go and manually remove these things because i want this to be a repeatable process. so when i move to testing deployment on different machine i want to be able to run the playbooks and make them work. any way i could control this from openstack ansible configs?
<jamesdenton> Well, the environment is in an incompatible state at the moment. The playbooks don't delete anything network related, only create/modify - but ideally you would setup openstack_user_config.yml and the correct bits would be done the first time
<jamesdenton> for OVS/OVN, you want an OVS provider bridge connected directly to a physical interface. The snippet i sent would result in the playbooks creating br-provider and connecting bond1.
<Tad> ohh great, it might me something that was created before so, let me clean install the enviroment run the script with the my latest openstack_user_config and let me get back to you then.
<jamesdenton> wanna ship that config over first so we can glance at it?
<Tad> sure, https://pastebin.com/XLduZ9uK
<jamesdenton> and your netplan?
<Tad> let me collect them in one file, give me a min
<Tad> netplan: https://pastebin.com/6erRikBP  user_variable https://pastebin.com/7qxF9rBG
<Tad> jamesdenton: do i need to use openvswitch bridges on my host machines?
<jamesdenton> no
<Tad> so the above configs seems okay?
<jamesdenton> ok, so my suggestion would be to remove br-vlan from netplan
<jamesdenton> then, in openstack_user_config, in the br-vlan network block, add 'network_interface: "bond1"' under 'container_bridge: "br-vlan"'
<jamesdenton> br-vlan will end up being created as an OVS bridge
<jamesdenton> everything else looks pretty sane at a glance
<jamesdenton> which version of OSA?
<Tad> zed
<Tad> 26.1.2
<jamesdenton> k
<jamesdenton> cool, give that a go and let us know. should be around for a bit
<Tad> great, i'll do that and get back to you later. i have learned a lot today. jamesdenton anskiy noonedeadpunk thank you very much for your time.
<jamesdenton> and you going to wipe and redeploy?
<Tad> what i usually do is i have a timeshift snapshot before i run anything on the machines, so i restore to that point and go on from there
<noonedeadpunk> #startmeeting openstack_ansible_meeting
<opendevmeet> Meeting started Tue Jul 18 15:02:18 2023 UTC and is due to finish in 60 minutes.  The chair is noonedeadpunk. Information about MeetBot at http://wiki.debian.org/MeetBot.
<opendevmeet> Useful Commands: #action #agreed #help #info #idea #link #topic #startvote.
<opendevmeet> The meeting name has been set to 'openstack_ansible_meeting'
<Tadios> noonedeadpunk jamesdenton anskiy, After yesterday's suggestions and recommendations, I was able to fix my networking issues. and thank you very much. Now my instances are able to communicate with the outside, but I have a few more questions.
<Tadios> 1. I still can't create an admin network from horizon dashboard Admin > Network > Networks > Create Network provides "Danger: An error occured. Please try again later." and it is returning 500 Internal Server Error. from devtools. I dont know where to look the log files for this error? I tried /var/apache2/error.log but notting.
<admin1> Tadios, try from the cli ? 
<Tadios> admin1: i tried from cli and it works fine.
<noonedeadpunk> Tadios: it should be in journald for apache2 unit
<Tadios> okay let me check
* lowercase (~lowercase@0002c12c.user.oftc.net) has joined
<noonedeadpunk> like `journalctl -f -u apache2` 
<Tadios> i should try this from inside horizon container right?
<noonedeadpunk> yup. you can do outside as well, but you'd need to select correct path with db
<noonedeadpunk> inside /var/log/journal
* spotz has quit (Ping timeout: 480 seconds)
<Tadios> it says "Undefined provider network types are found: ['v', 'l', 'a', 'n', ',', 'l', 'o', 'c', 'a', 'l', ',', 'g', 'e', 'n', 'e', 'v', 'e']
<Tadios> " when trying to create a network
<Tadios> https://paste.opendev.org/show/820717/
<noonedeadpunk> ah
<Tadios> does it have something to do with this specification in my user_variables.yml "neutron_ml2_drivers_type: "vlan,local,geneve""
<noonedeadpunk> Tadios: that looks like quite valid bug actually
<Tadios> oh really? why is it happening though?
<opendevreview> Dmitriy Rabotyagov proposed openstack/openstack-ansible-os_horizon master: Fix wrong neutron_ml2_drivers_type  https://review.opendev.org/c/openstack/openstack-ansible-os_horizon/+/888917
<noonedeadpunk> Tadios: this should fix it ^
<noonedeadpunk> but yes, given that you've defined neutron_ml2_drivers_type to their defaults, removing this variable and re-running os-horizon-install.yml should fix the issue
<Tadios> amazing! let me do that.
<noonedeadpunk> (you should be able to add --tags horizon-config to reduce time of running the playbook)
<Tadios> oh really, that would be handy. great now my horizon problem is also solved, i can create admin networks from the web interface.
<Tadios> and my second question was my vms can't still ping the gateway and i don't know what fixed the issue but it is working now, could it be the horizon fix or something else?
* spotz (~spotz@68.201.196.188) has joined
<noonedeadpunk> nah, it's not related to horizon for sure
<Tadios> oh my bad, it was Security Groups. and last question
<Tadios> Do we need to specify the container_type: "veth" in the provider_network: section of the openstack_user_config or is it optional? It is listed as required in the documentation. Also, what about container_interface? I asked this because I don't see these options on the configuration jamesdenton shared https://paste.opendev.org/show/bLkYnCApAH4vXAULykQk/
<noonedeadpunk> I'm not 100% sure, but I'd say yes
<noonedeadpunk> to be frank - I've never experimented enough with that
<noonedeadpunk> as "it works"™
* spatel (~spatel@static-98-110-253-55.bstnma.fios.verizon.net) has joined
<noonedeadpunk> Except, used `container_type: phys` to pass interface inside container
<Tadios> noonedeadpunk: okay great, thank you for your time, as always.
<noonedeadpunk> https://docs.openstack.org/openstack-ansible/latest/reference/configuration/extra-networks.html#using-sr-iov-interfaces-in-containers
<NeilHanlon> would guess veth is required for plumbing the pseudowires
<noonedeadpunk> I think that `container_type` would be passed to lxc config and then it's up to lxc defaults 
<Tadios> NeilHanlon: okay good to know.
<noonedeadpunk> `container_interface` is needed only for groups that have containers in fact. Like it is needed for neutron-server, but it is not for nova-compute, for instance
<Tadios> oh okay.
<NeilHanlon> you can check the lcx docs for what available options are 
<NeilHanlon> I've only ever used veth, phys, and Macvlan , but others are supported 
* Sage905 has quit (Ping timeout: 480 seconds)
<noonedeadpunk> ah, yes, macvlan was used as an option for octavia for some users as well
<noonedeadpunk> hamburgler: I can't recall if that was you who asked about 27.0.1 or not, but - it just went live
<opendevreview> Dmitriy Rabotyagov proposed openstack/openstack-ansible-os_adjutant master: Fix linters and metadata  https://review.opendev.org/c/openstack/openstack-ansible-os_adjutant/+/888469
<Tadios> so let's say i have three nodes node1,2 and 3 if use all three of the nodes to run my control plane services and my compute service, is it a going to be an okay high available system. or there is a problem with this design?
<admin1> depends
<admin1> test or production ? 
<NeilHanlon> admin1++
<Tadios> let's say company internal production
<admin1> if production , then no .. 
<NeilHanlon> Tadios: at the end of the day you're going to need to deal with resource contention between the things running ON your cloud and the things RUNNING your cloud. One has to have a higher priority, at the end of the day.. and hyper-convergence of everything as you describe is... an advanced topic
<admin1> one acceptable way is to only run ssh in your server, then create the necessary bridges for traffic and then create 2x virtual machines, one for controller and one for hypervisor 
<admin1> then you can use them 
<mgariepy> anyone see weird network traffic, like public network leaking to mgmt network ?
<admin1> mgariepy, it depends on the vlan and routing 
<mgariepy> i see arp req/res for api ip (which is on vlan XXX) passing on mgmt vlan (whcih is vlan YYY) no routing between, and the traffic is on the same L2, if i force ping -I vlanYYY api_ip
<mgariepy> only for IP/vlan that are on the controller
<mgariepy> admin1, not l3 in this case.
<NeilHanlon> my guess is that you're leaking routes in the default table between those two vlans on the controller, mgariepy
<NeilHanlon> fib routes, i mean
<mgariepy> is it leaked via something like rp_filter = 0 ?
<NeilHanlon> how are vlanX/vlanY setup? single interface w/ 802.1q on top?
<NeilHanlon> basically it seems like vlanX and vlanY share a bridge, and your host is flooding traffic between them, acting like a router and proxying arp requests
<mgariepy> the 2 are on ther controllers. bond > vlanX and bond > vlany > bridge
<NeilHanlon> oh, hm
<mgariepy> ip in question are on vlanX and bridge for the other network.
<mgariepy> leak is vlanx ip passing for some reason on vlanY. 
<mgariepy> great for performance.. but.. meh haha
<NeilHanlon> what does `bridge vlan show` show, out of curiosity?
<mgariepy> 1 PVID Egress Untagged
* spatel has quit (Quit: My MacBook has gone to sleep. ZZZzzz…)
<mgariepy> all of them
<hamburgler> noonedeadpunk: yes was me :) thanks very much!
<NeilHanlon> off topic: https://youtu.be/uq6BJCakbtA
<NeilHanlon> mgariepy: let me poke around in my lab and see what we can do
<NeilHanlon> s/we/I/
<mgariepy> great thanks :D
* ncuxo (~lessfooba@dynamic-046-114-000-029.46.114.pool.telefonica.de) has joined
* spatel (~spatel@c-73-89-243-254.hsd1.ma.comcast.net) has joined
<mgariepy> i think it's because of the lxc iptables rules.
* ncuxo has quit (Remote host closed the connection)
<Tadios> NeilHanlon admin1 : okay so, here is the case, we have about 4 dell poweredge 730 servers at the office and i'm tasked with deploying a private cloud on them for internal services. and i am confused on which way would it be a good way to setup openstack to utilize the hardware resource and openstack services

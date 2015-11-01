==================
Congress use cases
==================

Background
~~~~~~~~~~

This document contains user stories currently targeted for the OpenStack
Congress policy framework. Originally we used this doc to vote on which use
cases to prioritize.  We’ll leave the votes around for posterity.

Template user story
~~~~~~~~~~~~~~~~~~~

This is a summary of the use case.

Data sources
------------

* A list of data source plugins (OpenStack/other components) needed

Policy
------

* A list of datalog rules that represent the policy

Policy actions needed
---------------------
What type of action does Congress need to take for this use case? Examples:

* Monitoring

* Proactive: Responding to queries from other components about whether a given
  API call will cause new violations.

* Reactive: Computing/executing actions that bring the cloud back into
  compliance

* Assistive: Filling out missing fields of an API call that make the call
  consistent with policy.

* Sub-policy: Pushing policy down to another service

+5 Public/private networks with group membership (implemented)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Votes: +1 thinrichs, pballand, rajdeepd, sarob, banix

There are a plethora of restrictions that cloud operators may want to place on
how VMs are connected to networks.  For example, suppose we want to ensure that
every network a VM is connected to is either a public network or the owner of
the network and the other of the VM belong to the same ActiveDirectory/
Keystone/etc. group.

Data sources
------------

* Neutron: the list of public networks and the owner of each network

* Nova: the list of networks connected to VMs and the owners of those VMs

* ActiveDirectory/Keystone: which users are members of which groups

Policy
------

.. code:: console

   error(vm) :-
      nova:instance(vm),
      nova:network(vm, network),
      not neutron:public(network),
      nova:owner(vm, vmowner),
      neutron:owner(network, netowner),
      not same_group(vmowner, netowner)
   same_group(x,y) :-
      group(x,g),
      group(y,g)
   group(x,g) :-
      keystone:group(x, g)
   group(x,g) :-
      ad:group(x,g)

Policy actions
--------------

* Monitoring  (Could also illustrate proactive/reactive actions).

All VMs connected to the internet must be secured (implemented)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Every port on any VM connected to the internet must be assigned the security
group “lockdown”.

Data sources
------------

* Nova

* Neutron

Policy
------

* Define table that shows all the security group names for each port

.. code:: console

   port_security_group(port_id, sg_name) :-
      neutron:ports(security_groups_id=sg_groups, id=port_id),
      neutron:ports.security_groups(security_groups_id=sg_groups,
         security_group_id=sg_id),
      neutron:security_groups(name=sg_name, id=sg_id)

* Define table that shows which routers are connected to which networks

.. code:: console

   router_network(router_id, network_id) :-
      neutron:routers(id=router_id),
      neutron:ports(network_id=network_id, device_id=router_id)

* Define table representing routers not on the internet

.. code::

   empty_external_gateway_router(router_id) :-
      neutron:routers(external_gateway_info=\"None\", id=router_id)

* Define table representing which device_ids are connected to the internet
  (via which ports)

.. code:: console

   connected_to_internet(device_id, port_id) :-
      neutron:ports(network_id=network_id, id=port_id, device_id=device_id),
      router_network(router_id, network_id),
      not empty_external_gateway_router(router_id)

* Define policy that says every port on every VM connected to the internet
  must be assigned the security group named "lockdown"

.. code:: console

   error(vm) :-
      nova:servers(id=vm),
      connected_to_internet(vm, port),
      not port_security_group(port, \"lockdown\")

Policy actions needed
---------------------

* Monitoring

Identify underutilized servers (implemented)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Find those servers whose CPU utilization over the last 2 months was on average
less than 5%.  Such servers are being underutilized and could potentially be
reclaimed. Send an email to the owners of those VMs.

Data sources
------------

* Nova: for info about compute nodes and their owners

* Ceilometer: for CPU utilization statistics

* Keystone: for grabbing a user’s email

Policy
------

.. code:: console

   error(vm, email) :-
      nova:server_owner(vm, owner),
      two_months_before_today(start, end),
      ceilometer:statistics(vm, start, end, “cpu-util”, cpu),
      cpu < 5,
      keystone:email(owner, email)
   two_months_before_today(start, end) :-
      date:today(end),
      date:minus(end, “2 months”, start)

Policy actions needed
---------------------

What type of action does Congress need to take for this use case? Examples

* Monitoring

* Reactive: Computing/executing actions that bring the cloud back into
  compliance

+4 Upgrade of administrative software
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Votes: +1 sarob, +1 thinrichs, +1 su zhang, + mehdi sheikhalishahi

Every OpenStack release cycle, new stable OpenStack packages will be released
through the various distributions. As these become available, some will want to
stage and deploy into production as soon as possible. This policy would make
the upgrade process reactive to the new packages being available.

Datasources
-----------

* Keystone (?): current software versions

* ??: service that tells us latest released software versions: NVD (national
  vulnerability database) can help us on that, I’ve already written a NVD
  parser retrieving vulnerability and software version maps. Also
  vulnerability scanner can help us identify if there is vulnerable
  applications exist on the VM or not.

* Upgrade service: something that we can tell to upgrade a given piece of
  software.

Policy
------

.. code:: console

  error(software) :-
     current_version(software, current),
     latest_version(software, latest),
     // assuming that current/latest are simple numbers so that
     // we can use kudva’s builtins
     less_than(current, latest)

Policy actions
--------------

* Monitoring

* Reactive

Proposed features (pre-spec/blueprints details):
------------------------------------------------

#. Openstack upgrade policy

* Problem description:

* Proposed change/how to solve:

* Alternatives:

* Data model impact: TBD

* API impact: TBD

* Security impact: TBD

* Implementation plan: TBD

* Dependencies: TBD

+0 Evacuation of tenants for planned outage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Votes:

As part of any planned outage of resources, the users would need to either be
without access or be provided access to other services. In order to meet user
SLA stateful services, the users would either need to moved to new servers or
the servers would need to moved themselves. This use case will not required for
stateless services.

Typical enterprise applications are not stateless and will need to be
considered pets rather than cattle.

Data sources
------------

* Nova: which VM is located on which server, move VM from one server to another

* RDBMS: which servers will be down

Policy
------

.. code:: console

   error(vm) :-
      nova:virtual_machine(vm),
      nova:server(vm, server),
      rdbms:server_to_go_down(server, time_to_go_down),
      current_time(current),
      // an error if it’s an hour from when the server is supposed to go down
      current_time + 1 >= time_to_go_down

Policy actions
--------------

* Monitoring

* Reactive

  * HA of enterprise applications will make this use case viable.

  * Active passive approach

  * Active active approach

Proposed features (pre-spec/blueprints details):
------------------------------------------------

+3 Automatic Evacuation on Host Failure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Votes: +1 sarob, kudva, banix

This use case describes the scenario when a host is down and the vms running
inside have been tagged to have full availability.

The cloud operator will be able to define at project or vm level if this has to
be evacuated in case of a host failure. Once the host fails, this event is
notified and all the vms which are tagged or belong to a tenant which is
tagged, will be automatically evacuated to a different host, if any.

The time that the machines were out of service until they were evacuated will
be considered as part of the QoS metrics and SLA.

Data sources
------------

* Nova: which VM is located on which server, move VM from one server to another

* RDBMS: which VM is high-availability

* Ceilometer?: which hosts are down

Policy
------

.. code:: console

   error(vm) :-
      nova:virtual_machine(vm),
      nova:server(vm, server),
      rdbms:high_available(vm),
      ceilometer:server_down(server)

Policy actions
--------------

* Monitoring

* Reactive

Proposed features (pre-spec/blueprints details):
------------------------------------------------

#. Disaster recovery for stateful virtual machines policy

* Problem description: host is down and the vms running inside have been
  tagged to have full availability

* Proposed change/how to solve: By not using local disk to run the VMs, the VMs
  can be restarted on another compute node. Once heartbeat failure is detected,
  the dead hypervisor node would be taken out of the cluster, and a monitor
  service would schedule the VMs to be restarted.

* Alternatives: aggressively attempt to restart the dead hypervisor node
  through IPMI or PDU hard restart.

* Data model impact: TBD

* API impact: TBD

* Security impact: TBD

* Implementation plan: TBD

* Dependencies: TBD

+1 Noisy neighbor v-v
~~~~~~~~~~~~~~~~~~~~~

Votes: +1 sarob

Similar to planned tenant evacuation, except one container or VM would be moved
and/or isolated. In this case, the VM would get moved to another hypervisor.

Proposed features (pre-spec/blueprints details):
------------------------------------------------

#. Noisy neighbor policy

* Problem description
* Proposed change/how to solve: when using oversubscription, if memory, io, or
  cpu gets over X during Y period, move the VM.

* Alternatives: disable oversubscription

* Data model impact: TBD

* API impact: TBD

* Security impact: TBD

* Implementation plan: TBD

* Dependencies: TBD

+0 Noisy neighbor v-p
~~~~~~~~~~~~~~~~~~~~~

Votes:

Similar to planned tenant evacuation, except one container or VM would be moved
and/or isolated. In this case, the VM would get “transformed” into a baremetal
instance.

Proposed features (pre-spec/blueprints details):
------------------------------------------------

#. Noisy neighbor policy

* Problem description

* Proposed change/how to solve: when using oversubscription, if memory, io, or
  cpu gets over X during Y period, move the VM.

* Alternatives: disable oversubscription

* Data model impact: TBD

* API impact: TBD

* Security impact: TBD

* Implementation plan: TBD

* Dependencies: TBD

+6 Compromised VM
~~~~~~~~~~~~~~~~~

Votes: +1 banix, kudva, skn, pballand, thinrichs, ramki

IDS service notices malicious traffic originating from an insider VM trying to
send packets to hosts inside and outside of the tenant perimeter.  As this is
detected, some reactive response would need to be taken, such as isolating the
offending VM from the rest of the network.  This policy would facilitate one of
the reactive responses to be invoked when a compromise is reported by an IDS
service.

Data sources
------------

* IDS (intrusion detection service VM): IP address of the offending VM

* RepDBMS (reputation DBMS): database of known bad external IP addresses (may
  be replaced by API-based real time checks from many public blacklists)

Policy
------

.. code:: console

   error(vm) :-
      nova:virtual_machine(vm),
      ids:ip_packet(src_ip, dst_ip),
      neutron:port(vm, src_ip),	//finds out the port that has the VM’s IP
      rep_dbms:ip_blacklist(dst_ip).

Policy actions
--------------

* Monitoring: report/log the incident including the VM’s IP address, external
  IP, etc.

* Reactive: Invoke the neutron API to add a new rule to the port’s security
  group that blocks all traffic to/from the VM’s IP address (this is one of
  multiple ways of isolating the compromised VM)

Proposed features (pre-spec/blueprints details)
-----------------------------------------------

* TBD

+4 Vulnerable software
~~~~~~~~~~~~~~~~~~~~~~

Votes: +1 banix, kudva, skn , su zhang

Vulnerability scanner reports a VM running an unpatched OS or application
software, for instance, when a new vulnerability is reported and a patch is
made available by the software vendor.  As this is reported, some reactive
measures would need to be taken, such as redirecting the traffic for this VM to
another.  This policy would facilitate a reactive response to be taken when
such an incident is reported.

Data Sources
------------

* VulScanner: <IP, port> that runs a vulnerable service
* NVD (national vulnerability database)
* ???Info about pre-conditions and post-conditions

More specific information needed. For example pre-condition, post-condition of
the exploit.

Policy
------

.. code:: console

   error(vm) :-
      nova:virtual_machine(vm),
      vulscanner:vulnerable(vul_ip, vul_port),
      neutron:port(vm, vul_ip).

Policy actions
--------------

* Monitoring: log/report when a new vulnerable service is reported

* Reactive: take one of many corrective actions based on configuration
  parameters.

  * Option 1: deny all incoming traffic to this service ip/port

  * Option 2: install a new patched service in a new VM and redirect the
    traffic

Proposed features (pre-spec/blueprints details)
-----------------------------------------------

* TBD

+2 Honeypot redirection for inbound traffic
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Votes: +1 banix , skn

IDS service sees some suspicious inbound traffic and unsure of its legitimacy
but needs a reactive measure such as redirecting the traffic to a honeypot
created and deployed on-the-fly to understand it better.  This policy needs to
facilitate such mechanisms.

Data sources
------------

* IDS: <src_ip:src_port, dst_ip:dst_port> for the suspicious traffic

Policy
------

Policy actions
--------------

* Monitoring

* Reactive: Use sub-policy implemented by Neutron and HoneyPot service

+1 Virtual Machines Placement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Votes: +1 pettori

There are many examples where one would want Prod VMs to be on only Prod
Hypervisor and no dev/test VMs (PCI for example), or a Virtual Machine that
needs lot of throughput should go on a server that has 10G NIC and is not too
utilized

Data Sources
------------

* Nova

* Neutron

Policy
------

.. code:: console

  error(vm) :-
     nova:instance(vm),
     nova:stage(vm,stagevm),
     nova:compute(server),
     nova:stage(server,stageserver),
     not same_stage_group(stagevm, stageserver)
  same_stage_group(x,y) :-
     stage_group(x, g),
     stage_group(y, g)
  stage_group(“dev”, “devtest”)
  stage_group(“test”, “devtest”)
  stage_group(“prod”, “prod”)

Policy actions needed
---------------------

What type of action does Congress need to take for this use case? Examples:

* Proactive: Responding to queries from other components about whether a given
  API call will cause new violations.

* Reactive: Computing/executing actions that bring the cloud back into
  compliance

+0 Trusted Compute Pool
~~~~~~~~~~~~~~~~~~~~~~~

Votes:

Hardware enforced trust as an attribute for policy declaration. Only deploy VMs
in a pool with hardware attributes. Geofencing as a direct scheduler based on
attributes like VM and hosts. Intel was interested in this option.

Data sources
------------

Policy
------

Policy actions
--------------

+4 Placement and Scheduling for Cloud & NFV Systems based on PSCN
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Votes: +4 ramki, dilip, norival, madhu

The goal of PSCN is optimized placement and scheduling of Cloud/NFV systems
based on resource constraints and service request requirements. The initial key
goals are as follows

* IaaS scheduling across DCs of service provider #1 for supporting asynchronous
  replication model for state data as specified in the service request
  requirements by service provide #2 [NFV-USE-CASE]. The primary goal is to
  optimize resource utilization of network and energy in the infrastructure of
  service providers #1and #2. This applies to scheduling within a provider too.

* NFVIaaS placement across distributed NFV DCs of service provider #1 with
  various resource constraints such as compute, network and service request
  requirements as specified by service provider #2 [NFV-USE-CASE]. The primary
  goal is to optimize resource utilization of compute and network in the NFV
  infrastructure of service provider #1. This applies to placement within a
  provider too.

Data sources
------------

* Nova (Compute): VM – Tenant, Virtual Network Function (VNF), Server, NFV
  Data Center, Service Provider (needed for inter-provider use cases)

* Neutron (Network): Inter-DC Network - Tenant, Virtual Network Function (VNF),
  Service Provider (needed for inter-provider use cases), static network
  bandwidth, dynamic network bandwidth utilization

* Nova (Energy): Server power consumption in various idle states, Dynamic power
  consumption in servers (leverage IPMI or other standards)

Policy
------

* Hourly/daily/weekly time windows for scheduling certain category of
  application workloads (a good example would be asynchronous replication
  across DCs)

* Restrict VM placement only to certain DCs

* Maximum energy consumption per DC

* Dedicate capacity for example VMs in a DC, inter-DC network bandwidth etc.
  for handling certain types of workloads

Policy actions
--------------

* Notify applications of precise time windows (order of minutes) for scheduling
  workloads

Security group management (implemented)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Security group management policies check against existing security group
properties. In our implementation at Symantec, we alert security groups with no
incoming traffic control. . We trace such violation at both tenant and instance
level.  We also have policies to further highlight those violated VMs with
internet access.

Data Sources:
-------------

* Nova

* Neutron

Policies:
---------

.. code:: console

   // Name: tenants with no ingress control for certain ports
   ingress_free_tenant(tenant_id, security_group_id, protocol, ethertype,
   port_range_min, port_range_max):-
   neutronv2:security_group_rules(security_group_id, id, tenant_id,
   remote_group_id, "ingress", ethertype, protocol, port_range_min,
   port_range_max, "0.0.0.0/0")

   // Name:  instances with no ingress control for certain ports
   ingress_free_vm(vm_id, tenant_id, security_group_id):- neutronv2:
   security_group_port_bindings (port_id, security_group_id),
   neutronv2:ports(port_id, tenant_id, name, network_id, mac_address,
   admin_state_up, status, vm_id, port_type),
   ingress_free_tenant(tenant_id, security_group_id, protocol, ethertype,
   port_range_min, port_range_max)

   // Name: instances with internet access
   connected_to_internet(port_id, vm_id) :-
   neutronv2:external_gateway_infos(router_id=router_id, network_id=
   network_id_gw), neutronv2:ports(network_id = all_network, device_id =
   router_id),
   neutronv2:ports(network_id = all_network, id=port_id, device_id=vm_id),
   nova:servers(id=vm_id)

   // Name: violated instances with internet access
   external_ac_ingress_free_vm(vm_id, port_id):- connected_to_internet(port_id,
   vm_id), ingress_free_vm(vm_id, tenant_id, security_group_id)

References:
-----------

* [NFV-USE-CASE] “ETSI GS NFV 001Network Functions Virtualization (NFV); Use
  Cases,” `http://www.etsi.org/deliver/etsi_gs/NFV/001_099/001/01.01.01_60/
  gs_NFV001v010101p.pdf`

* [NFV-ARCH] “NFV Architectural Framework,” `http://www.etsi.org/deliver/
  etsi_gs/NFV/001_099/002/01.01.01_60/gs_NFV002v010101p.pdf`

* [NFV-REQ] “NFV Virtualization Requirements,” `http://www.etsi.org/deliver
  /etsi_gs/NFV/001_099/004/01.01.01_60/gs_NFV004v010101p.pdf`

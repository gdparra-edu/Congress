======================
Congress Examples
======================

Secure Network
~~~~~~~~~~~~~~

Rules
-----

A VM connected to Internet should be protected by a security group that is
called "secure"

Helper Table
------------

* Define a table that contains all the VM ports and the associated security
  group:

.. code:: console

   port_security_group(port, security_group_name) :-
      neutron:ports(addr_pairs, security_groups, extra_dhcp_opts,
              binding_cap, status, name, admin_state_up, network_id,
              tenant_id, binding_vif, device_owner, mac_address, fixed_ips,
              port, device_id, binding_host_id1),
      neutron:ports.security_groups(security_groups, security_group_id),
      neutron:security_groups(tenant_id2,security_group_name, desc2,
              security_group_id)

* Define a table that contains all the routers ID and the network ID to which
  that router is attached

.. code:: console

   router(network_id, router_id) :-
      neutron:routers(status1, external_gateway_info1, networks1, name1,
              admin_state_up1, tenent_id1, router_id),
      neutron:ports(addr_pairs2, security_groups2, extra_dhcp_opts2,
              binding_cap2, status2, name2, admin_state_up2, network_id,
              tenant_id2, binding_vif2, device_owner2, mac_address2,
              fixed_ips2, port2, router_id2, binding_host2)

* Define a table that contains all which device ID are connected to the
  internet through which port ID

.. code:: console

   connected_to_internet(device_id, port_id) :-
      neutron:ports(addr_pairs, security_groups, extra_dhcp_opts, binding_cap,
              status, name, admin_state_up, network_id, tenant_id,
              binding_vif, device_owner, mac_address, fixed_ips, port_id,
              device_id, binding_host_id),
      router(network_id, router_id),
      neutron:routers(status2, external_gateway_info, networks2, name2,
              admin_state_up2, tenant_id2, router_id),
      not neutron:routers(status2, \"None\", networks2, name2,
                  admin_state_up2, tenant_id2, router_id)

Error Rules
-----------

I have an error, if a Virtual Machine that is connected to Internet is not
using the security group called secure

.. code:: console

   error_secure(vm) :-
      nova:servers(vm, name, host, status, tenant_id, user_id, image_id, flavor_id),
      connected_to_internet(vm, port),
      not port_security_group(port, \"secure\")

Cloud Foundry Examples
~~~~~~~~~~~~~~~~~~~~~~

Rules
-----

#. A production application must be running at least 2 instances
#. Production applications can only be connected to production services and
   vice versa

Helper Table
------------

* We need to create a table to define what a production app means

.. code:: console

   prod_apps(guid) :-
      cloudfoundryv2:apps(guid=guid,name=name,production="True")

* We need to create a table to define what a production service means

.. code:: console

   prod_services(guid) :-
      cloudfoundryv2:services(guid=guid, name=name, service_plan_name="25mb")

Error Rules
-----------

* Report an scale error if there less than 2 production instances running

.. code:: console

   error_scale(guid, name) :-
      prod_apps(guid), cloudfoundryv2:apps(guid=guid, name=name,
         instances=instances),
         lt(instances,2)

* Report an binding error if a production application is binded to a non
  production service OR if a production service is binded to a non production
  application

.. code:: console

   error_binding(app_guid, app_name, service_instance_guid, service_name) :-
      cloudfoundryv2:service_bindings(app_guid, service_instance_guid),
      prod_apps(app_guid),
      not prod_services(service_instance_guid),
      cloudfoundryv2:services(guid=service_instance_guid, name=service_name),
      cloudfoundryv2:apps(guid=app_guid,name=app_name)

   error_binding(app_guid, app_name, service_instance_guid, service_name) :-
      cloudfoundryv2:service_bindings(app_guid, service_instance_guid),
      prod_services(service_instance_guid),
      not prod_apps(app_guid),
      cloudfoundryv2:services(guid=service_instance_guid, name=service_name),
      cloudfoundryv2:apps(guid=app_guid,name=app_name)

OpenStack CPU Utilization
~~~~~~~~~~~~~~~~~~~~~~~~~

Rules
-----

* Virtual Machines should be using at least 10 of average CPU

Helper Table
------------

* We need to create a table that will contain all the Virtual Machines that
  have an average CPU utilization of less than 10%

.. code:: console

   reclaim_server(vm) :-
      ceilometer:statistics("cpu_util",vm, avg, d,e,f,g,h,i,k,l,m,n,o),
      lt(avg, 10)

Error Rules
-----------

* There is an error showing the name of the user, his email and the name of the
  Virtual Machine for all the Virtual Machine reported in reclaim_server

.. code:: console

   error (user_id, email, vm_name) :-
      reclaim_server(vm),
      nova:servers(vm, vm_name, a, b, c, user_id ,d ,e ),
      keystone:users(h, f, q, w, user_id, email)


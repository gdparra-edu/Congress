==============================
Relationship to other projects
==============================

Related OpenStack components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* `Keystone <https://wiki.openstack.org/wiki/Keystone>`__:
  Keystone is an identity service providing authentication and high-level
  authorization for OpenStack. Congress can leverage Keystone as an input
  for policies. For example, an auditor might want to ensure that the
  running system is consistent with current Keystone authorization decisions.

* `Heat <https://wiki.openstack.org/wiki/Heat>`__:
  Heat is an orchestration service for application provisioning and lifecycle
  management. Congress can ensure that applications managed by Heat are
  consistent with business policy.

* `Mistral <https://wiki.openstack.org/wiki/Mistral>`__:
  Mistral is a task management service, or in other words Workflow as a
  service. Its primary use cases include Cron-for-the-cloud, execution of
  long-running tasks, and big data analysis. Congress could potentially utilize
  Mistral to execute actions that bring the cloud back into policy compliance.

Policy initiatives
~~~~~~~~~~~~~~~~~~

* `SolverScheduler (Nova blueprint)
  <https://blueprints.launchpad.net/nova/+spec/solver-scheduler>`__:
  The SolverScheduler provides an interface for using different constraint
  solvers to solve placement problems for Nova. Depending on how it is applied,
  could be used for initial provisioning, re-balancing loads, or both.

* `Gantt <https://github.com/openstack/gantt>`__:
  A scheduler framework for use by different OpenStack components. Used to be a
  `subgroup of Nova <https://wiki.openstack.org/wiki/Meetings/Scheduler>`__
  and focused on scheduling VMs based on resource utilization. Includes plugin
  framework for making arbitrary metrics available to the scheduler.

* `Neutron policy group <https://docs.google.com/document/d/
  1ZbOFxAoibZbJmDWx1oOrOsDcov6Cuom5aaBIrupCD9E/edit?pli=1>`__:
  This group aims to add a policy API to Neutron, where tenants express policy
  between groups of networks, ports, etc., and that policy is enforced. Policy
  statements are of the form "for every network flow between groups A and B
  that satisfies these conditions, apply a constraint on that flow". The
  constraints that can be enforced on a flow will grow as the enforcement
  engine matures; currently, the constraints are Allow and Deny, but there
  are plans for quality-of-service constraints as well.

* `Open Attestation <https://wiki.openstack.org/wiki/OpenAttestation>`__:
  This project provides an SDK for verifying host integrity. It provides some
  policy-based management capabilities, though documentation is limited.

* `Policy-based Scheduling Module (Nova blueprint)
  <https://blueprints.launchpad.net/nova/+spec/policy-based-scheduler>`__:
  This effort aims to schedule Nova resources per client, per cluster of
  resources and per context (e.g. overload, time, etc.). A proof of concept
  was presented at the Demo Theater at OpenStack Juno Summit.

* `Tetris <https://docs.google.com/document/d/1DMsnGxQ3P-OwZCF3uxaUeEFaKX8LqUq
  mmgQ_7EVK7Y8/edit>`__: This effort provides condition-action policies (under
  certain conditions, execute these actions). It is intended to be a generic
  condition-action engine handling complex actions and optimization. This
  effort subsumes the `Runtime Policies blueprint <https://blueprints.launchpad
  .net/nova/+spec/resource-optimization-service>`__ within Nova. It also
  appears to subsume the `Neat <http://openstack-neat.org/>`__ effort. Tetris
  and Congress have recently decided merge because of their highly aligned
  goals and approaches.

* `Convergence Engine (Heat spec) <https://review.openstack.org/#/c/95907/6/
  specs/convergence.rst>`__:
  This effort separates the ideas of desired state and observed state for the
  objects Heat manages. The Convergence Engine will detect when the desired
  state and observed state differ and take action to eliminate those
  differences.

* `Swift Storage Policies <http://docs.openstack.org/developer/swift/
  overview_policies.html>`__:
  A Swift storage policy describes a virtual storage system that Swift
  implements with physical devices. Today each policy dictates how many
  partitions the storage system has, how many replicas of each object it
  should maintain, and the minimum amount of time before a partition can be
  moved to a different physical location since the last time it was moved.

* `Graffiti <https://wiki.openstack.org/wiki/Graffiti>`__:
  Graffiti aims to store and query (hierarchical) metadata about OpenStack
  objects, e.g. tagging a Glance image with the software installed on that
  image. Currently, the team is working within other OpenStack projects to
  add user interfaces for people to create and query metadata and to store
  that metadata within the project's database. This project is about creating
  metadata, which could be useful for writing business policies, not about
  policies over that metadata.

Congress is complementary to all of these efforts. It is intended to be a
general-purpose policy component and hence could (potentially) express any of
the policies described above. However, to enforce those policies Congress would
carve off subpolicies and send them to the appropriate enforcement point in the
cloud. For example, Congress might carve off the compute Load Balancing policy
and send it to the Runtime Policy engine and/or carve off the networking policy
and sending it to the Neutron policy engine. Part of the goal of Congress is to
give administrators a single place to write and inspect the policy being
enforced throughout the datacenter or cloud, distribute the relevant portions
of that policy to all of the available enforcement points, and monitor the
state of the cloud to let administrators know if the cloud is in compliance.
(Congress also attempts to correct policy violations when they occur, but
optimization policies such as many of those addressed above will not be
enforceable by Congress directly.)

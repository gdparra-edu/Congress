================
What is congress
================

Congress aims to provide an extensible open-source framework for governance and
regulatory compliance across any cloud services (e.g. application, network,
compute and storage) within a dynamic infrastructure. It is a cloud service
whose sole responsibility is policy enforcement.

Congress aims to include the following functionality:

* Allow cloud administrators and tenants to use a high-level, general purpose
  declarative language to describe business logic. The policy language does not
  include a fixed collection of policy types or built-in enforcement
  mechanisms; rather, a policy simply defines which states of the cloud are in
  compliance and which are not, where the state of the cloud is the collection
  of data provided by the cloud services available to Congress. Some examples:

  * Application A is only allowed to communicate with application B.

  * Virtual machine owned by tenant A should always have a public network
    connection if tenant A is part of the group B.

  * Virtual machine A should never be provisioned in a different geographic
    region than storage B.

* Offer a pluggable architecture that connects to any collection of cloud
  services

* Enforce policy

  * Proactively: preventing violations before they occur

  * Reactively: correcting violations after they occur

  * Interactively: give administrators insight into policy and its violations,
    e.g. identifying violations, explaining their causes, computing potential
    remediations, simulating a sequence of changes.

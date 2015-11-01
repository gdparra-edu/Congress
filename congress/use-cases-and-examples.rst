======================
Use cases and examples
======================

Detailed use cases and examples can be viewed on:

.. toctree::
   :maxdepth: 2

   use-cases-and-examples/congress-use-cases.rst
   use-cases-and-examples/congress-examples.rst

Here we give an example of policies that each of our releases supports.

.. list-table::
   :header-rows: 1
   :widths: 153 27

   * - Use case
     - Congress release
   * - Monitor violations of: every network connected to a VM must either be
       public or owned by someone in the same group as the VM's owner
     - alpha
   * - Stop a user from constructing a new VM if she owns a VM averaging less
       than 1% CPU-utilization (requires API gateway or Nova support)
     - kilo
   * - Every time a user creates a Neutron security group that opens port 80,
       delete that security group
     - kilo


# Virtual Machines Manager

Set up and run virtual machines to test software within the safety of a sandboxed environment or complement your disaster recovery plan.

For more details see the [product page](https://www.synology.com/dsm/feature/virtual_machine_manager).

## Networking

If you configured **Link Aggregation** for two interfaces without **Open vSwitch** (e.g. _Automatic Load Balancing_), you need to remove the bond interface, enable Open vSwitch and recreate it back with the related typology.

As an example:
- Control Panel
- Network -> Network Interface
- Select the bond interface and click on _Delete_
- Enable Open vSwitch
- Create a new bond interface with a related typology (e.g. _Balance-SLB_)
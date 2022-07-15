# Work Group Conclusions for July 2022

## Technology to be used

This work group has concluded that the best solution ALP can offer for 1:1 system management is
actually the combination of two solutions.

* The main 1:1 system management tool should be Cockpit. Since running the main components
of Cockpit (`cockpit-bridge` and the Cockpit extensions) inside a container is not feasible in the
mid term, it would be necessary to install Cockpit into the system to make use of it. The packaged
Cockpit will present a properly themed look&feel and a set of extensions known to work properly
with ALP. Some additional extensions will be available as separate packages.
* A fully containerized version of YaST will be offered as an alternative for those cases in which
Cockpit doesn't fit or the YaST approach is preferred for any other reason (eg. users migrating
from SLE). It will include many of the modules and functionalities offered by YaST in SLE-15-SPx,
but not all of them. It will only require Podman or Docker (or any other engine for running OCI
containers).

## Commitment for September's Prototype

The plan of this work group is to stay working as a group and take the responsibility of delivering
a functional Cockpit and a useful containerized YaST for the ALP prototype in September.

Since the scope of this first prototype is limited, some use-cases may not be covered. That includes
most of the traditional usages of openSUSE Leap.

### Plans for Cockpit

This work group would like to take ownership of the Cockpit (and related) packages on the
`devel:LEO/ALP` project. Being in close control will allow us to ensure we provide a properly themed
and adapted version of Cockpit.

Apart from proper branding, that initial version will offer limited functionality including:

* Inspecting the system logs
* Managing systemd units
* Basic creation of system users
* Podman administration
* Terminal emulator

If NetworkManager is used to handle the network in the system (something that we are not taking for
granted for all ALP systems at this point in time), the Cockpit version at September's prototype
should also offer:

* Configuration of network interfaces
* Limited support for configuring network bonds, bridges and VLANs
* Management of Firewalld zones and rules

Installing some extra packages (bear in mind Cockpit and its dependencies need to be installed
directly in the system to be managed), it will also be possible to perform:

* Basic management of storage devices (eg. creating and formatting partitions)
* Visualization of some metrics gathered by PCP
* Limited management of virtual machines

And last but not least, we also aim to offer a way to manage software updates through Cockpit for
the September's prototype. But it will work in completely different ways depending on whether the
system is transactional or not. In the first case, the `cockpit-tukit` package should be installed
and used. For non-transactional system `cockpit-packagekit` should be used. We don't plan to
introduce any new mechanism to handle that automatically, so it may happen that the user would need
to install the right package based on its use-case.

In that regard, we plan to offer some short documentation on how to install and run Cockpit in both
a transactional and a non-transactional ALP system.

We don't expect Cockpit to work flawlessly on this first prototype. In particular, we are concerned
about some areas like proper management of configurations that are split over several files at
`/usr/etc` and `/etc`.

The commitment of the current group to maintain Cockpit beyond the first ALP prototype is subject to
the condition of getting valuable feedback from that prototype. There is no way to evolve the
solution further without information about what is missing or what doesn't fit the real needs of the
ALP use-cases.

### Plans for Containerized YaST

The commitment of this work group for September also extends to offering a first version of
containerized YaST with three working interfaces: graphical (Qt), text-mode (ncurses) and web via
VNC.

The group will provide some documentation about how to use the containerized YaST in a very
straightforward way in both transactional and non-transactional systems.

In that regard, we expect the YaST functionality to be somehow limited in transactional systems.
YaST usually needs to install software in the target system in order to configure it. But YaST
cannot handle software installation in transactional systems and will not be adapted in time for
the prototype in September.

YaST will offer quite some functionality for September's prototype included (but not limited to):

* Management of software and repositories
* Bootloader installation and tweaking
* Firewall configuration
* Management of system services
* Advanced kdump and fadump administration
* Configuration of iSCSI devices (iSCSI LIO target)
* Inspection of the systemd journal
* Management of users and groups
* Configuration of timezone, keyboard layout and language (if available in ALP)

Apart from that, YaST will offer full capabilities for configuring the network in systems using
Wicked (if that is possible in ALP).

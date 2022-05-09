# Requisites: 1:1 Management Tool(s) for ALP Host OS

## General considerations

The tool(s) should enable easy and convenient management of a so-called pet system (as opposed to
cattle). Although SLE offers tools for configuring every aspect of a particular system (basically
YaST and its myriad of modules), 1:1 management on ALP should be an exception rather than the rule.
It can be assumed that systems that are part of either a data-center or cloud or edge infrastructure
will be administered from a 1:many control system and only occasionally will be touched directly.

As a consequence, offering backwards compatibility in terms of what is configurable in SLE is not
a goal for the 1:1 management system of ALP. Moreover, the usual mechanisms used in SLE to specify
the configuration of a system (eg. AutoYaST profiles) are not expected to work in ALP unless
explicitly mentioned in this document.

The tool should focus on easing the management of a remote system (eg. the administrator connects
to the system using a web browser) but it should also allow local administration (eg. using a
keyboard and a screen connected to the system) when possible.

Since ALP will be based on Tumbleweed, it will follow the same pattern of splitting the system
configuration with a cascade approach like `/etc/example.conf` -> `/usr/etc/example.conf` ->
`/usr/etc/example.d/*.conf` -> `/etc/example.d/*.conf`.

The ALP Host OS will be an immutable transactional system. Most of the filesystem hierarchy will be 
read-only except a few exceptions. The `/etc` and `/var` directories will be read-write by design.
Moreover, these other directories may be read-write in the first versions of ALP, but in principle
they are not really intended to be modified by the system management tool: `/usr/local`, `/opt`,
`/srv` and `/home`.

## Use cases

Situations in which a 1:1 management system may be needed in the context of ALP.

**NOTE:** these use cases should be validated by the ALP architects. Removing and adding use
cases if needed. Then they should be rated according to its priority and urgency.

### Prepping system right after installation

Ideally, all the initial tasks to be performed after installation in order to get the system working
and managed through a 1:many system (even a self-hosted one) should be automated. But sometimes it
may be useful to offer a convenient way to manually setup some aspects like:

  - upload an initial ssh certificate for remote management
  - change the root password
  - create a management user
  - configure the network and/or the firewall to enable remote management

### Emergency access

Even in system managed through automated solutions (e.g., Salt, Ansible, Kubernetes Cluster API),
there may be cases in which such solutions are not able to control the system due to some problem.
In that case, the 1:1 system should allow to perform the tasks needed to recover control. Some
examples:

  - diagnose what is wrong by inspecting the logs and the status (network ports, services, etc.)
  of the system
  - restart the services associated to the 1:many management system
  - fix some authentication problem
  - manually trigger a rollback of the system

### Demo and testing

We may want to offer an easy way of installing and checking APL-based products on a test node or on
the desktop (e.g., by integrating them into the Rancher Desktop framework). In that case, it would
be great to be able to log into the 1:1 console to adjust the configuration and then download,
install and start up those products as VMs or containers.

### Configuration by example

Sometimes it's easier to set up a system once with an interactive tool and then clone the result
into a reproducible format than crafting a declarative configuration from scratch. That could be the
case in a lab or pre-production setting. Once the system behaves as expected, the operator would
want to export the configuration and then re-use it later (usually parametrized for the actual
production systems) from a 1:many management framework.

## Non-functional requirements

**NOTE:** these requirements should be qualified and validated by the ALP architecs, adding
new ones if needed. Then they should be rated according to its priority and urgency.

There should be a minimum impact on the size of ALP Host OS, both in terms of bytes and of
components that would need to be included in such a base system. If possible, all components of the
1:1 management solution should run as optional containers on top of the Host OS.

Should have (virtually) zero CPU and memory footprint if not used.

Should have small CPU and memory footprint when in use.

Should have a reasonable disk footprint when installed.

The actions of the tool should be accountable, so it's possible to inspect the changes and internal
decisions done while configuring a test system in order to generalize and translate them for the
1:many system that will be used to replicate that configuration in the production environment.

**MISSING:** Security requirements (eg. open ports needed to use the tool, logging of the 
activity done in the tool...) and their rationale.

**MISSING:** Accessibility requirements.

## Functional requirements

The tool should offer an intuitive and guided way to manage the following aspects of the system.

**IN PROGRESS:** Which aspects of the system should be configurable with the 1:1 system and to
what extent? Initially we are just throwing into the document everything we can think of, based on
what Cockpit, Ajenti2 and YaST can do. That doesn't mean all the requirements will make it to the
final version of the document.

### Reboot or Shutdown the Machine

It should be possible to immediately reboot or shutdown the machine or to schedule such an action
for a given date and time.

### Perform Transactional Updates and Rollbacks

TO BE WRITTEN

### Management of Snapshots

TO BE WRITTEN

  - Configure Snapper beyond the default transactional-updates config
  - Inspect snapshots to restore concrete files, etc.

### Inspect the System Logs

It should be possible to check the systemd journal and maybe other sources of information that could
be useful to debug the system. The tool should include some mechanisms to filter and search the
information as well as a way to download the bulk information for processing it with other tools.

### Configuration of systemd Units

There should be an easy and convenient way to start/stop and to enable/disable services, with quick
access to their logs to diagnose any potential problem when modifying their status. Going beyond
"services" in the traditional sense, the tool should allow to manage the different kinds of systemd
units like services, sockets, timers and targets.

### Configure Date, Time and NTP

Timezone, date and time should be configurable both statically and through NTP. Since the latter is
more common in big deployments, it should be possible to tweak several aspects like specifying which
servers to use (both public and local), the method and frequency to sync, etc.

### Configure Local Name and Domain

This is closely related to the network configuration. It should be possible to configure whether the
hostname and localdomain name are static (and set the names in that case) or configured via DHCP
(and specify which one of the existing network interfaces defines the names).

### Network Configuration

TO BE WRITTEN:

  - Configure the interfaces
    - ipV4, v6, routes, etc.)
    - iBFT
    - Bonding
    - Device activation
    - Udev rules
    - Firewall zones
    - Kernel modules
  - DNS (check tab in YaST): DNS policy, static DNS servers, domain search...
  - PROXY
  - VPN

### Firewall Management

TO BE WRITTEN

### Configuration of the Authentication System

TO BE WRITTEN

  - PAM
  - Connect to LDAP, Active Directory, Kerberos or other systems
  - Password settings, encryption method and so on
  - Delay after incorrect login attempts
  - Default SID, GID and many other stuff from useradd and logindef config

### Windows Domain Membership

TO BE WRITTEN

### Management of Local Users and Groups

TO BE WRITTEN

### Configuration of Language and Keyboard

TO BE WRITTEN (note: in SLE this implies reconfiguring the bootloader and regenerating initrd)

### Bootloader configuration

TO BE WRITTEN

### Kdump Management

TO BE WRITTEN

### Configure Security

TO BE WRITTEN

  - Apparmor and/or SELinux (access to boot arguments?)
  - CPU mitigation (access to boot arguments?)
  - TCP syncookies
  - Default file permissions

### Configure Storage Devices and Technologies

TO BE WRITTEN

  - Types of devices:
    - Local devices (hard disks, NVMe, removable devices, etc.)
    - DASDs - IBM s/390 mainframe (virtual) disks that needs previous activation process
    - Network storage (configuration needed):
      - iSCSI including support for iBFT
      - FCoE (and zFCP)
      - NVMe oF/TCP including support for the upcoming NBFT
      - NFS
      - SMB
      - DRDB
 - Storage organization:
    - Partitions (including several partition table types like GPT, MSDOS or DASD).
    - LVM (including snapshots, thin provisioning, etc.)
    - Btrfs capabilities (snapshots, subvolumes, multi-volume, RAID...)
    - Multipath
    - RAID (software RAID, pure firmware RAID and some intermediate solutions)
    - Tmpfs
    - Encryption (LUKS, TPM2, IBM's pervasive encryption...)
    - Caching (bcache)

### Management of Virtual Machines

TO BE WRITTEN

### Management of Containers

TO BE WRITTEN

### Management of Backups

TO BE WRITTEN

### Performance tunning

TO BE WRITTEN

### Create Support Inquiries

TO BE WRITTEN: some way of assisting the user to open a support case if help is needed

## References

  - https://confluence.suse.com/display/LEONG/1%3A1+System+Management
  - https://confluence.suse.com/display/LEONG/2022-04-29+WG-+ALP%3A+1%3A1+System+management+Work-Group
  - https://confluence.suse.com/pages/viewpage.action?spaceKey=~joachimwerner&title=1%3A1+Systems+Management+Considerations
  - https://gist.github.com/ancorgs/dbd25f8910cce7ffdff0b3d9cdb3b937

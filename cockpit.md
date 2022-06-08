# Cockpit as 1:1 System Management Solution for ALP

## Some Notes About this Document

This document contains a first evaluation of Cockpit as the main candidate to be the 1:1 System
Management solution for ALP. To put everything in context, the document also offers a comparison on
how some aspects and functionalities compare to the traditional SUSE solutions for the same goal
(mainly, but not limited to, YaST).

The document contains some open questions marked as **[FURTHER INVESTIGATION NEEDED]** or **[TO BE
WRITTEN]**, but is already complete enough to have a good view of the pros and cons of Cockpit.

## Scope

This document assumes the goal of the 1:1 system management solution is to configure and administer
the so-called ALP Host OS. In principle, there are no expectations of being able to use
Cockpit (or any similar tool) to configure any aspect of the so-called _workloads_ (software
running on containers and/or virtual machines instead of the Host OS)?

## Support for the `/etc` + `/usr/etc` Layout

Since ALP will be based on Tumbleweed, it will follow the same pattern of splitting the system
configuration with a cascade approach like `/etc/example.conf` -> `/usr/etc/example.conf` ->
`/usr/etc/example.d/*.conf` -> `/etc/example.d/*.conf`.

Currently Cockpit cannot deal with such a configuration split. **[FURTHER INVESTIGATION NEEDED]**
How hard would be to get the required changes upstream to make it possible?

Many parts of YaST are already adapted to this. Although YaST contains some generic mechanisms to
handle the situation, adaptations are done in a case-by-case basis because different applications
follow different rules for precedence, semantic, overwriting (parameter vs section vs file), etc.
See [etc-and-usr-etc.md](https://github.com/yast/yast-yast2/blob/master/doc/etc-and-usr-etc.md).

## Support for immutable systems

The ALP Host OS will be an immutable transactional system. Most of the filesystem hierarchy will be
read-only except a few exceptions. The `/etc` and `/var` directories are read-write "by design".
Moreover, these other directories will very likely be read-write, but in principle they are not
really intended to be modified by the system management tool: `/usr/local`, `/opt`, `/srv` and
`/home`.

Right now, both YaST and Cockpit assume a read-write system in which they can alter any part of the
configuration, directly or by delegating that to some commands. In general, it may look that being
able to modify `/etc`, `/var`, `/srv` and `/home` should be enough to configure a system. But
experience with YaST shows that's not necessarily true for all features people demand from a 1:1
configuration system. See some details at [this SUSE-internal Trello
card](https://trello.com/c/innXZvso/). Problematic areas include (but are not limited to) writing to
`/boot` (necessary nowadays even for making persistent changes in the system language), installing
packages or calling some commands like `firewall-cmd --state`.

Currently there are no realistic mid-term plans to make YaST fully work in transactional systems.
**[FURTHER INVESTIGATION NEEDED]** What about Cockpit?

## Cockpit vs YaST: the Usage Approach

Cockpit is quite immediate and interactive by design. Whenever the user changes something in the
interface, the system is modified and the change takes effect immediately. It also updates the UI
dynamically to reflect changes in the system even if they were not triggered by Cockpit, eg. a
network interface being (dis)connected.

YaST on the opposite works in three phases. First it reads the system and creates a representation
of its state, then it allows the user to adjust that representation to reflect how the system should
look like, finally it applies the changes to the system. It assumes nothing has changed in the
system meantime.

## Managing Multiple Systems from a Single Cockpit Interface

In principle, the Cockpit web interface is used to control the system in which it's running. But it
also offers the possibility to add additional hosts so all of them can be controlled with a single
instance of the web interface. In any case, that feature may be considered as secondary since as
Jiri pointed "_the focus should be on 1:1 (pet) management; There are better ways for
mass-management_".

![add host](https://gist.github.com/ancorgs/dbd25f8910cce7ffdff0b3d9cdb3b937/raw/de4bced95f45c9f3db4500220858ff5ecfd07194/zz_add_host.png)

**[FURTHER INVESTIGATION NEEDED]** Thorsten pointed this feature may be dead and removed from
upstream Cockpit. 

To control a remote system, the following two criteria need to be met:

 - Cockpit and all the corresponding extensions (see below for more information about extensions and
   functionality) must be installed in the system to be managed.
 - The system hosting the web interface must have SSH access to the system to be managed.

## Functionality vs Footprint

A basic installation of Cockpit has a quite small memory and hard disk footprint. The set of
dependencies is also quite reduced, although that will change (see note below about
`cockpit-bridge` rewrite). Such a basic installation provides limited functionality. Mainly:

 - [inspect the systemd journal](#inspecting-the-logs)
 - reboot or shutdown the machine
 - change the hostname
 - join an LDAP domain
 - set the timezone, date and NTP (the NTP part has [some problems](#datetime-management))
 - select a performance profile (if `tuned`, which depends on some python3 modules, is installed)
 - create and delete simple [user accounts](#users-management) (no management of groups or any
   advanced setting)
 - [configuration of systemd](#management-of-systemd-units-services-etc) services, targets, sockets,
   timers and paths
 - a pretty cool terminal emulator

Fortunatelly several `cockpit-*` extensions can be installed to expand the functionality. The
drawback is that each extension comes with its own dependencies. All the `cockpit-*` extensions must
be installed and executed, with all its dependencies, in the system being managed. Depending on the
extensions we want to provide for the ALP Host OS, that seriously conflicts with the goal of having
a minimal system.

Regarding the size of those dependencies, there are some extensions that are certainly slim:

 - **[cockpit-kdump](#kdump-configuration):** No important dependencies
 - **[cockpit-networkmanager](#network-and-firewalld):** Only depends on NetworkManager and firewalld
 - **[cockpit-sosreport](#collecting-system-debug-information-sos):** Only depends on sos
 - **cockpit-session-recording:** Only depends on tlog
 - **cockpit-pcp:** (4 packages) pcp pcp-conf pcp-libs pcp-selinux

Some other interesting extensions depend on python3 and some other packages:

 - **cockpit-navigator:** (3 packages) python3 rsync zip
 - **[cockpit-packagekit](#rpm-based-system-updates):** (4 packages) python3 python3-psutil tracer-common python3-tracer
 - **[cockpit-selinux](#selinux-management):** (4 packages) python3 python-dasbus setroubleshoot-plugins setroubleshoot-server
 - **[cockpit-file-sharing](#mounting-nfs-and-samba-shares):** (16 packages) python3 cups-libs libkadm5 lmdb python3-dns python3-ldb python3-samba
   python3-talloc python3-tdb python3-tevent samba-common-tools samba-dc-libs samba-libs tdb-tools samba
 - **[cockpit-storaged](#management-of-storage-devices):** (16 packages) python3 python3-dbus udisk2 clevis device-mapper-multipath-libs jose jq
   libjose libluksmeta luksmeta oniguruma tpm2-tools clevis-luks clevis-pin-tpm2 device-mapper-multipath
 - **[cockpit-composer](#image-builder-composer):** (39 packages) python3 bubblewrap container-selinux containers-common criu criu-libs crun fuse
   fuse-common fuse3 fuse3-libs libbsd libnet libslirp netavark osbuild osbuild-composer osbuild-composer-core
   osbuild-composer-dnf-json osbuild-composer-worker osbuild-luks2 osbuild-lvm2 osbuild-ostree osbuild-selinux ostree
   ostree-libs python3-attrs python3-jsonschema python3-osbuild python3-pyrsistent qemu-img rpm-ostree rpm-ostree-libs
   yajl aardvark-dns fuse-overlayfs skopeo slirp4netns
 - **[cockpit-389-ds](#administration-of-ldap-servers):** (99 packages) python3 cyrus-sasl-lib cyrus-sasl-plain nss nss-softokn nss-softokn-freebl nss-sysinit
   389-ds-base 389-ds-base-libs cyrus-sasl-md5 libdb-utils lmdb nss-tools openldap-clients openssl-perl perl-Algorithm-Diff
   perl-Archive-Tar perl-AutoLoader perl-B perl-Carp perl-Class-Struct perl-Compress-Raw-Bzip2 perl-Compress-Raw-Lzma
   perl-Compress-Raw-Zlib perl-DB_File perl-Data-Dumper perl-Digest perl-Digest-MD5 perl-DynaLoader perl-Encode nss-util
   perl-Errno perl-Exporter perl-Fcntl perl-File-Basename perl-File-Find perl-File-Path perl-File-Temp perl-File-stat
   perl-FileHandle perl-Getopt-Long perl-Getopt-Std perl-HTTP-Tiny perl-IO perl-IO-Compress perl-IO-Compress-Lzma
   perl-IO-Socket-IP perl-IO-Zlib perl-IPC-Open3 perl-MIME-Base64 perl-Net-SSLeay perl-POSIX perl-PathTools perl-Pod-Escapes
   perl-Pod-Perldoc perl-Pod-Simple perl-Pod-Usage perl-Scalar-List-Utils perl-SelectSaver perl-Socket perl-Storable
   perl-Symbol perl-Term-ANSIColor perl-Term-Cap perl-Term-ReadLine perl-Text-Diff perl-Text-ParseWords perl-Text-Tabs+Wrap
   perl-Tie perl-Time-Local perl-URI perl-base perl-constant perl-debugger perl-if perl-interpreter perl-libnet perl-libs
   perl-meta-notation perl-mro perl-overload perl-overloading perl-parent perl-podlators perl-sigtrap perl-subs perl-threads
   perl-threads-shared perl-vars python3-ldap python3-lib389 python3-pyasn1 python3-pyasn1-modules perl-Devel-Peek
   perl-IO-Socket-SSL perl-Mozilla-CA perl-NDBM_File
 - **[cockpit-machines](#management-of-virtual-machines):** (126 packages) python3 cyrus-sasl-gssapi cyrus-sasl-lib cyrus-sasl-plain alsa-lib cairo capstone
   cdparanoia-libs cyrus-sasl daxctl-libs edk2-ovmf fontconfig freetype fribidi fuse3-libs gnutls-dane gnutls-utils
   graphene graphite2 gstreamer1 gstreamer1-plugins-base harfbuzz iproute-tc ipxe-roms-qemu iso-codes kde-filesystem
   kf5-filesystem libX11 libX11-common libX11-xcb libXau libXext libXfixes libXft libXrender libXv libXxf86vm libburn
   libdatrie libdrm libepoxy libfdt libglvnd libglvnd-egl libglvnd-glx libisoburn libisofs libogg libpciaccess libpmem
   librdmacm libslirp libssh2 libthai libtheora libtpms libunwind libvirt-client libvirt-daemon libvirt-daemon-config-network
   libvirt-daemon-driver-interface libvirt-daemon-driver-network libvirt-daemon-driver-nodedev libvirt-daemon-driver-qemu
   libvirt-daemon-driver-storage-core libvirt-dbus libvirt-glib libvirt-libs libvisual libvorbis libwayland-client
   libwayland-cursor libwayland-egl libwayland-server libwsman1 libxcb libxshmfence libxslt linux-atm-libs llvm13-libs lzop
   mdevctl mesa-dri-drivers mesa-filesystem mesa-libEGL mesa-libGL mesa-libgbm mesa-libglapi mesa-vulkan-drivers ndctl-libs
   numad opus orc osinfo-db osinfo-db-tools pango python3-libvirt qemu-common qemu-img qemu-kvm-core qemu-system-x86-core
   qemu-ui-opengl qemu-ui-spice-core seabios-bin seavgabios-bin sgabios-bin spice-server swtpm swtpm-libs swtpm-tools
   systemd-container trousers trousers-lib unbound-libs usbredir virt-manager-common vulkan-loader xen-libs xen-licenses
   xml-common xorriso yajl libosinfo libvirt-daemon-driver-storage-disk qemu-block-curl qemu-char-spice qemu-device-usb-host
   qemu-device-usb-redirect virt-install

It's worth mentioning that Cockpit developers are in the process of rewriting `cockpit-bridge` with
Python. That's the core Cockpit component that needs to be installed in all managed systems. That
means in the mid-term python3 will become a dependency not only for the extensions mentioned above,
but even for the most basic usage of Cockpit. Note that, at the moment of writing, the ALP Host OS
includes python3 since it's also needed by some basic components of the system like SELinux or
Firewalld.

Finally, there is a third group of extensions that don't seem to currently bring python3 into the
dependency tree but that have a non-trivial set of dependencies:
  
 - **[cockpit-podman](#podman-administration):** (21 packages) conmon container-selinux containers-common criu criu-libs crun fuse-common
   fuse3 fuse3-libs libbsd libnet libslirp netavark podman shadow-utils-subid yajl aardvark-dns catatonit
   fuse-overlayfs slirp4netns
 - **[cockpit-ostree](#system-updates-with-ostree):** (24 packages) bubblewrap container-selinux containers-common criu criu-libs crun fuse
   fuse-common fuse3 fuse3-libs libbsd libnet libslirp netavark ostree ostree-libs rpm-ostree rpm-ostree-libs
   yajl aardvark-dns fuse-overlayfs skopeo slirp4netns 
   
Note the lists try to represent both the direct and indirect dependencies in a Fedora 36 box,
although they might not be 100% accurate.
 
## Functionality Compared to SLE Alternatives

This section give more details on what Cockpit can currently do. Comparing each area with the equivalent
tool that can be used to achieve the same in current versions of SLE.

### Basic System Management

#### Inspecting the Logs

Cockpit contains a viewer that is pretty similar in capabilities to
[yast2-journal](https://github.com/yast/yast-journal).  Maybe the YaST filters are a bit more
powerful, but both applications offer basically the same level of functionality.

![journal](https://gist.githubusercontent.com/ancorgs/dbd25f8910cce7ffdff0b3d9cdb3b937/raw/4106f1cf8439611007129adc7dd719990077cf41/zz_logs.png)

#### Date/time Management

NTP configuration in Cockpit is currently limited. As you can see in the screenshot below, the
option to use specific NTP servers is always greyed out. That's because the feature was written to
work with `systemd-timesyncd` instead of `chrony`, which is the system used by Red Hat, Fedora and
SLE. So that feature is basically pointless nowadays and would need to be rewritten.

![ntp](https://gist.githubusercontent.com/ancorgs/dbd25f8910cce7ffdff0b3d9cdb3b937/raw/4106f1cf8439611007129adc7dd719990077cf41/zz_ntp.png)

On the other hand, YaST allows to tweak `chrony` configuration in several aspects. See for example
[this section](https://yast.opensuse.org/blog/2020-04-02/sprint-96#ntp-servers) of a post in the
YaST blog (includes screenshots). It's worth mentioning that `yast2-ntp-client` contains some
(open)SUSE-specific logic (like selecting public vs private servers) and logic to deal with the
different involved systemd units (like `chronyd`, `chrony-wait` and systemd timers) depending on the
situation.

It may be possible to reuse the knowledge in YaST to add decent `chrony` support to Cockpit.

### Users Management

Cockpit allows to create, modify and delete local users in the system. It allows to define some
basic properties for those users, like the password and the SSH keys. See [this
article](https://fedoramagazine.org/managing-user-accounts-with-cockpit/) for some details.

The equivalent tool in SLE would be [yast2-users](https://github.com/yast/yast-users) which, despite
its pretty old and convulted code-base, offers way more functionality. For a summary of what
yast2-users can do, see the documents
[use-cases.md](https://github.com/yast/yast-users/blob/master/doc/use-cases.md) and
[plugins-system](https://github.com/yast/yast-users/blob/master/doc/plugins-system.md) at the
yast2-users repository.  Despite its broad range of functionality, yast2-users exhibits some
problems caused by the way YaST works (representing a full picture of how the system will look and
only performing the changes at the end). For example, its UI would need to be rethinked to work
consistently in systems with `USERGROUPS_ENAB=true`.

Some of the many features Cockpit lacks when compared to yast2-users include:

- Manage system users (100 <= UID <= 499)
- Adjust several user attributes like login shell, default group, additional groups, etc.
- Advanced management of home directory for a new user
  - Manage permissions
  - Allow to create users with empty home (whether to use skel or not)
  - Create home as a Btrfs subvolume
- Management of groups (both system groups and non-priviledges ones)
- Configure the settings for useradd and similar tools (default group, permissions, etc.)
- Configure the password encryption method

### Management of Systemd Units (services, etc.)

**[TO BE WRITTEN]** See https://fedoramagazine.org/managing-software-and-services-with-cockpit/

### Management of Storage Devices

**[TO BE WRITTEN]** See https://fedoramagazine.org/performing-storage-management-tasks-in-cockpit/

### Mounting NFS and Samba Shares

**[TO BE WRITTEN]**

### Network and Firewalld

**[TO BE WRITTEN]** See https://fedoramagazine.org/managing-network-interfaces-and-firewalld-in-cockpit/

### Kdump Configuration

The `cockpit-kdump` module offers a very limited functionality. It does not offer any mechanism to
enable or disable kernel crash dumping (the user must modify the kernel command line manually or
using a command-line utility), to define the crash limits (usage of external tools is again needed
to estimate sensible values) or to tweak the configuration.

![kdump](https://gist.githubusercontent.com/ancorgs/dbd25f8910cce7ffdff0b3d9cdb3b937/raw/94e61df0b837610a672cf3ea8be08ae0f9a5e4c4/zz_kdump.png)

It simply allows to:

- start/stop the associated service (the service is only useful if kdump was configured previously as explained)
- change the location of the dump file
- intentionally crash the system to check the configuration works

Contrary to `yast2-kdump` (current SLE alternative to configure kdump and related technologies),
Cockpit doesn't seem to have any kind of support for fadump (firmware assisted dump) or to offer any
guidance to the user. The YaST module is able to detect when fadump is available and, of course, is
able to fully configure both fadump and kdump adding the corresponding boot arguments to the kernel
and tweaking any other part of the system that needs to be adjusted. As shown in the following
screenshots, it also offers many more configuration options, each with its corresponding suggested
value (calculated based on the information of the system) and help texts.

![yast2-kdump](https://gist.githubusercontent.com/ancorgs/dbd25f8910cce7ffdff0b3d9cdb3b937/raw/7730963e2c8ccb37308318c6d5156b81438d3fd3/zz_yast2-kdump.png)

### SELinux Management

**[TO BE WRITTEN]**

### Management of Virtual Machines

The `cockpit-machines` extension offers an UI that can be used to define and manage virtual
machines. The equivalent in SLE would be [`virt-manager`](https://virt-manager.org/), which is a GUI
application.

According to this SUSE internal [confluence
page](https://confluence.suse.com/pages/viewpage.action?spaceKey=LEONG&title=ALP+Virtualization),
`cockpit-machines` misses quite some features when compared to `virt-manager`:

 - no video device choice
 - no watchdog choice
 - no TPM
 - can not add a controller (scsi, usb etc...)
 - creation without any template (very limited)
 - no machine type choice
 - no BIOS legacy / UEFI choice
 - add virtio devices is not yet possible ? (only passthrough seems to be available)

One of the good things about `virt-manager` is that it doesn't need to be installed in the same
machine hosting the virtual machines. Actually, an installation of `virt-manager` can be configured
to control several hosts (all of them with their corresponding VMs), accessing those hosts over SSH.
Although a similar effect could be achieved using the [mentioned
capability](#managing-multiple-systems-from-a-single-cockpit-interface) of Cockpit to manage several
hosts in one web interface, that would need `cockpit-bridge`, `cockpit-machines` and their [full
sets of dependencies](#functionality-vs-footprint) installed on each Host OS.

Read more about `cockpit-machines` in these articles:
[[1]](https://fedoramagazine.org/create-virtual-machines-with-cockpit-in-fedora/)
[[2]](https://fedoramagazine.org/reconfiguring-virtual-machines-with-cockpit/).

Take into account that, even if `cockpit-machines` is still quite far away from `virt-manager` in
terms of supported functionality, Red Hat decided to deprecate `virt-manager` in favor of the
Cockpit extension.

### Podman Administration

**[TO BE WRITTEN]**

### Image Builder (Composer)

**[TO BE WRITTEN]**

### RPM-Based System Updates

**[TO BE WRITTEN]** See https://fedoramagazine.org/managing-software-and-services-with-cockpit/

### System Updates with OSTree

**[TO BE WRITTEN]** See https://fedoramagazine.org/managing-software-and-services-with-cockpit/

We will not use OSTree in ALP. Very likely we will use
[tukit](https://github.com/openSUSE/transactional-update) and
[cockpit-tukit](https://github.com/openSUSE/cockpit-tukit).

### Collecting System Debug Information (sos)

**[TO BE WRITTEN]**

### Administration of LDAP Servers

**[TO BE WRITTEN]** See https://fedoramagazine.org/managing-user-accounts-with-cockpit/

## Missing Functionality Compared to SLE Alternatives
 
This section describe functionality provided by tools available at SLE (mainly YaST) and that have
no equivalent (not even a more limited one) at the current Cockpit ecosystem.
 
 **[TO BE WRITTEN]**

# Using Containers for System Management

## Advantages

Having a system management in a separate container has several advantages:

- Decreasing the size and the dependencies of the host system
- The management container might be installed only when needed and can be later
  removed when not needed anymore

## Disadvantages

Unfortunately there are also some disadvantages:

- More complex architecture
- We will need to build, distribute and maintain the containers
  (in addition to the RPM packages)
- It's a question whether all management tasks can be done from a container,
  maybe for some tasks we might need a dedicated service or special support
  in the host system

## New Problems

There are also some completely new problems, specific to the containers environment.

In the current SLE we might be pretty sure that for example in SLE-15-SP3 you use the
SLE-15-SP3 YaST to manage the system. So the YaST expectations about the system
(like file locations, file formats, package names, etc...) match the managed
system.

With containers you might easily run a container which contains a different version
of the system or even a non-SUSE distribution. That's actually the goal of the
containers.

So we need to either ensure the users can only install the matching containers
into the system or the system management container should check the host OS
properties at the very beginning.

The first option depends on how the containers are distributed and started,
in theory the starting script could ensure it starts the right container...

## Implementation Details

Unix/Linux has basically two main abstractions: files and processes, everything
is either a file or a process.

### File Access

Docker containers allow bind-mounting the host file system inside a container
using the
[--volume (-v)](
https://docs.docker.com/engine/reference/commandline/run/#mount-volume--v---read-only)
or [--mount](
https://docs.docker.com/engine/reference/commandline/run/#add-bind-mounts-or-volumes-using-the---mount-flag
) option. By default the access is read-write.

With this you can easily access the host files without any problems, for example
starting a container with the `-v /:/mnt` option and then running `touch /mnt/foo`
will create file `/foo` in the host system.

#### Example

```shell
# the host root / can be accessed in /mnt in the container
docker run -v /:/mnt -it --rm opensuse/leap:15.3 bash
```

#### Transactional Update

If the host OS uses transactional updates (e.g. read-only root file system)
then the things get even more complicated. But that's a separate problem, not
specific to containers. The native management tools would need to be adapted for
that as well...

### Accessing Processes

Accessing the processes running in the host is more difficult. Tools like `ps` or
`kill` will run in the container and cannot affect the processes running in the host.

But you can run some commands using `chroot` command in the mounted host root.
Compare running `ps aux` and `chroot /mnt ps aux` commands in the example container
above. But that does not work if the tool has to really interact with a process,
e.g. `chroot /mnt kill <PID>` does not work as expected.

The `chroot` workaround also requires the tool to be installed in the host system,
so it could be used only for the basic calls present in the minimal system. It
could not be used for the tools included in the container.

For full access we would need to use `ssh` or something similar and access the
host system via network.

In some cases it is possible to communicate with the host system via sockets.
E.g. with the `-v /var/run/docker.sock:/var/run/docker.sock` Docker option
you can use the `docker` command in the container to manage the Docker containers
running in the host. Or you can use sockets for accessing the X Server running
in the host ([example](
https://github.com/yast/yast-widget-demo/blob/master/docker/run.sh#L20-L22
)).

This trick is used by the [Portainer](
https://docs.portainer.io/v/ce-2.11/start/install/server/docker/linux) Docker
management tool which itself runs in a container.

### Privileged Access

Some actions might not be allowed to do from a container for security reasons.
Docker has several options how to enable that, see the [documentation](
https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities
).

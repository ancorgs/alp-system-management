# Work Group Conclussions for July 2022

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

## The Future of the Work Group

Will this work group be in charge of developing and packaging the above solutions for the first
versions of ALP? If not, who is this work group proposing as new maintainers/developers?

## Roadmap for the September Prototype

If this work group is going to take care, we need to have a plan for the prototype.

### Planned Status and Roadmap for Cockpit

* Which extensions/functionality are we going to offer?
* How are we going to ensure the proper theme is always installed?
* Is there any impact expected from the `/usr/etc` + `/etc` layout? If so, how will it be handled?
* What's the plan to properly support both transactional and non-transactional systems?

### Planned Status and Roadmap for Containerized YaST

* Which modules/functionality are we going to offer?
* What's the plan to properly support both transactional and non-transactional systems?

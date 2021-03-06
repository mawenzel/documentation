.. _taupage:

=======
Taupage
=======

**Taupage** is the base AMI allowing dockerized applications to run with STUPS.

As we want to foster immutable (and therefore deterministic and reproducible) deployments, we want to encourage the use
of Docker (and similar deployment technologies). The Taupage AMI is capable of starting a Docker container on boot. This
will enable teams to deploy 'what they want' as long as they package it in a Docker image. The server will be
set up to have an optimal configuration including managed SSH access, audit logging, log collection, monitoring and
reviewed security additions.

---------------------
Using the Taupage AMI
---------------------

There is currently no internal tooling but you can find the Taupage AMIs in your EC2 UI. They are maintained by the
Platform team and regularly updated with the newest security fixes and configuration improvements.

.. NOTE::
   The process of updating the AMI is not established nor discussed yet!

How to configure the AMI
++++++++++++++++++++++++

The Taupage AMI uses the official cloud-init project to receive user configuration. Different to the standard, you can
not use the normal user data mimetypes (no #cloud-config, shell scripts, file uploads, URL lists, ...) but only our own
configuration format::

   #zalando-ami-config

   application_id: my-nginx-test-app
   application_version: "1.0"

   runtime: Docker
   source: dockerfiles/nginx:latest

   ports:
     80: 80
     443: 443

   environment:
     STAGE: production

   capabilities_add:
     - NET_BIND_SERVICE
   capabilities_drop:
     - NET_ADMIN

   root: false

   mounts:
     /var/lib/zookeeper-logs:
       devices:
         - /dev/sdb
       setup: true

     /var/lib/zookeeper-data:
       devices:
         - /dev/sdc
         - /dev/sdd
       raid_mode: 0
       setup: true

   notify_cfn:
     stack: pharos
     resource: WebServerGroup

   ssh_ports:
     - 22

Provide this configuration as your user-data during launch of your EC2 instance.

application_id:
-----------------

**(required)**

The well-known, registered (in :ref:`kio`) application identifier/name. Examples: "order-engine", "eventlog-service", ..

application_version:
--------------------

**(required)**

The well-known, registered (in :ref:`kio`) application version string. Examples: "1.0", "0.1-alpha", ..

runtime:
--------

**(required)**

What kind of deployment artifact you are using. Currently supported:

* Docker

.. NOTE::
   We plan to integrate CoreOS's Rocket as a runtime for experimental use soon.

source:
-------

**(required)**

The source, the configured runtime uses to fetch your delpoyment artifact. For Docker, this is the Docker image.
Usually this will point to a Docker image stored in :ref:`pierone`.

ports:
------

**(optional, default: no ports open)**

A map of all ports that have to be opened from the container. The key is the original port in your container and its
value is the public server port to open.

environment:
------------

**(optional)**

A map of environment variables to set.

capabilities_add:
-----------------

**(optional)**

A list of capabilities to add to the execution (without the CAP_ prefix). See
http://man7.org/linux/man-pages/man7/capabilities.7.html for available capabilities.

capabilities_drop:
------------------

**(optional)**

A list of capabilities to drop of the execution (without the CAP_ prefix). See
http://man7.org/linux/man-pages/man7/capabilities.7.html for available capabilities.

root:
-----

**(optional, default: false)**

Specifies, if the container has to run as root. By default, containers run as an unprivileged user. See the
**capabilities_add** and prefer it always. This is only the last resort.

mounts:
-------

**(optional)**

A map of mount targets and device configurations. A device configuration has **device** to reference the root device
node and a **setup** flag if the device should be partitioned and formatted no boot (of not, the AMI expects and mounts
partition 1 from the device).

notify_cfn:
-----------

**(optional)**

Will send cloud formation the boot result if specified. If you specify it, you have to provide the **stack** name and
the stack **resource** with which this server was booted. This helps cloud formation to know, if starting you server
worked or not (else, it will run into a timeout, waiting for notifications to arrive).

If you would use the example stack
http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/example-templates-autoscaling.html
the resource name would be **WebServerGroup**.

ssh_ports:
----------

**(optional, default: 22)**

List of SSH server ports. This option allows using alternative TCP ports for the OpenSSH server.
This is useful if an application (runtime container) wants to use the default SSH port.

AMI internals
+++++++++++++

This section gives you an overview of customization, the Taupage AMI contains on top of the Ubuntu Cloud Images.

Hardening
---------

TODO

* Kernel grsecurity, PAX?
* Resrictive file permissions (no unused SUID bins etc)
* Unused users and groups removed
* Unused daemons disabled
* Zalando CA preinstalled
* Weak crypto algorithms disabled (SSH)
* Unused packages removed
* No passwords for users
* iptables preconfigured with only specified ports + ssh open
* hardened network settings (sysctl)
* disabled IPv6 (not possible in AWS anyways)

Auditing & Logs
---------------

TODO

* auditd logs all access
* all logs, including application logs (docker logs) are streamed to central logging service and rotated

Managed SSH access
------------------

SSH access is managed with the SSH access granting service. The AMI is set up to have automatic integration. Your
SSH key pair choice on AWS will be ignored - temporary access can only be gained via the granting service. All user
actions are logged for auditing reasons.

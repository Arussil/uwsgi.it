uwsgi.it
========

The next-generation Unbit hosting platform

Intro
-----

contrary to the current unbit.it hosting platform, the next generation one will be:

- fully open source (currently at least 60% of the unbit.it kernel patches are not released to the public)
- can be installed on vanilla kernels
- everyone can build it on his/her systems (and eventually buy commercial support from unbit.com ;)
- will not rely on apache (so .htaccess will not be supported, unless you install apache in your container and proxy it via uWSGI routing)

Goals
-----

- each customer has a pool of containers
- each container has an associated disk quota, a cpu share and a fixed amount of memory
- each container has an associated Emperor
- best possible isolation between containers
- each container can be mapped to a different distribution (both 32 and 64 bit)
- each container has its dedicated firewall based on the tuntap router plugin
- ssh access is governed by the container emperor using the pam-unbit project
- uid/gid mapping is managed using nss-unbit project
- each container runs with its own uid/gid
- each container has its own /etc/hosts, /etc/hostname and /etc/resolv.conf
- each vassal in the container subscribes to a central http router with a specific key (domain)
- containers' Emperor by default configure alarms for: disk quota, oom, memory thresholds, restarts
- gather metrics and generate graphs
- SNI is the only https/spdy supported approach
- cron and external processes (like dbs) are managed as vassals
- supported languages will be Perl, CPython, PyPy, Ruby, Lua and php (yes php apps works even without .htaccess...) 
- Websockets support (in the routers/proxy) is enabled by default
- Simple clustering and load-balancing
- Sending emails is not part of the infrastructure (read:no SMTP services), but each container has transparent support for the nullmailer spool service (so you can use it to asynchronously send mails to external smtp services like mandrill and sendgrid)


Status
------

Currently we are in pre-beta status, the api is almost complete, while the example web interface is still in early stage of development.

Beta phase will officially start soon after the installation docs are complete.


How it works
------------

On server startup the emperor.ini is run. This Emperor starts 4 services:

an http/https router

a fastrouter

a legion manager

a containers manager

The 4th service manage vassals in /etc/uwsgi/vassals

Each vassal is spawned in a new Linux namespace and cgroup (all is native, no lxc is involved)

An external app (well it could be hosted on the same infrastructure too) serves the api (a django app)

All the customers vassals are created by the api.

Each vassal spawns a sub-Emperor with uid and gid > 30000, the user (the owner of the container) can now use
this sub-Emperor to spawn its vassals.

The user can enter the container via ssh (a pam module calls setns() to attach to the running container)

The user can only view (and access) processes generated by the sub-Emperor (even the sub-Emperor is hidden)

Domains to containers mapping is done via the uWSGI secured subscription system. An RSA key pair is generated by the control webapp (a Django app) for each registered customer. This key is used by the user to subscribe to the http/https router

Subscriptions can pass SSL certificates to the router that reconfigure it to map them to domains (via SNI)

Three perl processes manage the infrastruture configuration:

- configurator.pl -> manage containers vassal files
- dominator.pl -> manage domains to rsa key mappings
- collector.pl -> gather statistics from the various exposed metrics


A web interface written in perl/Dancer will integrate with the api to give users/customer a user-friendly tool


TODO
----

- fastrouter-only implementation for nginx integration
- is SPDY support whorty ?
- Are CGIs still of interest ?


LONG TERM GOALS
---------------

- FreeBSD jail support in addition to Linux namespaces ?

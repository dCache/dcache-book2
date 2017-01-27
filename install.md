Installing DCACHE
=================

**Table of Contents**

* [Installing a dCache instance] (#installing-a-dcache-instance)  

      * [Prerequisites] (#prerequisites)  
      * [Installation of the dCache Software] (#installation-of-the-dCache-software)  
      * [Readying the PostgreSQL server for the use with dCache] (#readying-the-postgresql-server-for-the-use-with-dcache)  
      * [Configuring Chimera] (#configuring-chimera)  
      * [Configuring dCache] (#configuring_dcache)  
      * [Installing dCache on several nodes] (#installing-dcache-on-several-nodes)  


* [Securiting your dCache installation]  (#securiting-your-dcache-installation)
* [Upgrading a dCache Instance]  (#upgrading-a-dcache-instance)

The first section describes the installation of a fresh DCACHE instance using RPM files downloaded from [the DCACHE home-page]. It is followed by a guide to upgrading an existing installation. In both cases we assume standard requirements of a small to medium sized DCACHE instance without an attached [tertiary storage system] *(https://www.dcache.org/manuals/Book-2.16/reference/rf-glossary-fhs.shtml#gl-tss). The third section contains some pointers on extended features.

Installing a DCACHE instance
============================

In the following the installation of a DCACHE instance will be described. The CHIMERA name space provider, some management components, and the SRM need a PSQL server installed. We recommend running this PSQL on the local node. The first section describes the configuration of a PSQL server. After that the installation of CHIMERA and of the DCACHE components will follow. During the whole installation process root access is required.

Prerequisites
-------------

In order to install DCACHE the following requirements must be met:

-   An RPM-based Linux distribution is required for the following procedure. For Debian derived systems we provide Debian packages and for Solaris the Solaris packages or the tarball.

-   DCACHE requires Java 8 JRE. Please use the latest patch-level and check for upgrades frequently. It is recommended to use JDK as DCACHE scripts can make use of some extra features that JDK provides to gather more diagnostic information (heap-dump, etc). This helps when tracking down bugs.

-   PSQL must be installed and running. We recommend the use of PSQL version 9.2 (at least PSQL version 8.3 is required).

    > **Important**
    >
    > For good performance it is necessary to maintain and tune your PSQL server. There are several good books on this topic, one of which is [PostgreSQL 9.0 High Performance].

Installation of the DCACHE Software
-----------------------------------

The RPM packages may be installed right away, for example using the command:

    PROMPT-ROOT rpm -ivh dcache-DCACHE-PACKAGE-VERSION.noarch.rpm

The actual sources lie at []. To install for example Version DCACHE-PACKAGE-VERSION you would use this:

    PROMPT-ROOT rpm -ivh http://www.dcache.org/downloads/1.9/repo/DCACHE-VERSION/dcache-DCACHE-PACKAGE-VERSION.noarch.rpm

The client can be found in the download-section of the above url, too.

Readying the PSQL server for the use with DCACHE
------------------------------------------------

Using a PSQL server with DCACHE places a number of requirements on the database. You must configure PSQL for use by DCACHE and create the necessary PSQL user accounts and database structure. This section describes how to do this.

### Starting PSQL

Install the PSQL server with the tools of the operating system.

Initialize the database directory (for PSQL version 9.2 this is `/var/lib/pgsql/9.2/data/`) , start the database server, and make sure that it is started at system start-up.

    PROMPT-ROOT service postgresql-9.2 initdb
    Initializing database:                                     [  OK  ]
    PROMPT-ROOT service postgresql-9.2 start
    Starting postgresql-9.2 service:                           [  OK  ]
    PROMPT-ROOT chkconfig postgresql-9.2 on

### Enabling local trust

Perhaps the simplest configuration is to allow password-less access to the database and the following documentation assumes this is so.

To allow local users to access PSQL without requiring a password, ensure the file `pg_hba.conf`, which (for PSQL version 9.2) is located in `/var/lib/pgsql/9.2/data`, contains the following lines.

    # TYPE  DATABASE        USER            ADDRESS                 METHOD

    # "local" is for Unix domain socket connections only
    local   all             all                                     trust
    # IPv4 local connections:
    host    all             all             127.0.0.1/32            trust
    # IPv6 local connections:
    host    all             all             ::1/128                 trust

> **Note**
>
> Please note it is also possible to run DCACHE with all PSQL accounts requiring passwords. See [???] for more advice on the configuration of PSQL.

> **Important**
>
> If you have edited PSQL configuration files, you *must* restart PSQL for those changes to take effect. On many systems, this can be done with the following command:
>
>     PROMPT-ROOT service postgresql-9.2 restart
>     Stopping postgresql-9.2 service:                           [  OK  ]
>     Starting postgresql-9.2 service:                           [  OK  ]

Configuring CHIMERA
-------------------

CHIMERA is a library providing a hierarchical name space with associated meta data. Where pools in DCACHE store the content of files, CHIMERA stores the names and meta data of those files. CHIMERA itself stores the data in a relational database. We will use PSQL in this tutorial. The properties of CHIMERA are defined in `PATH-ODS-USD/defaults/chimera.properties`. See [???][1] for more information.

### Creating users and databases for DCACHE

Create the Chimera database and user.

    PROMPT-ROOT createdb -U postgres chimera
    CREATE DATABASE
    PROMPT-ROOT createuser -U postgres --no-superuser --no-createrole --createdb --pwprompt chimera
    Enter password for new role:
    Enter it again:
    You do not need to enter a password.

The DCACHE components will access the database server with the user srmdcache.

    PROMPT-ROOT createuser -U postgres --no-superuser --no-createrole --createdb --pwprompt srmdcache
    Enter password for new role:
    Enter it again:
    You do not need to enter a password.

Several management components running on the head node as well as the SRM will use the database dcache for storing their state information:

    PROMPT-ROOT createdb -U srmdcache dcache

There might be several of these on several hosts. Each is used by the DCACHE components running on the respective host.

Create the database used for the billing plots.

    PROMPT-ROOT createdb -O srmdcache -U postgres billing

And run the command `dcache database update`.

    PROMPT-ROOT dcache database update
    PnfsManager@dCacheDomain:
    INFO  - Successfully acquired change log lock
    INFO  - Creating database history table with name: databasechangelog
    INFO  - Reading from databasechangelog
    many more like this...
          

Now the configuration of CHIMERA is done.

Before the first start of DCACHE replace the file `PATH-ODE-ED/gplazma.conf` with an empty file.

    PROMPT-ROOT mv PATH-ODE-ED/gplazma.conf PATH-ODE-ED/gplazma.conf.bak
    PROMPT-ROOT touch PATH-ODE-ED/gplazma.conf

DCACHE can be started now.

    PROMPT-ROOT PATH-ODB-N-Sdcache start
    Starting dCacheDomain done

So far, no configuration of DCACHE is done, so only the predefined domain is started.

Configuring DCACHE
------------------

### Terminology

DCACHE consists of one or more domains. A domain in DCACHE is a Java Virtual Machine hosting one or more DCACHE cells. Each domain must have a name which is unique throughout the DCACHE instance and a cell must have a unique name within the domain hosting the cell.

A service is an abstraction used in the DCACHE configuration to describe atomic units to add to a domain. It is typically implemented through one or more cells. DCACHE keeps lists of the domains and the services that are to be run within these domains in the layout files. The layout file may contain domain- and service- specific configuration values. A pool is a cell providing physical data storage services.

### Configuration files

In the setup of DCACHE, there are three main places for configuration files:

-   PATH-ODS-USD/defaults
-   PATH-ODE-ED/dcache.conf
-   PATH-ODE-ED/layouts

The folder `PATH-ODS-USD/defaults` contains the default settings of the DCACHE. If one of the default configuration values needs to be changed, copy the default setting of this value from one of the files in `PATH-ODS-USD/defaults` to the file `PATH-ODE-ED/dcache.conf`, which initially is empty and update the value.

> **Note**
>
> In this first installation of DCACHE your DCACHE will not be connected to a tape sytem. Therefore please change the values for `pnfsmanager.default-retention-policy` and `pnfsmanager.default-access-latency` in the file `PATH-ODE-ED/dcache.conf`.
>
>     pnfsmanager.default-retention-policy=REPLICA
>     pnfsmanager.default-access-latency=ONLINE

Layouts describe which domains to run on a host and which services to run in each domain. For the customized configuration of your DCACHE you will have to create a layout file in `PATH-ODE-ED/layouts`. In this tutorial we will call it the `mylayout.conf` file.

> **Important**
>
> Do not update configuration values in the files in the defaults folder, since changes to these files will be overwritten by updates.

As the files in `PATH-ODS-USD/defaults/` do serve as succinct documentation for all available configuration parameters and their default values it is quite useful to have a look at them.

### Defining domains and services

Domains and services are defined in the layout files. Depending on your site, you may have requirements upon the doors that you want to configure and domains within which you want to organise them.

A domain must be defined if services are to run in that domain. Services will be started in the order in which they are defined.

Every domain is a Java Virtual Machine that can be started and stopped separately. You might want to define several domains for the different services depending on the necessity of restarting the services separately.

The layout files define which domains to start and which services to put in which domain. Configuration can be done per domain and per service.

A name in square brackets, *without* a forward-slash (`/`) defines a domain. A name in square brackets *with* a forward slash defines a service that is to run in a domain. Lines starting with a hash-symbol (`#`) are comments and will be ignored by DCACHE.

There may be several layout files in the layout directory, but only one of them is read by DCACHE when starting up. By default it is the `single.conf`. If the DCACHE should be started with another layout file you will have to make this configuration in `PATH-ODE-ED/dcache.conf`.

    dcache.layout=mylayout

This entry in
PATH-ODE-ED/dcache.conf
will instruct DCACHE to read the layout file
PATH-ODE-ED/layouts/mylayout.conf
when starting up.

These are the first lines of `PATH-ODE-ED/layouts/single.conf`:

    dcache.broker.scheme=none

    [dCacheDomain]
    [dCacheDomain/admin]
    [dCacheDomain/poolmanager]

`[DOMAIN-DCACHE]` defines a domain called DOMAIN-DCACHE. In this example only one domain is defined. All the services are running in that domain. Therefore no messagebroker is needed, which is the meaning of the entry `messageBroker=none`.

`[DOMAIN-DCACHE/CELL-ADMIN]` declares that the CELL-ADMIN service is to be run in the DOMAIN-DCACHE domain.

This is an example for the `mylayout.conf` file of a single node DCACHE with several domains.

    [dCacheDomain]
    [dCacheDomain/topo]
    [dCacheDomain/info]

    [namespaceDomain]
    [namespaceDomain/pnfsmanager]
    [namespaceDomain/cleaner]
    [namespaceDomain/dir]

    [poolmanagerDomain]
    [poolmanagerDomain/poolmanager]

    [adminDoorDomain]
    [adminDoorDomain/admin]

    [httpdDomain]
    [httpdDomain/httpd]
    [httpdDomain/billing]

    [gPlazmaDomain]
    [gPlazmaDomain/gplazma]

> **Note**
>
> If you defined more than one domain, a messagebroker is needed, because the defined domains need to be able to communicate with each other. This means that if you use the file `single.conf` as a template for a DCACHE with more than one domain you need to delete the line `messageBroker=none`. Then the default value will be used which is `messageBroker=cells`, as defined in the defaults `PATH-ODS-USD/defaults/dcache.properties`.

### Creating and configuring pools

DCACHE will need to write the files it keeps in pools. These pools are defined as services within DCACHE. Hence, they are added to the layout file of your DCACHE instance, like all other services.

The best way to create a pool, is to use the `dcache` script and restart the domain the pool runs in. The pool will be added to your layout file.

    [domainname/pool]
    name=poolname
    path=/path/to/pool
    pool.wait-for-files=${path}/data

The property `pool.wait-for-files` instructs the pool not to start up until the specified file or directory is available. This prevents problems should the underlying storage be unavailable (e.g., if a RAID device is offline).

> **Note**
>
> Please restart DCACHE if your pool is created in a domain that did not exist before.

    PROMPT-ROOT PATH-ODB-N-Sdcache pool create /srv/dcache/p1 pool1 poolDomain
    Created a pool in /srv/dcache/p1. The pool was added to poolDomain in
    file:/etc/dcache/layouts/mylayout.conf.

In this example we create a pool called pool1 in the directory `/srv/dcache/p1`. The created pool will be running in the domain `poolDomain`.

> **Note**
>
> The default gap for poolsizes is 4GiB. This means you should make a bigger pool than 4GiB otherwise you would have to change this gap in the DCACHE admin tool. See the example below. See also [???][2].
>
>     DC-PROMPT-LOCAL cd poolname
>     DC-PROMPT-POOL set gap 2G
>     DC-PROMPT-POOL save

Adding a pool to a configuration does not modify the pool or the data in it and can thus safely be undone or repeated.

### Starting DCACHE

Restart DCACHE to start the newly configured components `PATH-ODB-N-Sdcache restart` and check the status of DCACHE with `PATH-ODB-N-Sdcache status`.

    PROMPT-ROOT PATH-ODB-N-Sdcache restart
    Stopping dCacheDomain 0 1 done
    Starting dCacheDomain done
    Starting namespaceDomain done
    Starting poolmanagerDomain done
    Starting adminDoorDomain done
    Starting httpdDomain done
    Starting gPlazmaDomain done
    Starting poolDomain done
    PROMPT-ROOT PATH-ODB-N-Sdcache status
    DOMAIN            STATUS  PID   USER
    dCacheDomain      running 17466 dcache
    namespaceDomain   running 17522 dcache
    poolmanagerDomain running 17575 dcache
    adminDoorDomain   running 17625 dcache
    httpdDomain       running 17682 dcache
    gPlazmaDomain     running 17744 dcache
    poolDomain        running 17798 dcache

Now you can have a look at your DCACHE via The Web Interface, see [???][3]: <http://:2288/>, where httpd.example.org is the node on which your CELL-HTTPD service is running. For a single node DCACHE this is the machine on which your DCACHE is running.

### JAVA heap size

By default the JAVA heap size and the maximum direct buffer size are defined as

    dcache.java.memory.heap=512m
    dcache.java.memory.direct=512m

Again, these values can be changed in `PATH-ODE-ED/dcache.conf`.

For optimization of your DCACHE you can define the JAVA heap size in the layout file separately for every domain.

    [dCacheDomain]
    dcache.java.memory.heap=2048m
    dcache.java.memory.direct=0m
    ...
    [utilityDomain]
    dcache.java.memory.heap=384m
    dcache.java.memory.direct=16m

> **Note**
>
> DCACHE uses JAVA to parse the configuration files and will search for JAVA on the system path first; if it is found there, no further action is needed. If JAVA is not on the system path, the environment variable JAVA\_HOME defines the location of the JAVA installation directory. Alternatively, the environment variable JAVA can be used to point to the JAVA executable directly.
>
> If JAVA\_HOME or JAVA cannot be defined as global environment variables in the operating system, then they can be defined in either `/etc/default/dcache` or `/etc/dcache.env`. These two files are sourced by the init script and allow JAVA\_HOME, JAVA and DCACHE\_HOME to be defined.

Installing DCACHE on several nodes
----------------------------------

Installing DCACHE on several nodes is not much more complicated than installing it on a single node. Think about how DCACHE should be organised regarding services and domains. Then adapt the layout files, as described in [section\_title], to the layout that you have in mind. The files `PATH-ODE-ED/layouts/head.conf` and `PATH-ODE-ED/layouts/pool.conf` contain examples for a DCACHE head-node and a DCACHE pool respectively.

> **Important**
>
> You must configure a domain called DOMAIN-DCACHE but the other domain names can be chosen freely.
>
> Please make sure that the domain names that you choose are unique. Having the same domain names in different layout files on different nodes may result in an error.

On any other nodes than the head node, the property `dcache.broker.host` has to be added to the file `PATH-ODE-ED/dcache.conf`. This property should point to the host containing the special domain DOMAIN-DCACHE, because that domain acts implicitly as a broker.

> **Tip**
>
> On DCACHE nodes running only pool services you do not need to install PSQL. If your current node hosts only these services, the installation of PSQL can be skipped.

Securiting your dCache installation
===================================

DCACHE uses the LocationManager to discover the network topology of the internal communication: to which domains this domain should connect. The domain contacts a specific host and queries the information using UDP port `11111`. The response describes how the domain should react: whether it should allow incoming connections and whether it should contact any other domains.

Once the topology is understood, DCACHE domains connect to each other to build a network topology. Messages will flow over this topology, enabling the distributed system to function correctly. By default, these connections use TCP port `11111`.

It is essential that both UDP and TCP port `11111` are firewalled and that only other nodes within the DCACHE cluster are allowed access to these ports. Failure to do so can result in remote users running arbitrary commands on any node within the dCache cluster.

Upgrading a DCACHE Instance
===========================

> **Important**
>
> Always read the release notes carefully before upgrading!

Upgrading to bugfix releases within one supported branch (e.g. from DCACHE-PATCH-VERSION to DCACHE-NEXT-PATCH-VERSION) may be done by upgrading the packages with

    PROMPT-ROOT rpm -Uvh packageName

Now DCACHE needs to be started again.

  [the DCACHE home-page]: http://www.dcache.org
  [PostgreSQL 9.0 High Performance]: http://www.2ndquadrant.com/books/postgresql-9-0-high-performance
  []: http://www.dcache.org/downloads/IAgree.shtml
  [???]: #cb-postgres-configure
  [1]: #cf-chimera
  [2]: #intouch-admin
  [3]: #intouch-web
  [section\_title]: #in-install-layout

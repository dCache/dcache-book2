[comment]: <> (for DCACHE-VERSION-series (LAYOUT-CURRENT layout))
Preface
=======

Welcome to the DCACHE. DCACHE is a distributed storage solution for storing huge amounts of data without a hard limit, used to provide storage in the petabyte range. Therefore it qualifies as the storage system supporting data intensive experiments.

DCACHE is a joined effort between the Deutsches Elektronen-Synchrotron (DESY) in Hamburg, Nordic Data Grid Facility (NDGF based in Copenhagen), the Fermi National Accelerator Laboratory near Chicago with significant distributions and support from the University of California, San Diego, INFN, Bari as well as Rutherford Appleton Laboratory, UK and CERN in Geneva.

DCACHE can use hierarchical storage management (e.g., hard disk and tape), provides mechanisms to automatically increase performance and balance loads, increase resilience and availability. It also supplies advanced control systems to manage data as well as data flows. Normal filesystem (btrfs, ext4, XFS, ZFS) is used to store data on storage nodes.

There are several ways of accessing data stored in DCACHE:

-   NFS v4.1 (CHIMERA)

-   HTTP and WEBDAV

-   GRIDFTP (GSIFTP)

-   xrootd

-   SRM (versions 1.1 and 2.2)

-   DCAP and GSIDCAP

DCACHE supports certificate based authentication through the Grid Security Infrastructure used in GSIFTP, GSIDCAP transfer protocols and the SRM management protocol. Certificate authentication is also available for HTTP and WEBDAV.

DCACHE also supports fine-grain authorization with support for POSIX file permissions and NFS-style access control lists.

Other features of DCACHE are:

-   Resilience and high availability can be implemented in different ways by having multiple replicas of the same files.

-   Easy migration of data via the migration module.

-   A powerful cost calculation system that allows to control the data flow (reading and writing from/to pools, between pools and also between pools and tape).

-   Load balancing and performance tuning by hot pool replication (via cost calculation and replicas created by pool-to-pool-transfers).

-   Space management and support for space tokens.

-   Garbage collection of replicas, depending on their flags, age, et cetera.

-   Detailed logging and debugging as well as accounting and statistics.

-   XML information provider with detailed live information about the cluster.

-   Scriptable adminstration interface with a terminal-based front-end.

-   Web-interface with live information of the most important information.

-   Ensuring data integrity through checksumming.

DCACHE / SRM can transparently manage data distributed among dozens of disk storage nodes (sometimes distributed over several countries). The system has shown to significantly improve the efficiency of connected tape storage systems, by caching, gather and flush and scheduled staging techniques. Furthermore, it optimizes the throughput to and from data clients by dynamically replicating datasets on the detection of load hot spots. The system is tolerant against failures of its data servers, which allows administrators to deploy commodity disk storage components.

Access to the data is provided by various standard protocols. Furthermore the software comes with an implementation of the Storage Resource Manager protocol (SRM), which is an open standard for grid middleware to communicate with site specific storage fabrics.

Who should read this book?
--------------------------

This book is primerally targeted at system administrators.

Minimum System Requirements?
----------------------------

For minimal test installation:

-   Hardware: contemporary CPU , 1 GiB of RAM , 100 MiB free harddisk space

-   Software: Oracle/Sun Java, Postgres SQL Server

For a high performance Grid scenario the hardware requirements highly differ, which makes it impossible to provide such parameters here. However, if you wish to setup a DCACHE-based storage system, just let us know and we will help you with your system specifications. Just contact us: <support@dcache.org>.

What is inside?
---------------

This book shall introduce you to DCACHE and provide you with the details of the installation. It describes configuration, customization of DCACHE as well as the usage of several protocols that DCACHE supports. Additionally, it provides cookbooks for standard tasks.

Here is an overview part by part:

Part 1, Getting started: This part introduces you to the cells and domain concept in DCACHE. It provides a detailed description of installing, the basic configuration, and upgrading DCACHE.

Part 2, Configuration of dCache: Within this part the configuration of several additional features of DCACHE is described. They are not necessary to run DCACHE but will be needed by some users depending on their requirements.

Part 3, Cookbook: This part comprises guides for specific tasks a system administrator might want to perform.

Looking for help?
-----------------

This part gets you all the help that you might need:

-   For acquiring resources:

    -   The [download page].

    -   The [YUM repositories].

-   For getting help during installation:

    -   Developers <support@dcache.org>

    -   Additional Support:

        -   German support:<german-support@dcache.org>

        -   UK support:<GRIDPP-STORAGE@JISCMAIL.AC.UK>

        -   USA support:<osg-storage@opensciencegrid.org>

        -   User Forum: <user-forum@dcache.org>

-   For features that you would like to see in DCACHE or bugs that should be fixed: Just write an e-mail to <support@dcache.org>

-   If you like to stay up-to-date about new releases you can use the RSS feeds available from [our downloads page][download page].

  [download page]: http://www.dcache.org/downloads
  [YUM repositories]: http://trac.dcache.org/projects/dcache/wiki/manuals/Yum

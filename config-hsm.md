Chapter 8: The DCACHE Tertiary Storage System Interface
============================================


Table of Contents

* [Introduction](#introduction)
* [Scope of this chapter](#scope-of-this-chapter)
* [Requirements for a Tertiary Storage System](#requirements-for-a-tertiary-storage-system)

    [Migrating Tertiary Storage Systems with a file system interface.](#
    [Tertiary Storage Systems with a minimalistic PUT, GET and REMOVE interface](#

* [IHow dCache interacts with a Tertiary Storage System](#
* [IDetails on the TSS-support executable](#

     [Summary of command line options](#
     [Summary of return codes](#
     [The executable and the STORE FILE operation](#
     [The executable and the FETCH FILE operation](#
     [The executable and the REMOVE FILE operation](#

* [IConfiguring pools to interact with a Tertiary Storage System](#

     [The dCache layout files](#
     [What happens next](#

* [IHow to Store-/Restore files via the Admin Interface](#
* [IHow to monitor what’s going on](#

     [Log Files](#
     [Obtain information via the dCache Command Line Admin Interface](#

* [Example of an executable to simulate a tape backend](#


Introduction
============

One of the features DCACHE provides is the ability to migrate files from its disk repository to one or more connected Tertiary Storage Systems (TSS) and to move them back to disk when necessary. Although the interface between DCACHE and the TSS is kept simple, DCACHE assumes to interact with an intelligent TSS. DCACHE does not drive tape robots or tape drives by itself. More detailed requirements to the storage system are described in one of the subsequent paragraphs.

Scope of this chapter
=====================

This document describes how to enable a standard DCACHE installation to interact with a Tertiary Storage System. In this description we assume that

-   every DCACHE disk pool is connected to only one TSS instance.
-   all DCACHE disk pools are connected to the same TSS instance.
-   the DCACHE instance has not yet been populated with data, or only with a negligible amount of files.

In general, not all pools need to be configured to interact with the same Tertiary Storage System or with a storage system at all. Furthermore pools can be configured to have more than one Tertiary Storage System attached, but all those cases are not in the scope of the document.

Requirements for a Tertiary Storage System
==========================================

DCACHE can only drive intelligent Tertiary Storage Systems. This essentially means that tape robot and tape drive operations must be done by the TSS itself and that there is some simple way to abstract the file PUT, GET and REMOVE operation.

Migrating Tertiary Storage Systems with a file system interface.
----------------------------------------------------------------

Most migrating storage systems provide a regular POSIX file system interface. Based on rules, data is migrated from primary to tertiary storage (mostly tape systems). Examples for migrating storage systems are:

-   HPSS
    (High Performance Storage System)
-   DMF
    (Data Migration Facility)

Tertiary Storage Systems with a minimalistic PUT, GET and REMOVE interface
--------------------------------------------------------------------------

Some tape systems provide a simple PUT, GET, REMOVE interface. Typically, a copy-like application writes a disk file into the TSS and returns an identifier which uniquely identifies the written file within the Tertiary Storage System. The identifier is sufficient to get the file back to disk or to remove the file from the TSS. Examples are:

-   OSM
    (Object Storage Manager)
-   Enstore
    (FERMIlab)

How DCACHE interacts with a Tertiary Storage System
===================================================

Whenever DCACHE decides to copy a file from disk to tertiary storage a user-provided [EXECUTABLE] which can be either a script or a binary is automatically started on the pool where the file is located. That EXECUTABLE is expected to write the file into the Backend Storage System and to return a URI, uniquely identifying the file within that storage system. The format of the URI as well as the arguments to the EXECUTABLE, are described later in this document. The unique part of the URI can either be provided by the storage element, in return of the STOREFILE operation, or can be taken from DCACHE. A non-error return code from the EXECUTABLE lets DCACHE assume that the file has been successfully stored and, depending on the properties of the file, DCACHE can decide to remove the disk copy if space is running short on that pool. On a non-zero return from the EXECUTABLE, the file doesn't change its state and the operation is retried or an error flag is set on the file, depending on the error [return code] from the EXECUTABLE.

If DCACHE needs to restore a file to disk the same EXECUTABLE is launched with a different set of arguments, including the URI, provided when the file was written to tape. It is in the responsibility of the EXECUTABLE to fetch the file back from tape based on the provided URI and to return `0` if the FETCHFILE operation was successful or non-zero otherwise. In case of a failure the pool retries the operation or DCACHE decides to fetch the file from tape using a different pool.

Details on the TSS-support EXECUTABLE
=====================================

Summary of command line options
-------------------------------

This part explains the syntax of calling the EXECUTABLE that supports STOREFILE, `FETCH
	FILE` and REMOVEFILE operations.

put
pnfsID
filename
-si=
storage-information
other-options
get
pnfsID
filename
-si=
storage-information
-uri=
storage-uri
other-options
remove
-uri=
storage-uri
other-options
-   put
    /
    get
    /
    remove
    : these keywords indicate the operation to be performed.
    -   put
        : copy file from disk to TSS.
    -   get
        : copy file back from TSS to disk.
    -   remove
        : remove the file from TSS.
-   pnfsID
    : The internal identifier (i-node) of the file within DCACHE. The
    pnfsID
    is unique within a single DCACHE instance and globally unique with a very high probability.
-   filename
    : is the full path of the local file to be copied to the TSS (for
    put
    ) and respectively into which the file from the TSS should be copied (for
    get
    ).
-   storage-information
    : the storage information of the file, as explained
    below
    .
-   storage-uri
    : the URI, which was returned by the EXECUTABLE, after the file was written to tertiary storage. In order to get the file back from the TSS the information of the URI is preferred over the information in the
    storage-information
    .
-   other-options
    : -
    key
    =
    value
    pairs taken from the TSS configuration commands of the pool 'setup' file. One of the options, always provided is the option -command=
    full path of this EXECUTABLE
    .

### Storage Information

The storage-information is a string in the format

    -si=size=bytes;new=true/false;stored=true/false;sClass=StorageClass;\
    cClass0CacheClass;hsm=StorageType;key=value;[key=value;[...]]

    -si=size=1048576000;new=true;stored=false;sClass=desy:cms-sc3;cClass=-;hsm=osm;Host=desy;

-   size
    : Size of the file in bytes
-   new
    : FALSE if file already in the DCACHE; TRUE otherwise
-   stored
    : TRUE if file already stored in the TSS; FALSE otherwise
-   sClass
    : HSM depended, is used by the POOLMNGR for pool attraction.
-   cClass
    : Parent directory tag (cacheClass). Used by the POOLMNGR for pool attraction. May be '-'.
-   hsm
    : Storage manager name (enstore/osm). Can be overwritten by the parent directory tag (hsmType).

<!-- -->

-   group
    : The storage group of the file to be stored as specified in the ".(tag)(sGroup)" tag of the parent directory of the file to be stored.
-   store
    : The store name of the file to be stored as specified in the ".(tag)(OSMTemplate)" tag of the parent directory of the file to be stored.
-   bfid
    : Bitfile ID (get and remove only) (e.g. 000451243.2542452542.25424524)

<!-- -->

-   group
    : The storage group (e.g. cdf, cms ...)
-   family
    : The file family (e.g. sgi2test, h6nxl8, ...)
-   bfid
    : Bitfile ID (get only) (e.g. B0MS105746894100000)
-   volume
    : Tape Volume (get only) (e.g. IA6912)
-   location
    : Location on tape (get only) (e.g. : 0000\_000000000\_0000117)

There might be more key values pairs which are used by the DCACHE internally and which should not affect the behaviour of the EXECUTABLE.

### Storage URI

The storage-uri is formatted as follows:

    hsmType://hsmInstance/?store=storename&group=groupname&bfid=bfid

-   hsmType
    : The type of the Tertiary Storage System
-   hsmInstance
    : The name of the instance
-   storename
    and
    groupname
    : The store and group name of the file as provided by the arguments to this EXECUTABLE.
-   bfid
    : The unique identifier needed to restore or remove the file if necessary.

A storage-uri:

    osm://osm/?store=sql&group=chimera&bfid=3434.0.994.1188400818542

Summary of return codes
-----------------------

| Return Code         | Meaning                 | Behaviour for PUT FILE | Behaviour for GET FILE                             |
|---------------------|-------------------------|------------------------|----------------------------------------------------|
| 30 &lt;= rc &lt; 40 | User defined            | Deactivates request    | Reports problem to POOLMNGR                   |
| 41                  | No space left on device | Pool retries           | Disables pool and reports problem to POOLMNGR |
| 42                  | Disk read I/O error     | Pool retries           | Disables pool and reports problem to POOLMNGR |
| 43                  | Disk write I/O error    | Pool retries           | Disables pool and reports problem to POOLMNGR |
| other               | -                       | Pool retries           | Reports problem to POOLMNGR                   |

The EXECUTABLE and the STOREFILE operation
------------------------------------------

Whenever a disk file needs to be copied to a Tertiary Storage System DCACHE automatically launches an EXECUTABLE on the pool containing the file to be copied. Exactly one instance of the EXECUTABLE is started for each file. Multiple instances of the EXECUTABLE may run concurrently for different files. The maximum number of concurrent instances of the `EXECUTABLEs` per pool as well as the full path of the EXECUTABLE can be configured in the 'setup' file of the pool as described in [section\_title].

The following arguments are given to the EXECUTABLE of a STOREFILE operation on startup. `put` pnfsID filename -si=storage-information more options Details on the meaning of certain arguments are described in [section\_title][1].

With the arguments provided the EXECUTABLE is supposed to copy the file into the Tertiary Storage System. The EXECUTABLE must not terminate before the transfer of the file was either successful or failed.

Success must be indicated by a `0` return of the EXECUTABLE. All non-zero values are interpreted as a failure which means, DCACHE assumes that the file has not been copied to tape.

In case of a `0` return code the EXECUTABLE has to return a valid storage URI to DCACHE in formate:

    hsmType://hsmInstance/?store=storename&group=groupname&bfid=bfid

Details on the meaning of certain parameters are described [above].

The bfid can either be provided by the TSS as result of the STOREFILE operation or the `pnfsID` may be used. The latter assumes that the file has to be stored with exactly that `pnfsID` within the TSS. Whatever URI is chosen, it must allow to uniquely identify the file within the Tertiary Storage System.

> **Note**
>
> Only the URI must be printed to stdout by the EXECUTABLE. Additional information printed either before or after the URI will result in an error. stderr can be used for additional debug information or error messages.

The EXECUTABLE and the FETCHFILE operation
------------------------------------------

Whenever a disk file needs to be restored from a Tertiary Storage System DCACHE automatically launches an EXECUTABLE on the pool containing the file to be copied. Exactly one instance of the EXECUTABLE is started for each file. Multiple instances of the EXECUTABLE may run concurrently for different files. The maximum number of concurrent instances of the EXECUTABLE per pool as well as the full path of the EXECUTABLE can be configured in the 'setup' file of the pool as described in [section\_title].

The following arguments are given to the EXECUTABLE of a FETCHFILE operation on startup: `get` pnfsID filename -si=storage-information -uri=storage-uri more options Details on the meaning of certain arguments are described in [section\_title][1]. For return codes see [section\_title][return code].

The EXECUTABLE and the REMOVEFILE operation
-------------------------------------------

Whenever a file is removed from the DCACHE namespace (file system) a process inside DCACHE makes sure that all copies of the file are removed from all internal and external media. The pool which is connected to the TSS which stores the file is activating the EXECUTABLE with the following command line options: `remove` -uri=storage-uri more options Details on the meaning of certain arguments are described in [section\_title][1]. For return codes see [section\_title][return code].

The EXECUTABLE is supposed to remove the file from the TSS and report a zero return code. If a non-zero error code is returned, the DCACHE will call the script again at a later point in time.

Configuring pools to interact with a Tertiary Storage System
============================================================

The EXECUTABLE interacting with the Tertiary Storage System (TSS), as described in the chapter above, has to be provided to DCACHE on all pools connected to the TSS. The EXECUTABLE, either a script or a binary, has to be made “executable” for the user, DCACHE is running as, on that host.

The following files have to be modified to allow DCACHE to interact with the TSS.

-   The
    FILE-POOLMANAGER
    file (one per system)
-   The pool layout file (one per pool host)
-   The pool 'setup' file (one per pool)
-   The namespaceDomain layout file (one per system)

After the layout files and the various 'setup' files have been corrected, the following domains have to be restarted :

-   pool services
-   dCacheDomain
-   namespaceDomain

The DCACHE layout files
-----------------------

### The `` file

To be able to read a file from the tape in case the cached file has been deleted from all pools, enable the restore-option. The best way to do this is to log in to the Admin Interface and run the following commands:

    [example.dcache.org] DC-PROMPT-LOCAL cd CELL-POOLMNGR
    [example.dcache.org] DC-PROMPT-PM pm set -stage-allowed=yes
    [example.dcache.org] DC-PROMPT-PM save

A restart of the DOMAIN-DCACHE is not necessary in this case.

Alternatively, if the file `` already exists then you can add the entry

    pm set -stage allowed=yes

and restart the DOMAIN-DCACHE.

> **Warning**
>
> Do not create the file
> FILE-POOLMANAGER
> with this single entry! This will result in an error.

### The pool layout

The DCACHE layout file must be modified for each pool node connected to a TSS. If your pool nodes have been configured correctly to work without TSS, you will find the entry `lfs=precious` in the layout file (that is located in `PATH-ODE-ED/layouts` and in the file `PATH-ODE-ED/dcache.conf` respectively) for each pool service. This entry is a disk-only-option and has to be removed for each pool which should be connected to a TSS. This will default the `lfs` parameter to `hsm` which is exactly what we need.

### The pool 'setup' file

The pool 'setup' file is the file `$poolHomeDir/$poolName/setup`. It mainly defines 3 details related to TSS connectivity.

-   Pointer to the EXECUTABLE which is launched on storing and fetching files.
-   The maximum number of concurrent STOREFILE requests allowed per pool.
-   The maximum number of concurrent FETCHFILE requests allowed per pool.

Define the EXECUTABLE and Set the maximum number of concurrent PUT and GET operations:

    hsm set hsmType [hsmInstanceName] [-command=/path/to/executable] [-key=value]

    #
    #  PUT operations
    # set the maximum number of active PUT operations >= 1
    #
    st set max active numberOfConcurrentPUTS

    #
    # GET operations
    # set the maximum number of active GET operations >= 1
    #
    rh set max active numberOfConcurrentGETs

-   hsmType
    : the type ot the TSS system. Must be set to
    osm
    for basic setups.
-   hsmInstanceName
    : the instance name of the TSS system. Must be set to
    osm
    for basic setups.
-   /path/to/executable
    : the full path to the EXECUTABLE which should be launched for each TSS operation.

Setting the maximum number of concurrent PUT and GET operations.

Both numbers must be non zero to allow the pool to perform transfers.

We provide a [script][EXECUTABLE] to simulate a connection to a TSS. To use this script place it in the directory ``, and create a directory to simulate the base of the TSS.

    PROMPT-ROOT mkdir -p /hsmTape/data

Login to the Admin Interface to change the entry of the pool 'setup' file for a pool named pool\_1.

    DC-PROMPT-LOCAL cd pool_1
    DC-PROMPT-POOL1 hsm set osm osm
    DC-PROMPT-POOL1 hsm set osm -command=PATH-ODJ-USDL/hsmscript.sh
    DC-PROMPT-POOL1 hsm set osm -hsmBase=/hsmTape
    DC-PROMPT-POOL1 st set max active 5
    DC-PROMPT-POOL1 rh set max active 5
    DC-PROMPT-POOL1 save

### The namespace layout

In order to allow DCACHE to remove files from attached TSSes, the “cleaner.enable.hsm = true” must be added immediately underneath the \[namespaceDomain/cleaner\] service declaration:

    [namespaceDomain]
     ... other services ...
    [namespaceDomain/cleaner]
    cleaner.enable.hsm = true
    .. more ...

What happens next
-----------------

After restarting the necessary DCACHE domains, pools, already containing files, will start transferring them into the TSS as those files only have a disk copy so far. The number of transfers is determined by the configuration in the pool 'setup' file as described above in [section\_title].

How to Store-/Restore files via the Admin Interface
===================================================

In order to see the state of files within a pool, login into the pool in the admin interface and run the command `rep ls`.

    [example.dcache.org] DC-PROMPT-POOL rep ls

The output will have the following format:

    PNFSID <MODE-BITS(LOCK-TIME)[OPEN-COUNT]> SIZE si={STORAGE-CLASS}

-   PNFSID: The PNFSID of the file
-   MODE-BITS:
           CPCScsRDXEL
           |||||||||||
           ||||||||||+--  (L) File is locked (currently in use)
           |||||||||+---  (E) File is in error state
           ||||||||+----  (X) File is pinned (aka "sticky")
           |||||||+-----  (D) File is in process of being destroyed
           ||||||+------  (R) File is in process of being removed
           |||||+-------  (s) File sends data to back end store
           ||||+--------  (c) File sends data to client (DCAP,FTP...)
           |||+---------  (S) File receives data from back end store
           ||+----------  (C) File receives data from client (DCAP,FTP)
           |+-----------  (P) File is precious, i.e., it is only on disk
           +------------  (C) File is on tape and only cached on disk.

-   LOCK-TIME: The number of milli-seconds this file will still be locked. Please note that this is an internal lock and not the pin-time (SRM).
-   OPEN-COUNT: Number of clients currently reading this file.
-   SIZE: File size
-   STORAGE-CLASS: The storage class of this file.

    [example.dcache.org] DC-PROMPT-POOL1 rep ls
    00008F276A952099472FAD619548F47EF972 <-P---------L(0)[0]> 291910 si={dteam:STATIC}
    00002A9282C2D7A147C68A327208173B81A6 <-P---------L(0)[0]> 2011264 si={dteam:STATIC}
    0000EE298D5BF6BB4867968B88AE16BA86B0 <C----------L(0)[0]> 1976 si={dteam:STATIC}

In order to `flush` a file to the tape run the command `flush pnfsid`.

    [example.dcache.org] DC-PROMPT-POOL flush pnfsid pnfsid

    [example.dcache.org] DC-PROMPT-POOL1 flush pnfsid 00002A9282C2D7A147C68A327208173B81A6
    Flush Initiated

A file that has been flushed to tape gets the flag 'C'.

    [example.dcache.org] DC-PROMPT-POOL1 rep ls
    00008F276A952099472FAD619548F47EF972 <-P---------L(0)[0]> 291910 si={dteam:STATIC}
    00002A9282C2D7A147C68A327208173B81A6 <C----------L(0)[0]> 2011264 si={dteam:STATIC}
    0000EE298D5BF6BB4867968B88AE16BA86B0 <C----------L(0)[0]> 1976 si={dteam:STATIC}

To remove such a file from the repository run the command `rep rm`.

    [example.dcache.org] DC-PROMPT-POOL rep rm pnfsid

    [example.dcache.org] DC-PROMPT-POOL1 rep rm  00002A9282C2D7A147C68A327208173B81A6
    Removed 00002A9282C2D7A147C68A327208173B81A6

In this case the file will be restored when requested.

To `restore` a file from the tape you can simply request it by initializing a reading transfer or you can fetch it by running the command `rh
      restore`.

    [example.dcache.org] DC-PROMPT-POOL rh restore [-block] pnfsid

    [example.dcache.org] DC-PROMPT-POOL1 rh restore 00002A9282C2D7A147C68A327208173B81A6
    Fetch request queued

How to monitor what's going on
==============================

This section briefly describes the commands and mechanisms to monitor the TSS PUT, GET and REMOVE operations. DCACHE provides a configurable logging facility and a Command Line Admin Interface to query and manipulate transfer and waiting queues.

Log Files
---------

By default DCACHE is configured to only log information if something unexpected happens. However, to get familiar with Tertiary Storage System interactions you might be interested in more details. This section provides advice on how to obtain this kind of information.

### The EXECUTABLE log file

Since you provide the EXECUTABLE, interfacing DCACHE and the TSS, it is in your responsibility to ensure sufficient logging information to be able to trace possible problems with either DCACHE or the TSS. Each request should be printed with the full set of parameters it receives, together with a timestamp. Furthermore information returned to DCACHE should be reported.

### DCACHE log files in general

In DCACHE, each domain (e.g. DOMAIN-DCACHE, DOMAIN-POOL etc) prints logging information into its own log file named after the domain. The location of those log files it typically the `/var/log` or `/var/log/dCache` directory depending on the individual configuration. In the default logging setup only errors are reported. This behavior can be changed by either modifying `PATH-ODE-ED/logback.xml` or using the DCACHE CLI to increase the log level of particular components as described [below].

#### Increase the DCACHE log level by changes in `PATH-ODE-ED/logback.xml`

If you intend to increase the log level of all components on a particular host you would need to change the `PATH-ODE-ED/logback.xml` file as described below. DCACHE components need to be restarted to activate the changes.

    <threshold>
         <appender>stdout</appender>
         <logger>root</logger>
         <level>warn</level>
       </threshold>

needs to be changed to

    <threshold>
         <appender>stdout</appender>
         <logger>root</logger>
         <level>info</level>
       </threshold>

> **Important**
>
> The change might result in a significant increase in log messages. So don't forget to change back before starting production operation. The next section describes how to change the log level in a running system.

#### Increase the DCACHE log level via the Command Line Admin Interface

Login into the DCACHE Command Line Admin Interface and increase the log level of a particular service, for instance for the POOLMNGR service:

    [example.dcache.org] DC-PROMPT-LOCAL cd PoolManager
    [example.dcache.org] DC-PROMPT-PM log set stdout ROOT INFO
    [example.dcache.org] DC-PROMPT-PM log ls
    stdout:
      ROOT=INFO
      dmg.cells.nucleus=WARN*
      logger.org.dcache.cells.messages=ERROR*
    .....

Obtain information via the DCACHE Command Line Admin Interface
--------------------------------------------------------------

The DCACHE Command Line Admin Interface gives access to information describing the process of storing and fetching files to and from the TSS, as there are:

-   The
    Pool Manager Restore Queue
    . A list of all requests which have been issued to all pools for a FETCHFILE operation from the TSS (rc ls)
-   The
    Pool Collector Queue
    . A list of files, per pool and storage group, which will be scheduled for a STOREFILE operation as soon as the configured trigger criteria match.
-   The
    Pool STOREFILE Queue
    . A list of files per pool, scheduled for the STOREFILE operation. A configurable amount of requests within this queue are active, which is equivalent to the number of concurrent store processes, the rest is inactive, waiting to become active.
-   The
    Pool FETCHFILE Queue
    . A list of files per pool, scheduled for the FETCHFILE operation. A configurable amount of requests within this queue are active, which is equivalent to the number of concurrent fetch processes, the rest is inactive, waiting to become active.

For evaluation purposes, the pinboard of each component can be used to track down DCACHE behavior. The pinboard only keeps the most recent 200 lines of log information but reports not only errors but informational messages as well.

Check the pinboard of a service, here the POOLMNGR service.

    [example.dcache.org] DC-PROMPT-LOCAL cd PoolManager
    [example.dcache.org] DC-PROMPT-PM show pinboard 100
    08.30.45  [Thread-7] [pool_1 PoolManagerPoolUp] sendPoolStatusRelay: ...
    08.30.59  [writeHandler] [NFSv41-dcachetogo PoolMgrSelectWritePool ...
    ....

**The CELL-POOLMNGR Restore Queue.**

Remove the file `test.root` with the PNFS-ID 00002A9282C2D7A147C68A327208173B81A6.

    [example.dcache.org] (pool_1) admin > rep rm  00002A9282C2D7A147C68A327208173B81A6

Request the file `test.root`

    PROMPT-USER PATH-ODDB-N-Sdccp dcap://example.dcache.org:/data/test.root test.root

Check the CELL-POOLMNGR Restore Queue:

    [example.dcache.org] DC-PROMPT-LOCAL cd PoolManager
    [example.dcache.org] DC-PROMPT-PM rc ls
    0000AB1260F474554142BA976D0ADAF78C6C@0.0.0.0/0.0.0.0-*/* m=1 r=0 [pool_1] [Staging 08.15 17:52:16] {0,}

**The Pool Collector Queue.**

    [example.dcache.org] DC-PROMPT-LOCAL cd pool_1
    [example.dcache.org] DC-PROMPT-POOL1 queue ls -l queue
                       Name: chimera:alpha
                  Class@Hsm: chimera:alpha@osm
     Expiration rest/defined: -39 / 0   seconds
     Pending   rest/defined: 1 / 0
     Size      rest/defined: 877480 / 0
     Active Store Procs.   :  0
      00001BC6D76570A74534969FD72220C31D5D


    [example.dcache.org] DC-PROMPT-POOL1 flush ls
    Class                 Active   Error  Last/min  Requests    Failed
    dteam:STATIC@osm           0       0         0         1         0

**The pool STOREFILE Queue.**

    [example.dcache.org] DC-PROMPT-LOCAL cd pool_1
    [example.dcache.org] DC-PROMPT-POOL1 st ls
    0000EC3A4BFCA8E14755AE4E3B5639B155F9  1   Fri Aug 12 15:35:58 CEST 2011

**The pool FETCHFILE Queue.**

    [example.dcache.org] DC-PROMPT-LOCAL cd pool_1
    [example.dcache.org] DC-PROMPT-POOL1  rh ls
    0000B56B7AFE71C14BDA9426BBF1384CA4B0  0   Fri Aug 12 15:38:33 CEST 2011

To check the repository on the pools run the command `rep ls` that is described in the beginning of [section\_title][2].

Example of an EXECUTABLE to simulate a tape backend
===================================================

    #!/bin/sh
    #
    #set -x
    #
    logFile=/tmp/hsm.log
    #
    ################################################################
    #
    #  Some helper functions
    #
    ##.........................................
    #
    # print usage
    #
    usage() {
       echo "Usage : put|get <pnfsId> <filePath> [-si=<storageInfo>] [-key[=value] ...]" 1>&2
    }
    ##.........................................
    #
    #
    printout() {
    #---------
       echo "$pnfsid : $1" >>${logFile}
       return 0
    }
    ##.........................................
    #
    #  print error into log file and to stdout.
    #
    printerror() {
    #---------

       if [ -z "$pnfsid" ] ; then
    #      pp="000000000000000000000000000000000000"
          pp="------------------------------------"
       else
          pp=$pnfsid
       fi

       echo "$pp : (E) : $*" >>${logFile}
       echo "$pp : $*" 1>&2

    }
    ##.........................................
    #
    #  find a key in the storage info
    #
    findKeyInStorageInfo() {
    #-------------------

       result=`echo $si  | awk  -v hallo=$1 -F\; '{ for(i=1;i<=NF;i++){ split($i,a,"=") ; if( a[1] == hallo )print a[2]} }'`
       if [ -z "$result" ] ; then return 1 ; fi
       echo $result
       exit 0

    }
    ##.........................................
    #
    #  find a key in the storage info
    #
    printStorageInfo() {
    #-------------------
       printout "storageinfo.StoreName : $storeName"
       printout "storageinfo.store : $store"
       printout "storageinfo.group : $group"
       printout "storageinfo.hsm   : $hsmName"
       printout "storageinfo.accessLatency   : $accessLatency"
       printout "storageinfo.retentionPolicy : $retentionPolicy"
       return 0
    }
    ##.........................................
    #
    #  assign storage info the keywords
    #
    assignStorageInfo() {
    #-------------------

        store=`findKeyInStorageInfo "store"`
        group=`findKeyInStorageInfo "group"`
        storeName=`findKeyInStorageInfo "StoreName"`
        hsmName=`findKeyInStorageInfo "hsm"`
        accessLatency=`findKeyInStorageInfo "accessLatency"`
        retentionPolicy=`findKeyInStorageInfo "retentionPolicy"`
        return 0
    }
    ##.........................................
    #
    # split the arguments into the options -<key>=<value> and the
    # positional arguments.
    #
    splitArguments() {
    #----------------
    #
      args=""
      while [ $# -gt 0 ] ; do
        if expr "$1" : "-.*" >/dev/null ; then
           a=`expr "$1" : "-\(.*\)" 2>/dev/null`
           key=`echo "$a" | awk -F= '{print $1}' 2>/dev/null`
             value=`echo "$a" | awk -F= '{for(i=2;i<NF;i++)x=x $i "=" ; x=x $NF ; print x }' 2>/dev/null`
           if [ -z "$value" ] ; then a="${key}=" ; fi
           eval "${key}=\"${value}\""
           a="export ${key}"
           eval "$a"
        else
           args="${args} $1"
        fi
        shift 1
      done
      if [ ! -z "$args" ] ; then
         set `echo "$args" | awk '{ for(i=1;i<=NF;i++)print $i }'`
      fi
      return 0
    }
    #
    #
    ##.........................................
    #
    splitUri() {
    #----------------
    #
      uri_hsmName=`expr "$1" : "\(.*\)\:.*"`
      uri_hsmInstance=`expr "$1" : ".*\:\/\/\(.*\)\/.*"`
      uri_store=`expr "$1" : ".*\/\?store=\(.*\)&group.*"`
      uri_group=`expr "$1" : ".*group=\(.*\)&bfid.*"`
      uri_bfid=`expr "$1" : ".*bfid=\(.*\)"`
    #
      if [  \( -z "${uri_store}" \) -o \( -z "${uri_group}" \) -o \(  -z "${uri_bfid}" \) \
         -o \( -z "${uri_hsmName}" \) -o \( -z "${uri_hsmInstance}" \) ] ; then
         printerror "Illegal URI formal : $1"
         return 1
      fi
      return 0

    }
    #########################################################
    #
    echo "--------- $* `date`" >>${logFile}
    #
    #########################################################
    #
    createEnvironment() {

       if [ -z "${hsmBase}" ] ; then
          printerror "hsmBase not set, can't continue"
          return 1
       fi
       BASE=${hsmBase}/data
       if [ ! -d ${BASE} ] ; then
          printerror "${BASE} is not a directory or doesn't exist"
          return 1
       fi
    }
    ##
    #----------------------------------------------------------
    doTheGetFile() {

       splitUri $1
       [ $? -ne 0 ] && return 1

       createEnvironment
       [ $? -ne 0 ] && return 1

       pnfsdir=${BASE}/$uri_hsmName/${uri_store}/${uri_group}
       pnfsfile=${pnfsdir}/$pnfsid

       cp $pnfsfile $filename 2>/dev/null
       if [ $? -ne 0 ] ; then
          printerror "Couldn't copy file $pnfsfile to $filename"
          return 1
       fi

       return 0
    }
    ##
    #----------------------------------------------------------
    doTheStoreFile() {

       splitUri $1
       [ $? -ne 0 ] && return 1

       createEnvironment
       [ $? -ne 0 ] && return 1

       pnfsdir=${BASE}/$hsmName/${store}/${group}
       mkdir -p ${pnfsdir} 2>/dev/null
       if [ $? -ne 0 ] ; then
          printerror "Couldn't create $pnfsdir"
          return 1
       fi
       pnfsfile=${pnfsdir}/$pnfsid

       cp $filename $pnfsfile 2>/dev/null
       if [ $? -ne 0 ] ; then
          printerror "Couldn't copy file $filename to $pnfsfile"
          return 1
       fi

       return 0

    }
    ##
    #----------------------------------------------------------
    doTheRemoveFile() {

       splitUri $1
       [ $? -ne 0 ] && return 1

       createEnvironment
       [ $? -ne 0 ] && return 1

       pnfsdir=${BASE}/$uri_hsmName/${uri_store}/${uri_group}
       pnfsfile=${pnfsdir}/$uri_bfid

       rm $pnfsfile 2>/dev/null
       if [ $? -ne 0 ] ; then
          printerror "Couldn't remove file $pnfsfile"
          return 1
       fi

       return 0
    }
    #########################################################
    #
    #  split arguments
    #
      args=""
      while [ $# -gt 0 ] ; do
        if expr "$1" : "-.*" >/dev/null ; then
           a=`expr "$1" : "-\(.*\)" 2>/dev/null`
           key=`echo "$a" | awk -F= '{print $1}' 2>/dev/null`
             value=`echo "$a" | awk -F= '{for(i=2;i<NF;i++)x=x $i "=" ; x=x $NF ; print x }' 2>/dev/null`
           if [ -z "$value" ] ; then a="${key}=" ; fi
           eval "${key}=\"${value}\""
           a="export ${key}"
           eval "$a"
        else
           args="${args} $1"
        fi
        shift 1
      done
      if [ ! -z "$args" ] ; then
         set `echo "$args" | awk '{ for(i=1;i<=NF;i++)print $i }'`
      fi
    #
    #
    if [ $# -lt 1 ] ; then
        printerror "Not enough arguments : ... put/get/remove ..."
        exit 1
    fi
    #
    command=$1
    pnfsid=$2
    #
    # !!!!!!  Hides a bug in the dCache HSM remove
    #
    if [ "$command" = "remove" ] ; then pnfsid="000000000000000000000000000000000000" ; fi
    #
    #
    printout "Request for $command started `date`"
    #
    ################################################################
    #
    if [ "$command" = "put" ] ; then
    #
    ################################################################
    #
      filename=$3
    #
      if [ -z "$si" ] ; then
         printerror "StorageInfo (si) not found in put command"
         exit 5
      fi
    #
      assignStorageInfo
    #
      printStorageInfo
    #
      if [ \( -z "${store}" \) -o \( -z "${group}" \) -o \( -z "${hsmName}" \) ] ; then
         printerror "Didn't get enough information to flush : hsmName = $hsmName store=$store group=$group pnfsid=$pnfsid "
         exit 3
      fi
    #
      uri="$hsmName://$hsmName/?store=${store}&group=${group}&bfid=${pnfsid}"

      printout "Created identifier : $uri"

      doTheStoreFile $uri
      rc=$?
      if [ $rc -eq 0 ] ; then echo $uri ; fi

      printout "Request 'put' finished at `date` with return code $rc"
      exit $rc
    #
    #
    ################################################################
    #
    elif [ "$command" = "get"  ] ; then
    #
    ################################################################
    #
      filename=$3
      if [ -z "$uri" ] ; then
         printerror "Uri not found in arguments"
         exit 3
      fi
    #
      printout "Got identifier : $uri"
    #
      doTheGetFile $uri
      rc=$?
      printout "Request 'get' finished at `date` with return code $rc"
      exit $rc
    #
    ################################################################
    #
    elif [ "$command" = "remove" ] ; then
    #
    ################################################################
    #
       if [ -z "$uri" ] ; then
          printerror "Illegal Argument error : URI not specified"
          exit 4
       fi
    #
       printout "Remove uri = $uri"
       doTheRemoveFile $uri
       rc=$?
    #
       printout "Request 'remove' finished at `date` with return code $rc"
       exit $rc
    #
    else
    #
       printerror "Expected command : put/get/remove , found : $command"
       exit 1
    #
    fi

  [EXECUTABLE]: #tss-executable
  [return code]: #cf-tss-support-return-codes
  [section\_title]: #cf-tss-pools-layout-setup
  [1]: #cf-tss-support-clo
  [above]: #cf-tss-support-storage-uri
  [below]: #cf-tss-monitor-log-cli
  [2]: #cf-tss-pools-admin

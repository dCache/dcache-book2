CHAPTER 4. CHIMERA
==================

###Table of Contents 

* [Mounting Chimera through NFS](#mounting-chimera-through-nfs)

             *[Using dCap with a mounted file system](using-dcap-with-a-mounted-file-system)
             
 * [Communicating with Chimera](#communicating-with-chimera)
    IDs
    Directory Tags
        Create, List and Read Directory Tags if the Namespace is not Mounted
        Create, List and Read Directory Tags if the Namespace is Mounted
        Directory Tags and Command Files
        Directory Tags for dCache
        Storage Class and Directory Tags


DCACHE is a distributed storage system, nevertheless it provides a single-rooted file system view. While DCACHE supports multiple namespace providers, CHIMERA is the recommended provider and is used by default.

The inner DCACHE components talk to the namespace via a module called CELL-PNFSMNGR, which in turn communicates with the CHIMERA database using a thin JAVA layer, which in turn communicates directly with the CHIMERA database. CHIMERA allows direct access to the namespace by providing an NFS3 and NFS4 server. Clients can NFS-mount the namespace locally. This offers the opportunity to use OS-level tools like PROG-LS, PROG-MKDIR, PROG-MV for CHIMERA. Direct I/O-operations like PROG-CP and PROG-CAT are possible with the NFS4 door.

The properties of CHIMERA are defined in `PATH-ODS-USD/defaults/chimera.properties`. For customisation the files `PATH-ODE-ED/layouts/mylayout.conf` or `PATH-ODE-ED/dcache.conf` should be modified (see [???]).

This example shows an extract of the `PATH-ODE-ED/layouts/mylayout.conf` file in order to run DCACHE with NFS3.

    [namespaceDomain]
    [namespaceDomain/pnfsmanager]
    [namespaceDomain/nfs]
    nfs.version=3

If you want to run the NFS4 server you need to add the corresponding NFS service to a domain in the `PATH-ODE-ED/layouts/mylayout.conf` file and start this domain.

    [namespaceDomain]
    [namespaceDomain/pnfsmanager]
    [namespaceDomain/nfs]
    nfs.version = 4.1

If you wish DCACHE to access your CHIMERA with a PSQL user other than chimera then you must specify the username and password in `PATH-ODE-ED/dcache.conf`.

    chimera.db.user=myuser
    chimera.db.password=secret

> **Important**
>
> Do not update configuration values in `PATH-ODS-USD/defaults/chimera.properties`, since changes to this file will be overwritten by updates.

Mounting CHIMERA through NFS
============================

DCACHE does not need the CHIMERA filesystem to be mounted but a mounted file system is convenient for administrative access. This offers the opportunity to use OS-level tools like `ls` and `mkdir` for CHIMERA. However, direct I/O-operations like `cp` are not possible, since the NFS3 interface provides the namespace part only. This section describes how to start the CHIMERA NFS3 server and mount the name space.

If you want to mount CHIMERA for easier administrative access, you need to edit the `/etc/exports` file as the CHIMERA NFS server uses it to manage exports. If this file doesn't exist it must be created. The typical `exports` file looks like this:

    / localhost(rw)
    /data
    # or
    # /data *.my.domain(rw)

As any RPC service CHIMERA NFS requires rpcbind service to run on the host. Nevertheless rpcbind has to be configured to accept requests from CHIMERA NFS.

On RHEL6 based systems you need to add

    RPCBIND_ARGS="-i"

into `/etc/sysconfig/rpcbind` and restart rpcbind. Check your OS manual for details.

    PROMPT-ROOT service rpcbind restart
    Stopping rpcbind:                                          [  OK  ]
    Starting rpcbind:                                          [  OK  ]

If your OS does not provide rpcbind CHIMERA NFS can use an embedded rpcbind. This requires to disable the portmap service if it exists.

    PROMPT-ROOT /etc/init.d/portmap stop
    Stopping portmap: portmap

and restart the domain in which the NFS server is running.

    PROMPT-ROOT PATH-ODB-N-Sdcache restart namespaceDomain

Now you can mount CHIMERA by

    PROMPT-ROOT mount localhost:/ /mnt

and create the root of the CHIMERA namespace which you can call `data`:

    PROMPT-ROOT mkdir -p /mnt/data

If you don't want to mount chimera you can create the root of the CHIMERA namespace by

    PROMPT-ROOT CHIMERA-CLI mkdir /data

You can now add directory tags. For more information on tags see [section\_title].

    PROMPT-ROOT CHIMERA-CLI writetag /data sGroup "chimera"
    PROMPT-ROOT CHIMERA-CLI writetag /data OSMTemplate "StoreName sql"

Using DCAP with a mounted file system
-------------------------------------

If you plan to use DCAP with a mounted file system instead of the URL-syntax (e.g. PROG-DCCP `/data/file1` `/tmp/file1`), you need to mount the root of CHIMERA locally (remote mounts are not allowed yet). This will allow us to establish wormhole files so DCAP clients can discover the DCAP doors.

    PROMPT-ROOT mount localhost:/ /mnt
    PROMPT-ROOT mkdir /mnt/admin/etc/config/dCache
    PROMPT-ROOT touch /mnt/admin/etc/config/dCache/dcache.conf
    PROMPT-ROOT touch /mnt/admin/etc/config/dCache/'.(fset)(dcache.conf)(io)(on)'
    PROMPT-ROOT echo "door host:port" > /mnt/admin/etc/config/dCache/dcache.conf

The default values for ports can be found in [???][1] (for DCAP the default port is `22125`) and in the file `PATH-ODS-USD/defaults/dcache.properties`. They can be altered in `PATH-ODE-ED/dcache.conf`

Create the directory in which the users are going to store their data and change to this directory.

    PROMPT-ROOT mkdir -p /mnt/data
    PROMPT-ROOT cd /mnt/data

Now you can copy a file into your DCACHE

    PROMPT-ROOT dccp /bin/sh test-file
    735004 bytes (718 kiB) in 0 seconds

and copy the data back using the PROG-DCCP command.

    PROMPT-ROOT dccp test-file /tmp/testfile
    735004 bytes (718 kiB) in 0 seconds

The file has been transferred succesfully.

Now remove the file from the DCACHE.

    PROMPT-ROOT rm  test-file

When the configuration is complete you can unmount CHIMERA:

    PROMPT-ROOT umount /mnt

> **Note**
>
> Please note that whenever you need to change the configuration, you have to remount the root `localhost:/` to a temporary location like `/mnt`.

Communicating with CHIMERA
==========================

Many configuration parameters of CHIMERA and the application specific meta data is accessed by reading, writing, or creating files of the form `.(command)(para)`. For example, the following prints the CHIMERAID of the file `/data/some/dir/file.dat`:

    PROMPT-USER cat /data/any/sub/directory/'.(id)(file.dat)'
    0004000000000000002320B8 PROMPT-USER 

From the point of view of the NFS protocol, the file `.(id)(file.dat)` in the directory `/data/some/dir/` is read. However, CHIMERA interprets it as the command `id` with the parameter `file.dat` executed in the directory `/data/some/dir/`. The quotes are important, because the shell would otherwise try to interpret the parentheses.

Some of these command files have a second parameter in a third pair of parentheses. Note, that files of the form `.(command)(para)` are not really files. They are not shown when listing directories with `ls`. However, the command files are listed when they appear in the argument list of `ls` as in

    PROMPT-USER ls -l '.(tag)(sGroup)'
    -rw-r--r-- 11 root root 7 Aug 6 2010 .(tag)(sGroup)

Only a subset of file operations are allowed on these special command files. Any other operation will result in an appropriate error. Beware, that files with names of this form might accidentally be created by typos. They will then be shown when listing the directory.

IDs
===

Each file in CHIMERA has a unique 18 byte long ID. It is referred to as ChimeraID or as pnfsID. This is comparable to the inode number in other filesystems. The ID used for a file will never be reused, even if the file is deleted. DCACHE uses the ID for all internal references to a file.

The ID of the file `example.org/data/examplefile` can be obtained by reading the command-file `.(id)(examplefile)` in the directory of the file.

    PROMPT-USER cat /example.org/data/'.(id)(examplefile)'
    0000917F4A82369F4BA98E38DBC5687A031D

A file in CHIMERA can be referred to by the ID for most operations.

The name of a file can be obtained from the ID with the command `nameof` as follows:

    PROMPT-USER cd /example.org/data/
    PROMPT-USER cat '.(nameof)(0000917F4A82369F4BA98E38DBC5687A031D)'
    examplefile

And the ID of the directory it resides in is obtained by:

    PROMPT-USER cat '.(parent)(0000917F4A82369F4BA98E38DBC5687A031D)'
    0000595ABA40B31A469C87754CD79E0C08F2

This way, the complete path of a file may be obtained starting from the ID.

Directory Tags
==============

In the CHIMERA namespace, each directory can have a number of tags. These directory tags may be used within DCACHE to control the file placement policy in the pools (see [???][2]). They might also be used by a [tertiary storage system] for similar purposes (e.g. controlling the set of tapes used for the files in the directory).

> **Note**
>
> Directory tags are not needed to control the behaviour of DCACHE. DCACHE works well without directory tags.

Create, List and Read Directory Tags if the Namespace is not Mounted
--------------------------------------------------------------------

You can create tags with

    PROMPT-USER CHIMERA-CLI writetag directory tagName "content"

list tags with

    PROMPT-USER CHIMERA-CLI lstag directory 

and read tags with

    PROMPT-USER CHIMERA-CLI readtag directory tagName

Create tags for the directory `data` with

    PROMPT-USER CHIMERA-CLI writetag /data sGroup "myGroup"
    PROMPT-USER CHIMERA-CLI writetag /data OSMTemplate "StoreName myStore"

list the existing tags with

    PROMPT-USER CHIMERA-CLI lstag /data
    Total: 2
    OSMTemplate
    sGroup

and their content with

    PROMPT-USER CHIMERA-CLI readtag /data OSMTemplate
    StoreName myStore
    PROMPT-USER CHIMERA-CLI readtag /data sGroup
    myGroup

Create, List and Read Directory Tags if the Namespace is Mounted
----------------------------------------------------------------

If the namespace is mounted, change to the directory for which the tag should be set and create a tag with

    PROMPT-USER cd directory
    PROMPT-USER echo 'content1' > '.(tag)(tagName1)'
    PROMPT-USER echo 'content2' > '.(tag)(tagName2)'

Then the existing tags may be listed with

    PROMPT-USER cat '.(tags)()'
    .(tag)(tagname1)
    .(tag)(tagname2)

and the content of a tag can be read with

    PROMPT-USER cat '.(tag)(tagname1)'
    content1
    PROMPT-USER cat '.(tag)(tagName2)'
    content2

Create tags for the directory `data` with

    PROMPT-USER cd data
    PROMPT-USER echo 'StoreName myStore' > '.(tag)(OSMTemplate)'
    PROMPT-USER echo 'myGroup' > '.(tag)(sGroup)'

list the existing tags with

    PROMPT-USER cat '.(tags)()'
    .(tag)(OSMTemplate)
    .(tag)(sGroup)

and their content with

    PROMPT-USER cat '.(tag)(OSMTemplate)'
    StoreName myStore
    PROMPT-USER cat '.(tag)(sGroup)'
     myGroup

A nice trick to list all tags with their contents is

    PROMPT-USER grep "" $(cat  ".(tags)()")
    .(tag)(OSMTemplate):StoreName myStore
    .(tag)(sGroup):myGroup

Directory Tags and Command Files
--------------------------------

When creating or changing directory tags by writing to the command file as in

    PROMPT-USER echo 'content' > '.(tag)(tagName)'

one has to take care not to treat the command files in the same way as regular files, because tags are different from files in the following aspects:

1.  The tagName is limited to 62 characters and the content to 512 bytes. Writing more to the command file, will be silently ignored.

2.  If a tag which does not exist in a directory is created by writing to it, it is called a *primary* tag.

3.  Tags are *inherited* from the parent directory by a newly created directory. Changing a primary tag in one directory will change the tags inherited from it in the same way. Creating a new primary tag in a directory will not create an inherited tag in its subdirectories.

    Moving a directory within the CHIMERA namespace will not change the inheritance. Therefore, a directory does not necessarily inherit tags from its parent directory. Removing an inherited tag does not have any effect.

4.  Empty tags are ignored.

Directory Tags for DCACHE
-------------------------

The following directory tags appear in the DCACHE context:

OSMTemplate  
Must contain a line of the form “`StoreName` storeName” and specifies the name of the store that is used by DCACHE to construct the [storage class] if the HSM Type is `osm`.

HSMType  
The `HSMType` tag is normally determined from the other existing tags. E.g., if the tag `OSMTemplate` exists, `HSMType`=`osm` is assumed. With this tag it can be set explicitly. A class implementing that HSM type has to exist. Currently the only implementations are `osm` and `enstore`.

sGroup  
The storage group is also used to construct the [storage class] if the `HSMType` is `osm`.

cacheClass  
The cache class is only used to control on which pools the files in a directory may be stored, while the storage class (constructed from the two above tags) might also be used by the HSM. The cache class is only needed if the above two tags are already fixed by HSM usage and more flexibility is needed.

hsmInstance  
If not set, the `hsmInstance` tag will be the same as the `HSMType` tag. Setting this tag will only change the name as used in the [storage class] and in the pool commands.

WriteToken  
Assign a `WriteToken` tag to a directory in order to be able to write to a space token without using the SRM.

Storage Class and Directory Tags
--------------------------------

The [storage class][3] is a string of the form StoreName:StorageGroup@hsm-type, where StoreName is given by the `OSMTemplate` tag, StorageGroup by the `sGroup` tag and hsm-type by the `HSMType` tag. As mentioned above the `HSMType` tag is assumed to be `osm` if the tag `OSMTemplate` exists.

In the examples above two tags have been created.

    PROMPT-USER CHIMERA-CLI lstag /data
    Total: 2
    OSMTemplate
    sGroup

As the tag `OSMTemplate` was created the tag `HSMType` is assumed to be `osm`.

The storage class of the files which are copied into the directory `/data` after the tags have been set will be `myStore:myGroup@osm`.

If directory tags are used to control the behaviour of DCACHE and/or a tertiary storage system, it is a good idea to plan the directory structure in advance, thereby considering the necessary tags and how they should be set up. Moving directories should be done with great care or even not at all. Inherited tags can only be created by creating a new directory.

Assume that data of two experiments, `experiment-a` and `experiment-b` is written into a namespace tree with subdirectories `/data/experiment-a` and `/data/experiment-b`. As some pools of the DCACHE are financed by `experiment-a` and others by `experiment-b` they probably do not like it if they are also used by the other group. To avoid this the directories of `experiment-a` and `experiment-b` can be tagged.

    PROMPT-USER CHIMERA-CLI writetag /data/experiment-a OSMTemplate "StoreName exp-a"
    PROMPT-USER CHIMERA-CLI writetag /data/experiment-b OSMTemplate "StoreName exp-b"

Data from `experiment-a` taken in 2010 shall be written into the directory `/data/experiment-a/2010` and data from `experiment-a` taken in 2011 shall be written into `/data/experiment-a/2011`. Data from `experiment-b` shall be written into `/data/experiment-b`. Tag the directories correspondingly.

    PROMPT-USER CHIMERA-CLI writetag /data/experiment-a/2010 sGroup "run2010"
    PROMPT-USER CHIMERA-CLI writetag /data/experiment-a/2011 sGroup "run2011"
    PROMPT-USER CHIMERA-CLI writetag /data/experiment-b sGroup "alldata"

List the content of the tags by

    PROMPT-USER CHIMERA-CLI readtag /data/experiment-a/2010 OSMTemplate
    StoreName exp-a
    PROMPT-USER CHIMERA-CLI readtag /data/experiment-a/2010 sGroup
    run2010
    PROMPT-USER CHIMERA-CLI readtag /data/experiment-a/2011 OSMTemplate
    StoreName exp-a
    PROMPT-USER CHIMERA-CLI readtag /data/experiment-a/2011 sGroup
    run2011
    PROMPT-USER CHIMERA-CLI readtag /data/experiment-b/2011 OSMTemplate
    StoreName exp-b
    PROMPT-USER CHIMERA-CLI readtag /data/experiment-b/2011 sGroup
    alldata

As the tag `OSMTemplate` was created the `HSMType` is assumed to be `osm`.

The storage classes of the files which are copied into these directories after the tags have been set will be

-   exp-a:run2010@osm
    for the files in
    /data/experiment-a/2010
-   exp-a:run2011@osm
    for the files in
    /data/experiment-a/2011
-   exp-b:alldata@osm
    for the files in
    /data/experiment-b

To see how storage classes are used for pool selection have a look at the example 'Reserving Pools for Storage and Cache Classes' in the PoolManager chapter.

There are more tags used by DCACHE if the `HSMType` is `enstore`.

  [???]: #in-install-layout
  [section\_title]: #chimera-tags
  [1]: #rf-ports
  [2]: #cf-pm-psu
  [tertiary storage system]: #cf-tss
  [storage class]: #chimera-tags-storageClass
  [3]: #secStorageClass

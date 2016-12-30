DCACHE as NFS4 Server
=====================

This chapter explains how to configure DCACHE in order to access it via the NFS4 protocol, allowing clients to mount DCACHE and perform POSIX IO using standard NFS4 clients.

> **Important**
>
> The pNFS mentioned in this chapter is the protocol NFS4-PNFS and not the namespace PNFS.

Setting up
==========

To allow file transfers in and out of DCACHE using NFS4-PNFS, a new DOOR-NFS4 must be started. This door acts then as the mount point for NFS clients.

To enable the DOOR-NFS4, you have to change the layout file corresponding to your DCACHE-instance. Enable the NFS within the domain that you want to run it by adding the following line

    ..
    [domainName/nfs]
    nfs.version = 4.1
    ..

You can just add the following lines to the layout file:

    ..
    [nfs-${host.name}Domain]
    [nfs-${host.name}Domain/nfs]
    nfs.version = 4.1
    ..

In addition to run an DOOR-NFS4 you need to add exports to the `/etc/exports` file. The format of `/etc/exports` is similar to the one which is provided by Linux:

    #
    <path> [host [(options)]]

Where options is a comma separated combination of:

`ro`  
matching clients can access this export only in read-only mode

`rw`  
matching clients can access this export only in read-write mode

`sec=krb5`  
matching clients must access NFS using RPCSEC\_GSS authentication. The Quality of Protection (QOP) is NONE, e.g., the data is neither encrypted nor signed when sent over the network. Nevertheless the RPC packets header still protected by checksum.

`sec=krb5i`  
matching clients have to access NFS using RPCSEC\_GSS authentication. The Quality of Protection (QOP) is INTEGRITY. The RPC requests and response are protected by checksum.

`sec=krb5p`  
matching clients have to access NFS using RPCSEC\_GSS authentication. The Quality of Protection (QOP) is PRIVACY. The RPC requests and response are protected by encryption.

For example:

    #
    /pnfs/dcache.org/data *.dcache.org (rw,sec=krb5i)

Notice, that security flavour used at mount time will be used for client - pool comminication as well.

Configuring DOOR-NFS4 with GSS-API support
==========================================

Adding `sec=krb5` into `/etc/exports` is not sufficient to get kerberos authentication to work.

All clients, pool nodes and node running DOOR-NFS4 must have a valid kerberos configuration. Each clients, pool node and node running DOOR-NFS4 must have a `/etc/krb5.keytab` with `nfs` service principal:

    nfs/host.domain@YOUR.REALM

The `PATH-ODE-ED/dcache.conf` on pool nodes and node running DOOR-NFS4 must enable kerberos and RPCSEC\_GSS:

    nfs.rpcsec_gss=true
    dcache.authn.kerberos.realm=YOUR.REALM
    dcache.authn.jaas.config=PATH-ODE-ED/gss.conf
    dcache.authn.kerberos.key-distribution-center-list=your.kdc.server

The `PATH-ODE-ED/gss.conf` on pool nodes and node running DOOR-NFS4 must configure Java's security module:

    com.sun.security.jgss.accept {
    com.sun.security.auth.module.Krb5LoginModule required
    doNotPrompt=true
    useKeyTab=true
    keyTab="${/}etc${/}krb5.keytab"
    debug=false
    storeKey=true
    principal="nfs/host.domain@YOUR.REALM";
    };

Now your NFS client can securely access DCACHE.

Configuring principal-id mapping for NFS access
===============================================

The NFS4 uses utf8 based strings to represent user and group names. This is the case even for non-kerberos based accesses. Nevertheless UNIX based clients as well as DCACHE internally use numbers to represent uid and gids. A special service, called idmapd, takes care for principal-id mapping. On the client nodes the file `/etc/idmapd.conf` is usually responsible for consistent mapping on the client side. On the server side, in case of DCACHE mapping done through gplazma2. The `identity` type of plug-in required by id-mapping service. Please refer to [???] for instructions about how to configure CELL-GPLAZMA.

Note, that `nfs4 domain` on clients must match `nfs.domain` value in `dcache.conf`.

To avoid big latencies and avoiding multiple queries for the same information, like ownership of a files in a big directory, the results from CELL-GPLAZMA are cached within DOOR-NFS4. The default values for cache size and life time are good enough for typical installation. Nevertheless they can be overriden in `dcache.conf` or layoutfile:

    ..
    # maximal number of entries in the cache
    nfs.idmap.cache.size = 512

    # cache entry maximal lifetime
    nfs.idmap.cache.timeout = 30

    # time unit used for timeout. Valid values are:
    # SECONDS, MINUTES, HOURS and DAYS
    nfs.idmap.cache.timeout.unit = SECONDS
    ..

  [???]: #cf-gplazma

dCache as xRootd-Server
=======================

This chapter explains how to configure DCACHE in order to access it via the XROOTD protocol, allowing XROOTD-Clients like ROOT's TXNetfile and xrdcp to do file operations against a DCACHE instance in a transparent manner. DCACHE implements version XROOTD-VERSION of XROOTD protocol.

Setting up
==========

To allow file transfers in and out of DCACHE using xrootd, a new DOOR-XROOTD must be started. This door acts then as the entry point to all XROOTD requests. Compared to the native xrootd server-implementation (produced by SLAC), the DOOR-XROOTD corresponds to the `redirector node`.

To enable the DOOR-XROOTD, you have to change the layout file corresponding to your DCACHE-instance. Enable the xrootd-service within the domain that you want to run it by adding the following line

    ..
    [domainName/xrootd]
    ..

You can just add the following lines to the layout file:

    ..
    [xrootd-${host.name}Domain]
    [xrootd-${host.name}Domain/xrootd]
    ..

After a restart of the domain running the DOOR-XROOTD, done e.g. by executing

    PROMPT-ROOT ${dCacheHome}/bin/dcache restart xrootd-babelfishDomain
    Stopping xrootd-babelfishDomain (pid=30246) 0 1 2 3 4 5 6 7 done
    Starting xrootd-babelfishDomain done

the DOOR-XROOTD should be running. A few minutes later it should appear at the web monitoring interface under "Cell Services" (see [???]).

Parameters
----------

The default port the DOOR-XROOTD is listening on is 1094. This can be changed two ways:

1.  *Per door:* Edit your instance's layout file, for example `PATH-ODE-ED/layouts/example.conf` and add the desired port for the DOOR-XROOTD in a separate line (a restart of the domain(s) running the DOOR-XROOTD is required):

        ..
        [xrootd-${host.name}Domain]
        [xrootd-${host.name}Domain/xrootd]
            port = 1095
        ..

2.  *Globally:* Edit `PATH-ODE-ED/dcache.conf` and add the variable `xrootd.net.port` with the desired value (a restart of the domain(s) running the DOOR-XROOTD is required):

        ..
        xrootd.net.port=1095
        ..

For controlling the TCP-portrange within which XROOTD-movers will start listening in the DOMAIN-POOL, you can add the properties `dcache.net.lan.port.min` and `dcache.net.lan.port.max` to `PATH-ODE-ED/dcache.conf` and adapt them according to your preferences. The default values can be viewed in `PATH-ODS-USD/defaults/dcache.properties`.

    ..
    dcache.net.lan.port.min=30100
    dcache.net.lan.port.max=30200
    ..

Quick tests
===========

The subsequent paragraphs describe a quick guide on how to test XROOTD using the `xrdcp` and `ROOT` clients.

Copying files with xrdcp
------------------------

A simple way to get files in and out of DCACHE via XROOTD is the command xrdcp. It is included in every xrootd and ROOT distribution.

To transfer a single file in and out of DCACHE, just issue

    PROMPT-USER xrdcp /bin/sh root://xrootd-door.example.org/pnfs/example.org/data/xrd_test
    PROMPT-USER xrdcp root://xrootd-door.example.org/pnfs/example.org/data/xrd_test /dev/null

Accessing files from within ROOT
--------------------------------

This simple ROOT example shows how to write a randomly filled histogram to a file in DCACHE:

    root [0] TH1F h("testhisto", "test", 100, -4, 4);
    root [1] h->FillRandom("gaus", 10000);
    root [2] TFile *f = new TXNetFile("root://door_hostname//pnfs/example.org/data/test.root","new");
    061024 12:03:52 001 Xrd: Create: (C) 2004 SLAC INFN XrdClient 0.3
    root [3] h->Write();
    root [4] f->Write();
    root [5] f->Close();
    root [6] 061101 15:57:42 14991 Xrd: XrdClientSock::RecvRaw: Error reading from socket: Success
    061101 15:57:42 14991 Xrd: XrdClientMessage::ReadRaw: Error reading header (8 bytes)

Closing remote XROOTD files that live in DCACHE produces this warning, but has absolutely no effect on subsequent ROOT commands. It happens because DCACHE closes all TCP connections after finishing a file transfer, while xrootd expects to keep them open for later reuse.

To read it back into ROOT from DCACHE:

    root [7] TFile *reopen = TXNetFile ("root://door_hostname//pnfs/example.org/data/test.root","read");
    root [8] reopen->ls();
    TXNetFile**             //pnfs/example.org/data/test.root
     TXNetFile*             //pnfs/example.org/data/test.root
      KEY: TH1F     testhisto;1     test

XROOTD security
===============

Read-Write access
-----------------

Per default DCACHE XROOTD is restricted to read-only, because plain XROOTD is completely unauthenticated. A typical error message on the clientside if the server is read-only looks like:

     PROMPT-USER xrdcp -d 1 /bin/sh root://ford.desy.de//pnfs/desy.de/data/xrd_test2
    Setting debug level 1
    061024 18:43:05 001 Xrd: main: (C) 2004 SLAC INFN xrdcp 0.2 beta
    061024 18:43:05 001 Xrd: Create: (C) 2004 SLAC INFN XrdClient kXR_ver002+kXR_asyncap
    061024 18:43:05 001 Xrd: ShowUrls: The converted URLs count is 1
    061024 18:43:05 001 Xrd: ShowUrls: URL n.1: root://ford.desy.de:1094//pnfs/desy.de/data/asdfas.
    061024 18:43:05 001 Xrd: Open: Access to server granted.
    061024 18:43:05 001 Xrd: Open: Opening the remote file /pnfs/desy.de/data/asdfas
    061024 18:43:05 001 Xrd: XrdClient::TryOpen: doitparallel=1
    061024 18:43:05 001 Xrd: Open: File open in progress.
    061024 18:43:06 5819 Xrd: SendGenCommand: Server declared: Permission denied. Access is read only.(error code: 3003)
    061024 18:43:06 001 Xrd: Close: File not opened.
    Error accessing path/file for root://ford//pnfs/desy.de/data/asdfas

To enable read-write access, add the following line to `${dCacheHome}/etc/dcache.conf`

    ..
    xrootdIsReadOnly=false
    ..

and restart any domain(s) running a DOOR-XROOTD.

Please note that due to the unauthenticated nature of this access mode, files can be written and read to/from any subdirectory in the PNFS namespace (including the automatic creation of parent directories). If there is no user information at the time of request, new files/subdirectories generated through XROOTD will inherit UID/GID from its parent directory. The user used for this can be configured via the `xrootd.authz.user` property.

Permitting read/write access on selected directories
----------------------------------------------------

To overcome the security issue of uncontrolled XROOTD read and write access mentioned in the previous section, it is possible to restrict read and write access on a per-directory basis (including subdirectories).

To activate this feature, a colon-seperated list containing the full paths of authorized directories must be added to `PATH-ODE-ED/dcache.conf`. You will need to specify the read and write permissions separately.

    ..
    xrootd.authz.read-paths=/pnfs/example.org/rpath1:/pnfs/example.org/rpath2
    xrootd.authz.write-paths=/pnfs/example.org/wpath1:/pnfs/example.org/wpath2
    ..

A restart of the DOOR-XROOTD is required to make the changes take effect. As soon as any of the above properties are set, all read or write requests to directories not matching the allowed path lists will be refused. Symlinks are however not restricted to these prefixes.

Token-based authorization
-------------------------

The XROOTD DCACHE implementation includes a generic mechanism to plug in different authorization handlers. The only plugin available so far implements token-based authorization as suggested in [].

The first thing to do is to setup the keystore. The keystore file basically specifies all RSA-keypairs used within the authorization process and has exactly the same syntax as in the native xrootd tokenauthorization implementation. In this file, each line beginning with the keyword `KEY` corresponds to a certain Virtual Organisation (VO) and specifies the remote public (owned by the file catalogue) and the local private key belonging to that VO. A line containing the statement `"KEY
	VO:*"` defines a default keypair that is used as a fallback solution if no VO is specified in token-enhanced XROOTD requests. Lines not starting with the `KEY` keyword are ignored. A template can be found in `PATH-ODS-USD/examples/xrootd/keystore`.

The keys itself have to be converted into a certain format in order to be loaded into the authorization plugin. DCACHE expects both keys to be binary DER-encoded (Distinguished Encoding Rules for ASN.1). Furthermore the private key must be PKCS \#8-compliant and the public key must follow the X.509-standard.

The following example demonstrates how to create and convert a keypair using OpenSSL:

    Generate new RSA private key
    PROMPT-ROOT openssl genrsa -rand 12938467 -out key.pem 1024

    Create certificate request
    PROMPT-ROOT openssl req -new -inform PEM -key key.pem -outform PEM -out certreq.pem

    Create certificate by self-signing certificate request
    PROMPT-ROOT openssl x509 -days 3650 -signkey key.pem -in certreq.pem -req -out cert.pem

    Extract public key from certificate
    PROMPT-ROOT openssl x509 -pubkey -in cert.pem -out pkey.pem
    PROMPT-ROOT openssl pkcs8 -in key.pem -topk8 -nocrypt -outform DER -out new_private_key
    PROMPT-ROOT openssl enc -base64 -d -in pkey.pem -out new_public_key

Only the last two lines are performing the actual conversion, therefore you can skip the previous lines in case you already have a keypair. Make sure that your keystore file correctly points to the converted keys.

To enable the plugin, it is necessary to add the following two lines to the file `PATH-ODE-ED/dcache.conf`, so that it looks like

    ..
    	xrootdAuthzPlugin=org.dcache.xrootd.security.plugins.tokenauthz.TokenAuthorizationFactory
    	xrootdAuthzKeystore=Path_to_your_Keystore
    	..

After doing a restart of DCACHE, any requests without an appropriate token should result in an error saying "`authorization check failed: No authorization token
	found in open request, access denied.(error code:
	3010)`".

If both tokenbased authorization and read-only access are activated, the read-only restriction will dominate (local settings have precedence over remote file catalogue permissions).

Strong authentication
---------------------

The XROOTD-implementation in DCACHE includes a pluggable authentication framework. To control which authentication mechanism is used by XROOTD, add the `xrootdAuthNPlugin` option to your DCACHE configuration and set it to the desired value.

For instance, to enable GSI authentication in XROOTD, add the following line to `PATH-ODE-ED/dcache.conf`:

    ..
    xrootdAuthNPlugin=gsi
    ..

When using GSI authentication, depending on your setup, you may or may not want DCACHE to fail if the host certificate chain can not be verified against trusted certificate authorities. Whether DCACHE performs this check can be controlled by setting the option `dcache.authn.hostcert.verify`:

    ..
    dcache.authn.hostcert.verify=true
    ..

*Authorization* of the user information obtained by strong authentication is performed by contacting the CELL-GPLAZMA service. Please refer to [???][1] for instructions about how to configure CELL-GPLAZMA.

> **Warning**
>
> In general GSI on XROOTD is not secure. It does not provide confidentiality and integrity guarantees and hence does not protect against man-in-the-middle attacks.

Precedence of security mechanisms
---------------------------------

The previously explained methods to restrict access via XROOTD can also be used together. The precedence applied in that case is as following:

> **Note**
>
> The XROOTD-door can be configured to use either token authorization or strong authentication with CELL-GPLAZMA authorization. A combination of both is currently not possible.

The permission check executed by the authorization plugin (if one is installed) is given the lowest priority, because it can controlled by a remote party. E.g. in the case of token based authorization, access control is determined by the file catalogue (global namespace).

The same argument holds for many strong authentication mechanisms - for example, both the GSI protocol as well as the KERBEROS protocols require trust in remote authorities. However, this only affects user *authentication*, while authorization decisions can be adjusted by local site administrators by adapting the CELL-GPLAZMA configuration.

To allow local site's administrators to override remote security settings, write access can be further restricted to few directories (based on the local namespace, the PNFS). Setting XROOTD access to read-only has the highest priority, overriding all other settings.

Other configuration options
---------------------------

The XROOTD-door has several other configuration properties. You can configure various timeout parameters, the thread pool sizes on pools, queue buffer sizes on pools, the XROOTD root path, the XROOTD user and the XROOTD IO queue. Full descriptions on the effect of those can be found in `PATH-ODS-USD/defaults/xrootd.properties`.

  [???]: #intouch-web
  []: http://people.web.psi.ch/feichtinger/doc/authz.pdf
  [1]: #cf-gplazma

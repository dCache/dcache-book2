The SRM Client Suite
====================

An SRM URL has the form <srm://dmx.lbl.gov:6253//srm/DRM/srmv1?SFN=/tmp/try1> and the file URL looks like <file:////tmp/aaa>.

srmcp
srmcp
Copy a file from or to an SRM or between two SRMs.
srmcp
option
sourceUrl
destUrl
sourceUrl  
The URL of the source file.

destUrl  
The URL of the destination file.

<!-- -->

`gss_expected_name`  
To enable the user to specify the gss expected name in the DN (Distinguished Name) of the srm server. The default value is `host`.

If the CN of host where srm server is running is `CN=srm/tam01.fnal.gov`, then `gss_expected_name` should be `srm`.

    PROMPT-USER srmcp --gss_expected_name=srm sourceUrl destinationUrl

`globus_tcp_port_range`  
To enable the user to specify a range of ports open for tcp connections as a pair of positive integers separated by “`:`”, not set by default.

This takes care of compute nodes that are behind firewall.

`globus_tcp_port_range=40000:50000` 

    PROMPT-USER srmcp --globus_tcp_port_range=minVal:maxVal sourceUrl destinationUrl

`streams_num`  
To enable the user to specify the number of streams to be used for data transfer. If set to 1, then stream mode is used, otherwise extended block mode is used.

    PROMPT-USER srmcp --streams_num=1 sourceUrl destinationUrl

`server_mode`  
To enable the user to set the (gridftp) server mode for data transfer. Can be `active` or `passive`, `passive` by default.

This option will have effect only if transfer is performed in a stream mode (see `streams_num`)

    PROMPT-USER srmcp --streams_num=1 --server_mode=active sourceUrl destinationUrl

Description
-----------

srmstage
srmstage
Request staging of a file.
srmstage
srmUrl
srmUrl  
The URL of the file which should be staged.

Description
-----------

Provides an option to the user to stage files from HSM to DCACHE and not transfer them to the user right away. This case will be useful if files are not needed right away at user end, but its good to stage them to dcache for faster access later.

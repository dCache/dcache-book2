Chapter 29. dCache Default Port Values 
======================================

You can use the command `dcache ports` to get the list of ports used by dCache.

| Port number     | Description                                                                                        | Component                                                                      |
|-----------------|----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| 32768 and 32768 | is used by the NFS layer within DCACHE which is based upon rpc. This service is essential for rpc. | NFS                                                                            |
| 1939 and 33808  | is used by portmapper which is also involved in the rpc dependencies of DCACHE.                    | portmap                                                                        |
| 34075           | is for postmaster listening to requests for the PSQL database for DCACHE database functionality.   | Outbound for **SRM**, PnfsDomain, dCacheDomain and doors; inbound for PSQL server. |
| 33823           | is used for internal DCACHE communication.                                                         | By default: outbound for all components, inbound for dCache domain.            |
| 8443            | is the `SRM` port. See [ Chapter 13, dCache Storage Resource Manager ](https://www.dcache.org/manuals/Book-2.16/config/cf-srm-fhs.shtml)                                                                         | Inbound for **SRM**                                                                |
| 2288            | is used by the web interface to DCACHE.                                                            | Inbound for httpdDomain                                                        |
| 22223           | is used for the DCACHE admin interface. See [ the section called “The Admin Interface”](https://www.dcache.org/manuals/Book-2.16/start/intouch-admin-fhs.shtml)                                               | Inbound for adminDomain                                                        |
| 22125           | is used for the DCACHE **dCap** protocol.                                                              | Inbound for **dCap** door                                                          |
| 22128           | is used for the DCACHE **GSIdCap** .                                                                   | Inbound for **GSIdCap** door                                                       |

  [???]: #cf-srm
  [1]: #intouch-admin

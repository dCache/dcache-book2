Part II. Configuration of dCache
================================

This part contains descriptions of the components of DCACHE, their role, functionality within the framework. In short, all information necessary for configuring them.


Table of Contents
-----------------

 + [4. Chimera](config-chimera.md)    
       
      [Mounting Chimera through NFS](config-chimera.md#mounting-chimera-through-nfs)   
      [Using dCap with a mounted file system](config-chimera.md#using-dcap-with-a-mounted-file-system)  
      [Communicating with Chimera](config-chimera.md#communicating-with-chimera)  
      [IDs](config-chimera.md#ids)    
      [Directory Tags](config-chimera.md#directory-tags)    
             
      [Create, List and Read Directory Tags if the Namespace is not Mounted](config-chimera.md)  
      [Create, List and Read Directory Tags if the Namespace is Mounted](config-chimera.md)  
      [Directory Tags and Command Files](config-chimera.md)  
      [Directory Tags for dCache](config-chimera.md)  
      [Storage Class and Directory Tags](config-chimera.md) 
 + [5. The Cell Package](config-cellpackage.md)    
 + [6. The replica Service (Replica Manager)](config-ReplicaManager.md)    
              
              [The Basic Setup(config-ReplicaManager.md)    
              [Define a poolgroup for resilient pools(config-ReplicaManager.md)      
              [Operation(config-ReplicaManager.md)      
              [Pool States(config-ReplicaManager.md)     
              [Startup(config-ReplicaManager.md)      
              [Avoid replicas on the same host(config-ReplicaManager.md)      
              [Hybrid dCache(config-ReplicaManager.md)      
              [Commands for the admin interface(config-ReplicaManager.md)      
              [Properties of the replica service(config-ReplicaManager.md) 
              
 + [7. The poolmanager Service](config-PoolManager.md)
 
             The Pool Selection Mechanism(config-PoolManager.md)   
Links(config-PoolManager.md)    
Examples(config-PoolManager.md)    
The Partition Manager(config-PoolManager.md)    
Overview(config-PoolManager.md)   
Managing Partitions(config-PoolManager.md)    
Using Partitions(config-PoolManager.md)    
Classic Partitions(config-PoolManager.md)   
Link Groups(config-PoolManager.md)  

 + [8. The dCache Tertiary Storage System Interface](config-hsm.md)    
Introduction  
Scope of this chapter  
Requirements for a Tertiary Storage System  
Migrating Tertiary Storage Systems with a file system interface.  
Tertiary Storage Systems with a minimalistic PUT, GET and REMOVE interface  
How dCache interacts with a Tertiary Storage System  
Details on the TSS-support executable  
Summary of command line options  
Summary of return codes  
The executable and the STORE FILE operation  
The executable and the FETCH FILE operation  
The executable and the REMOVE FILE operation  
Configuring pools to interact with a Tertiary Storage System  
The dCache layout files  
What happens next  
How to Store-/Restore files via the Admin Interface  
How to monitor what’s going on  
Log Files  
Obtain information via the dCache Command Line Admin Interface  
Example of an executable to simulate a tape backend  
 + [9. File Hopping](config-hopping.md)    
File Hopping on arrival from outside dCache  
File mode of replicated files  
File Hopping managed by the PoolManager  
File Hopping managed by the HoppingManager  
 + [10. Authorization in dCache](config-gplazma.md)    
Basics  
Configuration  
Plug-ins  
Using X.509 Certificates  
CA Certificates  
User Certificate  
Host Certificate  
VOMS Proxy Certificate  
Configuration files  
storage-authzdb  
The gplazmalite-vorole-mapping plug-in  
Authorizing a VO    
The kpwd plug-in  
The gridmap plug-in  
gPlazma specific dCache configuration  
Enabling Username/Password Access for WebDAV  
gPlazma config example to work with authenticated webadmin  
 + [11. dCache as xRootd-Server](config-xrootd.md)    
Setting up  
Parameters  
Quick tests  
Copying files with xrdcp  
Accessing files from within ROOT  
xrootd security  
Read-Write access  
Permitting read/write access on selected directories  
Token-based authorization  
Strong authentication  
Precedence of security mechanisms  
Other configuration options  
 + [12. dCache as NFSv4.1 Server](config-nfs.md)  
Setting up  
Configuring NFSv4.1 door with GSS-API support  
Configuring principal-id mapping for NFS access  
 + [13. dCache Storage Resource Manager](config-SRM.md)    
Introduction  
Configuring the srm service  
The Basic Setup    
Important srm configuration options  
Utilization of Space Reservations for Data Storage  
Properties of Space Reservation  
dCache specific concepts  
Activating SRM SpaceManager  
Explicit and Implicit Space Reservations for Data Storage in dCache  
SpaceManager configuration  
SRM SpaceManager and Link Groups  
Making a Space Reservation  
SRM configuration for experts  
Configuring the PostgreSQL Database  
SRM or srm monitoring on a separate node  
General SRM Concepts (for developers)  
The SRM service  
Space Management Functions  
Data Transfer Functions  
Request Status Functions  
Directory Functions  
Permission functions  
 + [14. The statistics Service](config-statistics.md)    
The Basic Setup  
The Statistics Web Page  
Explanation of the File Format of the xxx.raw Files  
 + [15. The billing Service](config-billing.md)   
The billing log files  
The billing database  
Customizing the database  
Generating and Displaying Billing Plots  
Upgrading a Previous Installation  
 + [16. The alarms Service](config-alarms.md)    
The Basic Setup  
Configure where the alarms service is Running  
Types of Alarms  
Alarm Priority  
Working with Alarms: Shell Commands  
Working with Alarms: Admin Commands  
Working with Alarms: The Webadmin Alarms Page  
Advanced Service Configuration: Enabling Automatic Cleanup  
Advanced Service Configuration: Enabling Email Alerts  
Advanced Service Configuration: Custom Alarm Definitions  
Miscellaneous Properties of the alarms Service  
 + [17. dCache Webadmin Interface](config-webadmin.md)  
Installation  
 + [18. ACLs in dCache](config-acl.md)  
Introduction  
Database configuration  
Configuring ACL support  
Enabling ACL support  
Administrating ACLs  
How to set ACLs  
Viewing configured ACLs  
 + [19. GLUE Info Provider](config-info-provider.md)  
Internal collection of information  
Configuring the info provider  
Testing the info provider  
Decommissioning the old info provider  
Publishing dCache information  
Troubleshooting BDII problems  
Updating information  
 + [20. Stage Protection](config-stage-protection.md)  
Configuration of Stage Protection  
Definition of the White List  
 + [21. Using Space Reservations without SRM](config-write-token.md)  
The Space Reservation    
The WriteToken tag    
Copy a File into the WriteToken    


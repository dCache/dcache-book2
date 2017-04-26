Part II. Configuration of dCache
================================

This part contains descriptions of the components of DCACHE, their role, functionality within the framework. In short, all information necessary for configuring them.


##Table of Contents

  [4. Chimera](https://www.dcache.org/manuals/Book-2.16/config/cf-chimera-fhs-comments.shtml)  
       [Mounting Chimera through NFS](https://www.dcache.org/manuals/Book-2.16/config/chimera-mount-fhs-comments.shtml)  
             [Using dCap with a mounted file system](https://www.dcache.org/manuals/Book-2.16/config/chimera-mount-fhs-comments.shtml#chimera-useDcap)  
       [Communicating with Chimera](https://www.dcache.org/manuals/Book-2.16/config/chimera-commands-fhs-comments.shtml)  
       [IDs]()  
       [Directory Tags]()  
            Create, List and Read Directory Tags if the Namespace is not Mounted
            Create, List and Read Directory Tags if the Namespace is Mounted
            Directory Tags and Command Files
            Directory Tags for dCache
            Storage Class and Directory Tags
5. The Cell Package
6. The replica Service (Replica Manager)
The Basic Setup
Define a poolgroup for resilient pools
Operation
Pool States
Startup
Avoid replicas on the same host
Hybrid dCache
Commands for the admin interface
Properties of the replica service
7. The poolmanager Service
The Pool Selection Mechanism
Links
Examples
The Partition Manager
Overview
Managing Partitions
Using Partitions
Classic Partitions
Link Groups
8. The dCache Tertiary Storage System Interface
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
9. File Hopping
File Hopping on arrival from outside dCache
File mode of replicated files
File Hopping managed by the PoolManager
File Hopping managed by the HoppingManager
10. Authorization in dCache
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
11. dCache as xRootd-Server
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
12. dCache as NFSv4.1 Server
Setting up
Configuring NFSv4.1 door with GSS-API support
Configuring principal-id mapping for NFS access
13. dCache Storage Resource Manager
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
14. The statistics Service
The Basic Setup
The Statistics Web Page
Explanation of the File Format of the xxx.raw Files
15. The billing Service
The billing log files
The billing database
Customizing the database
Generating and Displaying Billing Plots
Upgrading a Previous Installation
16. The alarms Service
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
17. dCache Webadmin Interface
Installation
18. ACLs in dCache
Introduction
Database configuration
Configuring ACL support
Enabling ACL support
Administrating ACLs
How to set ACLs
Viewing configured ACLs
19. GLUE Info Provider
Internal collection of information
Configuring the info provider
Testing the info provider
Decommissioning the old info provider
Publishing dCache information
Troubleshooting BDII problems
Updating information
20. Stage Protection
Configuration of Stage Protection
Definition of the White List
21. Using Space Reservations without SRM
The Space Reservation
The WriteToken tag
Copy a File into the WriteToken
Prev 	 	 Next
Files 	

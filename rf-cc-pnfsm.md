CELL-PNFSMNGR Commands
======================

pnfsidof
pnfsidof
Print the PNFS id of a file given by its global path.
pnfsidof
globalPath
Description
-----------

Print the PNFS id of a file given by its global path. The global path always starts with the “VirtualGlobalPath” as given by the “`info`”-command.

flags remove
flags remove
Remove a flag from a file.
flags remove
pnfsId
key
pnfsId  
The PNFS id of the file of which a flag will be removed.

key  
flags which will be removed.

Description
-----------

flags ls
flags ls
List the flags of a file.
flags ls
pnfsId
pnfsId  
The PNFS id of the file of which a flag will be listed.

Description
-----------

flags set
flags set
Set a flag for a file.
flags set
pnfsId
key
=
value
pnfsId  
The PNFS id of the file of which flags will be set.

key  
The flag which will be set.

value  
The value to which the flag will be set.

Description
-----------

metadataof
metadataof
Print the meta-data of a file.
metadataof
pnfsId
globalPath
-v
-n
-se
pnfsId  
The PNFS id of the file.

globalPath  
The global path of the file.

Description
-----------

pathfinder
pathfinder
Print the global or local path of a file from its PNFS id.
pathfinder
pnfsId
-global
-local
pnfsId  
The PNFS Id of the file.

-global  
Print the global path of the file.

-local  
Print the local path of the file.

Description
-----------

set meta
set meta
Set the meta-data of a file.
set meta
pnfsId
globalPath
uid
gid
perm
levelInfo
pnfsId  
The PNFS id of the file.

globalPath  
The global path oa the file.

uid  
The user id of the new owner of the file.

gid  
The new group id of the file.

perm  
The new file permitions.

levelInfo  
The new level information of the file.

Description
-----------

storageinfoof
storageinfoof
Print the storage info of a file.
storageinfoof
pnfsId
globalPath
-v
-n
-se
pnfsId  
The PNFS id of the file.

globalPath  
The global path oa the file.

Description
-----------

cacheinfoof
cacheinfoof
Print the cache info of a file.
cacheinfoof
pnfsId
globalPath
pnfsId  
The PNFS id of the file.

globalPath  
The global path oa the file.

Description
-----------

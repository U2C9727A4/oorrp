# Warning  
This document is currently marked as incomplete.  
  
# Object Oriented Remote Resource Protocol V1  
OORRP is a asynchronus RPC protocol mainly intended for embedded devices. It aims to have a minimal memory and CPU footprint.  
It is intended to be point-to-point with master-slave and master-master topology. (master-master explained in a later section.)  

# OORRP Request  
An OORRP request is a request with base headers and object-specific headers.  
It is fully self-contained and must be held fully in memory before parsing.  
## In-flight requests  
In-flight requests are requests that a server has not outputted a response to.

# OORRP Objects
An OORRP Object (can also be called a resource) is a command endpoint that exposes RPCs based on its type.  
All objects have two addresses;  
name,
oid (Object ID).
  
Name is a human-readable name of the object, in ASCII. It cannot be used for object RPCs.  
oid is defined below.  
Both name and OID is unique to their respective objects.
  
## Object ID  
Object IDs are permanent addresses of objects. OIDs persist for the lifetime of the server and after server reboots, resets etc.  
OIDs by their nature are unique to their object.  
OIDs are a uint64.
OID 0 (decimal) is reserved for root object.  

## Object types
Object types define what RPCs an object must implement.  
Object types are a uint16.

All object types have their own RPCs addresses by rpcid's. rpcid's are an uint16. rpcid 0 (decimal) is reserved for no-op.

Object types are to be defined by OORRP versions. Type 0 (decimal) is reserved for typeless objects.

## Object Hierarchy  
Objects follow a tree hierarchy. The root object is a special object that has itself as its parent.
### Root object
The root object is a special object that implements protocol RPCs, is typeless and has itself as its parent.  
The root object has the name "root" and OID of 0 (decimal) and a type of 0 (decimal).
Root object implements these RPCs:


# Request Base headers
(All integers in base headers are little endian.)  
Base headers are a special type of headers that do not change accross OORRP versions. (Left side comes first.)  
Base header definition:  
    rsiz[uint32], rid[uint8], ot[uint16], oid[uint64], rpcid[uint16]


# After this point is V1 specific.

TODO:
Topology types

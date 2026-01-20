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
OIDs by their nature are unique to their object. and is not hiererical.  
OIDs are a uint64.
OID 0 (decimal) is reserved for root object.  

## Object types
Object types define what RPCs an object must implement.  
Object types are a uint16.

All object types have their own RPCs addresses by rpcids. rpcids are an uint16. rpcid 0 (decimal) is reserved for no-op.
The highest bit of the rpcid is the response bit. If it is set, then the message is a response.  
The second highest bit of the rpcid is the error bit. If it is set, then the message is an error.
all RPCs other than special discovery RPCs (implemented by the root object) are not allowed to have variable lenght fields.

Object types are to be defined by OORRP versions. Type 0 (decimal) is reserved for typeless objects.
Typeless object should not implement any RPCs except the root object.

## Object Hierarchy  
Objects follow a tree hierarchy. The root object is a special object that has itself as its parent.
All objects must have a parent object.
### Root object
The root object is a special object that implements protocol RPCs, is typeless and has itself as its parent.  
The root object has the name "root" and OID of 0 (decimal) and a type of 0 (decimal).
Root object implements these RPCs (Tailing r means it is the response definition, the heading numbers are their RPCIDs in decimal.):  
TODO: Errors for each RPC
1: typeof. `oid[uint64]`. The typeof RPC is used to get the object type of the object at oid.
1r: typeof. `otype[uint16]`. otype represents the object's type. Little endian.  

2: oidof. `name_len[uint32], name[name_len]` name_len represents the lenght of the name. name represents the lenght of the name. returns the OID of the queried object. name_len is little endian.
2r: oidof `oid[uint64]` oid represents the object ID of the queried object. oid is little endian.  

3: parentof. `oid[uint64]`, Returns the parent OID of the object at oid. oid is little endian.  
3r: parentof. `oid[uint64]` oid represents the object ID of the parent object. oid is little endian.  

4: child_amountof. `oid[uint64]` Returns the amount of children the object at oid has. oid is little endian.  
4r: child_amountof. `children_amount[uint64]` children_amount represents the amount of children the queried object has. children_amount is little endian.  

5: get_children. `oid[uint64], 




# Base headers
(All integers in base headers are little endian.)  
Base headers are a special type of headers that do not change accross OORRP versions. (Left side comes first.)  
Base header definition:  
    rsiz[uint32], rid[uint8], oid[uint64], rpcid[uint16], otype[uint16], crc[uint32], ... (Additional headers based on object type and RPCID)

rsiz represents the full message size. This size includes rsiz's space aswell.
rid is the message ID of the message. It is only used to corrolate request-reponse pairs. Due to this, it only applies to messages that are in-flight. message ID collisions are allowed as long as the colliding messages are not in-flight at the same time.


# After this point is V1 specific.

TODO:
Topology types

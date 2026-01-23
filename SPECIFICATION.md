# Warning  
This document is currently marked as incomplete.  
  
# Object Oriented Remote Resource Protocol  
OORRP is a asynchronus RPC protocol mainly intended for embedded devices. It aims to have a minimal memory and CPU footprint.  
It is intended to be a control plane with point-to-point, master-slave and master-master topology. (master-master explained in a later section.)  

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

## Object versioning
OORRP is defined atomically. Objects, RPCs and error values may not be updated independently.  
  
## Object ID  
Object IDs are permanent addresses of objects. OIDs persist for the lifetime of the server and after server reboots, resets etc.  
OIDs by their nature are unique to their object.  
OIDs are a uint64.
OID 0 (decimal) is reserved for root object.  

## Object types
Object types define what RPCs an object must implement.  
Object types are a uint16.

All object types have their own RPC addresses by rpcids. rpcids are an uint16. rpcid 0 (decimal) is reserved for no-op.
RPCIDs are unique to their RPC functions.
The highest bit of the rpcid is the response bit. If it is set, then the message is a response.  
The second highest bit of the rpcid is the error bit. If it is set, then the message is an error.
Due to the nature of errors, an error message *must* be a response message aswell.
all RPCs other than special discovery RPCs (implemented by the root object) are not allowed to have variable lenght fields.

### RPC Errors
All errors follow this definition: `err[uint32]` err is the error code. err is little endian.  
Error codes are documented below.

Object types are to be defined by OORRP versions. Type 0 (decimal) is reserved for root object.

## Object Hierarchy  
OORRP has a flat object hierachy. This is a design decision to prevent variable-lenght RPC arguements and responses. 
### Root object
The root object is a special optional object that implements protocol RPCs, and is typeless.
For servers that do not implement a root object, the root OID must not have an object at it and return no object error.
The root object has the name "root" and OID of 0 (decimal) and a type of 0 (decimal).
Root object implements these RPCs (Tailing r means it is the response definition, the heading numbers are their RPCIDs in decimal.):  
TODO: Errors for each RPC
1: typeof. `oid[uint64]`. The typeof RPC is used to get the object type of the object at oid.
1r: typeof. `otype[uint16]`. otype represents the object's type. Little endian.  
1re: 
    RPC Error point + 1: Object does not exist.

2: oidof. `name_len[uint32], name[name_len]` name_len represents the lenght of the name in bytes.. name represents the lenght of the name. returns the OID of the queried object. name_len is little endian.
2r: oidof `oid[uint64]` oid represents the object ID of the queried object. oid is little endian.  
2re: 
    RPC error point + 1: Name too long.  
    RPC error point + 2: No object with name exists.


3: nameof `oid[uint64]`. Returns the name of the object at oid. oid is little endian.  
3r: nameof. `name_len[uint32], name[name_len]`. name_len represent the lenght of the name in bytes. name is the name of the queried object. name_len is little endian.  
3re:
    RPC error point + 1: Object does not exist.

4: sizeof. `Adds no additional headers.` Returns the maximum message size the OORRP server can handle.  
4r: sizeof. `size[uint64]`. size represents the maximum message size the server can handle. size is little endian.  
4re: 
    (This RPC cannot fail.)

5: get_object_amount. `Adds no additional headers.`. returns the amount of objects the server has.
5r: get_object_amount. `obj_amount[uint64]`. obj_amount represents the amount of objects the server has. obj_amount is little endian.
5re:
    (This RPC cannot fail.)

6: get_oid `i[uint64]`. Gets the OID of the i'th object. (This object table must be stable as it is used for discovery.) i is little endian.  
6r: get_oid. `oid[uint64]` oid represents the object id of the object. oid is little endian.
6re:
    RPC error point + 1: No object at position i.

7: get_inflight_size. `Adds no additional headers.` Returns the maximum amount of in-flight messages the server can handle for the querying client.  
7r: get_inflight_size `size[uint8]` size represents the maximum in-flight messages the server can handle for this client. 
7re:
    (This RPC cannot fail.)  

8: promote `Adds no additional headers`. Promotes the server from a slave into master.  
8r: promote. `Adds no additional headers.` Server successfully promoted to master.  
8re:
    RPC error point + 1: Underlying transport failure.
    RPC Error point + 2: Promotion refused.

## Object lifetimes
An object at an OID may not dynamically change its type. The only change that can happen is for the object to be deleted. Any other change during its life is not allowed.
Objects may dynamically appear and disappear, but must obey the OID rule of having a set one accross its lifetimes.

# Base headers
(All integers in base headers are little endian.)  
Base headers are a special type of headers that do not change accross OORRP versions. (Left side comes first.)  
Base header definition:  
    rsiz[uint32], rid[uint8], oid[uint64], rpcid[uint16], otype[uint16], crc[uint32], maver[uint8], miver[uint8] ... (Additional headers based on object type and RPCID)

rsiz represents the full message size. This size includes rsiz's space aswell.
rid is the message ID of the message. It is only used to corrolate request-reponse pairs. Due to this, it only applies to messages that are in-flight. message ID collisions are allowed as long as the colliding messages are not in-flight at the same time.
oid is the object ID to be operated on.  
rpcid is the RPCID of the object.
otype is the object type of the object.
crc is the CRC of the message. It is to be filled with zero when the CRC is being computed. It follows the CRC-32 standard.
maver is the major version.  
miver is the minor version.

## rid and async
RID applies *per client*. If a server is handling multiple clients, RID collisions among clients are allowed.  
Messages follow a FCFS queue, but it is not enforced. Implementations are free to prioritize operations on some objects over others. Due to this, clients cannot expect responses to come in order.

# Errors
Errors in OORRP are an uint32. with this layout:
(numbers are decimal)
Error codes ranging from 0 to 1000 are transport errors
0: Invalid CRC.
1: Malformed Message
2: Desync recovery failed.

Error codes ranging from 1000 to 2000 are message errors.
1000: Invalid RID
1001: Invalid OID
1002: Invalid RPCID
1003 Invalid Object Type.
1004: Object does not exist.
1005: Request size exceeds buffer limits.

Error codes after `2^32 - 1 / 2` (2_147_483_647 decimal) (This value may be referred as "RPC Error Point" in this document.) are open for use by RPCs defined by current and later versions of OORRP. But before that point it is reserved for OORRP.
After the RPC error point, error codes are tied to their RPCs. Due to this diffirent RPCs can have colliding error codes.

# Master-Slave and Master-Master  
Master-Slave is the default operating mode. The client is the master, the server is the slave. Only the client can initiate requests.  
Master-Master is the secondary operating mode. Both sides are masters. In this mode, both sides can make requests to eachother. This is made possible by the fact that the only diffirence between a request and response message is just a response bit in the RPCID. However, not all transports can support this, due to this the slave is free to refuse being promoted to a master.

# Retries
When a message errors out with a invalid CRC error, It must be retried for a minimum of 5 times before the client gives up. The failed message must be sent identically.

# Transport
OORRP is Transport-agnostic. It only expects a:
Point to point
Ordered
byte-stream transport.

However, these expections do not have to be fully met by the transport. These requirement can also be met by a HAL layer. Message-based transports can still be used if the HAL can do fragmentation and eventually convert it into a byte-stream.
# After this point is version specific.
# OORRP version V1.0

+TODO:
+Topology types
# Object types
## Digital Pin object. (Object type 1)  
The digital pin object is exactly as it sounds; It represents a digital pin.TO
### RPCs
(All integers are little endian., the heading numbers are RPCIDs in decimal, tailing r means response, tailing e means error.)
1: set_state. `state[uint8]`. Sets the state of the pin. state of 1 (decmial) means the pin should be at a logical 1. state of 0 means the pin should be at a logical 0.  
1r: set_state. `Adds no additional headers.` Only tells the operation has completed.  
1re: This RPC cannot fail.

TODODP: get_state, set_mode (out, in, modifier pull-down and pullup), get_mode

TODOV1:
-Digital pin object
PWM pin object
ADC object
DAC object
AxisLinearActuator object
RotatingActuator object
Heater object

---
layout: post
title: "Paper reviews: Implementing Remote Procedure Calls "
date:   2019-11-18
author: Faushine
tags: 
- Distributed system
---

# RPC

[Remote procedure call](/img/in-post/2019-11-18/RPC.pdf) is an useful paradigm for providing communication across a network between programs and it is written in a high-level language. There are lots of open source RPC framework such as [gRPC](https://grpc.io/) and [Dubbo](https://dubbo.apache.org/en-us/).

## Modes of Communication

- Raw message passing: Low level, and not easy to use.
  
    – TCP/IP: reliable stream

    – UDP: unreliable, connectionless, packet oriented.

- Distributed shared memory: higher overhead, and failure handling is difficult

- File system: Low level and slow.
  
## Working of RPC

When making a remote call, five pieces of program are involved: the user, the user-stub, the RPC communications package (known as RPCRuntime), the server-stub, and the server. The user, the user-stub, and one instance of RPCRuntime execute in the caller machine; the server, the server-stub and another instance of RPCRuntime execute in the callee machine.

![alt text](/img/in-post/2019-11-18/rpc.jpg)

```
Interface{   	
    // has enough information for compile time    
 	// checking, and generating proper calling sequence
    int func1(int arg)
    int func2(char * buffer, int size)
}
```

### func1

Client stub for func1

```
int func1(int arg) {
    Create req
    Pack  fid, arg, …etc
    Send req
    Wait for reply
    Unpack results
    Return result
}
 
```

Server stub for func1

```
Server stub fun1{
	Unpack request
 	Find the requested server function
	Call the function
	Pack results
	Send results
}
```

### func2

problems:

- *buffer points to local address space, and invocation on a remote node: copy/restore if needed.
- Semantic is not clear can be input or output: add an argument type (in|out|both).
- Complex data structures

## Binding

There are two aims to binding which we consider in turn. How does a client of the binding mechanism specify what he wants to be bound to? How does a caller determine the machine address of the callee and specify to the callee the procedure to be invoked ? Basically the first is solved by **naming** and the second is solved by **locating**.

> Naming: binding an importer of an interface to an exporter of an interface. There are two parts to the name of an interface: the type and the instance.

> Locating: can be a network address that allow the importer to bind and communicate to the exporter.

System described in the paper uses Grapevine (a distributed database) for RPC binding. Client gets the address from Grapevine, then uses the address and function id to invoke the
procedure. Server returns Table index and function ID to the client. Client tries to invoke the function using table index, fun ID, call ID and args

There are 3 different types of binding:

- Importer specifies only the type of the interface, the instance is chosen dynamically. 

- Importer specifies that RName, which delays the resolution of the actual instance. 

- Importer explicitly specifies the instance that it wants to run the RPC on.


Example: RPC-based file system

![alt text](/img/in-post/2019-11-18/rpc-server.jpg)

![alt text](/img/in-post/2019-11-18/rpc-client.jpg)


## RPC-Runtime

![alt text](/img/in-post/2019-11-18/rpc-run.jpg)

Runtime choices:

- TCP: High overhead in terms of latency and server state (having to keep several open TCP connections).
  
- UDP: Unreliable, doesn’t wait for acknowledgements of packet reception.

## Call semantics

- At least once: the function is executed at least once.

- At most once: the function is executed once or not at all

## Fault tolerance

**1. Lost reply from server**

![alt text](/img/in-post/2019-11-18/rpc-lost.jpg) 

**Solution**: Call ID used to enforce **precisely once** semantics of procedure execution. Call sequence numbers are strictly increasing. Each calling activity has a latest call ID stored on callee machines. All the calls made to the callee machines should have Call IDs greater than this number, else the precisely once semantics will be violated.

![alt text](/img/in-post/2019-11-18/rpc-once.jpg) 

![alt text](/img/in-post/2019-11-18/rpc-onesolv.jpg) 

**2. Client failure with repeated seq**

Client rebooted before receiving result and then call func1() with different sequence number. fun1() was acked the second time without execution.

![alt text](/img/in-post/2019-11-18/rpc-ack.jpg) 

**Solution**: add clock based conversation id, to every call. *<Conversation_id, seq#>* is strictly increasing.

**3. Server crash**

Server updated prev.calls table and then crashed.
 
![alt text](/img/in-post/2019-11-18/rpc-scrash.jpg) 

**Solution**: Rebind on server restart with new fid.

![alt text](/img/in-post/2019-11-18/rpc-scsolv.jpg)

## Demo

[gRPCdemo](https://github.com/faushine/gRPCdemo) implemented greet and streaming calls.

## Reference

[CS 754: Advanced Distributed Systems](https://cs.uwaterloo.ca/~alkiswan/Classes/CS754-19F/)


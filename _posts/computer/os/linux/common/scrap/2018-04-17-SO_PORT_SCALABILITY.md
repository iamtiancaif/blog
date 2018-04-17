---
layout: post
title:  "SO_PORT_SCALABILITY"
categories: computer
tags:  computer linux java jixianzhao
author: web
source: https://msdn.microsoft.com/en-us/library/cc150670(v=vs.85).aspx
ignore: true
---

* content
{:toc}


SO\_PORT\_SCALABILITY
=====================

The **SO\_PORT\_SCALABILITY** socket option enables local port scalability for a socket.

**SO\_PORT\_SCALABILITY**

0x3006

The **SO\_PORT\_SCALABILITY** socket option enables local port scalability by allowing port allocation to be maximized by allocating wildcard ports multiple times for different local address port pairs on a local machine.

Remarks
-------

Note: on platforms where both SO\_PORT\_SCALABILITY and SO\_REUSE\_UNICASTPORT are supported, prefer to use SO\_REUSE\_UNICASTPORT.

Proxy server environments have scalability issues because of limited local port availability. One way to work around this is to add more IP addresses to the machine. However, by default wildcard ports used with the [**bind**](https://msdn.microsoft.com/en-us/library/ms737550(v=vs.85).aspx) function are limited to the size of the dynamic port range on the local machine (up to 64K ports, but usually less) no matter the number of IP addresses on the local machine. Working around this requires the application to maintain its own port pool either with port reservation or by using heuristics.

To avoid having every application that requires scalability manage its own port pool, and to allow for greater scalability while maintaining application compatibility, Windows Server 2008 introduced the **SO\_PORT\_SCALABILITY** socket option to help maximize wildcard port allocation. Port allocation is maximized by allowing an application to allocate wildcard ports for each unique local address and port pair. So if a local machine has four IP addresses, then up to 256 K wildcard ports (64 K ports × 4 IP addresses) can be allocated by wildcard [**bind**](https://msdn.microsoft.com/en-us/library/ms737550(v=vs.85).aspx) function requests.

When the **SO\_PORT\_SCALABILITY** socket option is set on a socket and a call to the [**bind**](https://msdn.microsoft.com/en-us/library/ms737550(v=vs.85).aspx) function is made for a specified address and wildcard port (the _name_ parameter is set with a specific address and a port of 0), Winsock will allocate a port for the specified address. This allocation will be based on all of the possible IP addresses and ports/per address on the local computer. If a wildcard port is acquired using the **SO\_PORT\_SCALABILITY**option, that port cannot be allocated by another socket without the **SO\_PORT\_SCALABILITY** option. This restriction is in place to avoid backward-compatibility problems with applications that assume a wildcard local port cannot be reused. Note that this means that applications which acquire a large number of ports using **SO\_PORT\_SCALABILITY** can starve legacy applications of ports. If all available ephemeral ports have been acquired for at least one address with **SO\_PORT\_SCALABILITY** , then no more wildcard port allocations are possible without the socket option.

To have any effect, the **SO\_PORT\_SCALABILITY** option must be set before the [**bind**](https://msdn.microsoft.com/en-us/library/ms737550(v=vs.85).aspx) function is called. An example of how this would be used on a computer with two addresses is outlined below:

*   The [**socket**](https://msdn.microsoft.com/en-us/library/ms740506(v=vs.85).aspx) function is called by a process to create a socket.
*   The [**setsockopt**](https://msdn.microsoft.com/en-us/library/ms740476(v=vs.85).aspx) function is called to enable the **SO\_PORT\_SCALABILITY** socket option on the newly created socket.
*   The [**bind**](https://msdn.microsoft.com/en-us/library/ms737550(v=vs.85).aspx) function is called to do a bind on one of the local computer's IP addresses and port 0.
*   The [**connect**](https://msdn.microsoft.com/en-us/library/ms737625(v=vs.85).aspx) function is then called to connect to a remote IP address. The socket is used by the application as needed.
*   A [**socket**](https://msdn.microsoft.com/en-us/library/ms740506(v=vs.85).aspx) function is called by the same process (possibly a different thread) to create a second socket.
*   The [**setsockopt**](https://msdn.microsoft.com/en-us/library/ms740476(v=vs.85).aspx) function is called to enable the **SO\_PORT\_SCALABILITY** socket option on the newly created second socket.
*   The [**bind**](https://msdn.microsoft.com/en-us/library/ms737550(v=vs.85).aspx) function is called with the local computer's second IP address and port 0. Even when all ports have been previously allocated, this call succeeds because there are multiple IP addresses available on the local computer and the **SO\_PORT\_SCALABILITY** socket option was set on both sockets in the same process.
*   The [**connect**](https://msdn.microsoft.com/en-us/library/ms737625(v=vs.85).aspx) function is then called to connect to a remote IP address. The second socket is used by the application as needed.

Requirements
------------

Minimum supported client

None supported

Minimum supported server

Windows Server 2008 \[desktop apps only\]

Header

Ws2def.h

See also
--------

[**getsockopt**](https://msdn.microsoft.com/en-us/library/ms738544(v=vs.85).aspx)

[**setsockopt**](https://msdn.microsoft.com/en-us/library/ms740476(v=vs.85).aspx)

[**SOL_SOCKET Socket Options**](https://msdn.microsoft.com/en-us/library/ms740532(v=vs.85).aspx)

[**Socket Options**](https://msdn.microsoft.com/en-us/library/ms740525(v=vs.85).aspx)






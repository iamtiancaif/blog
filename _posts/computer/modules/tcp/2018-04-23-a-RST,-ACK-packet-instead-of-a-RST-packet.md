---
layout: post
title:  "a RST, ACK packet instead of a RST packet"
categories: computer
tags:  computer tcp stackexchange
author: web
source: https://networkengineering.stackexchange.com/questions/2012/why-do-i-see-a-rst-ack-packet-instead-of-a-rst-packet
ignore: true
---

* content
{:toc}


Q:
--

Looking in Wireshark, I often see TCP Streams end with a RST, ACK packet instead of a RST packet. Anyone know why this is?

An example of what I see:

SYN SYN, ACK ...data... RST, ACK

Wireshark is not picking up a RST packet prior to the RST, ACK packet.

  

  

A1:
---

A RST/ACK is not an acknowledgement of a RST, same as a SYN/ACK is not exactly an acknowledgment of a SYN. TCP establishment actually is a four-way process: Initiating host sends a SYN to the receiving host, which sends an ACK for that SYN. Receiving host sends a SYN to the initiating host, which sends an ACK back. This establishes stateful communication.

    SYN --> 
        <-- ACK
        <-- SYN
    ACK -->
    

To make this more efficient, the receiving host can ACK the SYN, and send its own SYN in the same packet, creating the three-way process we are used to seeing.

    SYN -->
        <-- SYN/ACK
    ACK -->
    

In the case of a RST/ACK, The device is acknowledging whatever data was sent in the previous packet(s) in the sequence with an ACK and then notifying the sender that the connection has closed with the RST. The device is simply combining the two packets into one, just like a SYN/ACK. A RST/ACK is usually not a normal response in closing a TCP session, but it's not necessarily indicative of a problem either.

  

A2:
---

Once the connection is established, all packets need to have ACK set and match the sequence number of the received packets for reliable transport/security. RST without ACK will not be accepted. When one side sends RST, the socket is closed immediately and the receiving side also closes the socket immediately after receiving valid RST. It does not need to be and can't be acknowledged.

after TCP handshake

A --->B Syn=x, Ack=y, len=z, ACK Flag

B --->A Syn=y, Ack=x+z, len=o, ACK Flag

A --->B Syn=x+z, Ack=y+o, len=p, ACK Flag

B --->A Syn=y+o, ACK=x+z+p,len=q, RST, ACK Flag

B closes the socket after it sends the last packet and A closes the socket after it receives it.

(not considering TCP window here, or there might be more packets from one end before the acknoledgement)

ACK Flag, acknowledgement number and the procedure of acknowledgement are related but not the same thing.

[Per RFC793](https://www.ietf.org/rfc/rfc793.txt)

> RFC793

Acknowledgment Number: 32 bits

    If the ACK control bit is set this field contains the value of the
    next sequence number the sender of the segment is expecting to
    receive.  Once a connection is established this is always sent.
    

Reset Processing

In all states except SYN-SENT, all reset (RST) segments are validated by checking their SEQ-fields. A reset is valid if its sequence number is in the window. In the SYN-SENT state (a RST received in response to an initial SYN), the RST is acceptable if the ACK field acknowledges the SYN.

The receiver of a RST first validates it, then changes state. If the receiver was in the LISTEN state, it ignores it. If the receiver was in SYN-RECEIVED state and had previously been in the LISTEN state, then the receiver returns to the LISTEN state, otherwise the receiver aborts the connection and goes to the CLOSED state. If the receiver was in any other state, it aborts the connection and advises the user and goes to the CLOSED state.






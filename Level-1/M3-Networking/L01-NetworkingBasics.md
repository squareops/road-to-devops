# Networking Basics:

Let's forget the DevOps side of things to begin with here, we now need to look very briefly into some of the Networking fundamentals.
## Network Devices 
Host are any devices which send or receive traffic.

![](Images/n1.png)

IP Address the identity of each host.

![](Images/n2.png)

Network is what transports traffic between hosts. If we did not have networks there would be a lot of manual movement of data!

A logical group of hosts which require similar connectivity.

![](Images/n3.png)

Switches facilitate communication within a network. A switch forwards data packets between hosts. A switch sends packets directly to hosts.
- Network: A Grouping of hosts which require similar connectivity.
- Hosts on a Network share the same IP address space.
 
Routers facilitate communication between networks. If we said before that a switch looks after communication within a network a router allows us to join these networks together or at least give them access to each other if permitted.
A router can provide a traffic control point (security, filtering, redirecting) More and more switches also provide some of these functions now.
Routers learn which networks they are attached to. This is known as routes, a routing table is all the networks a router knows about.
A router has an IP address in the networks they are attached to. This IP is also going to be each host's way out of their local network also known as a gateway.
Routers also create the hierarchy in networks I mentioned earlier.

![](Images/n4.png)

## Switches vs Routers
Routing is the process of moving data between networks.
- A router is a device whose primary purpose is Routing.
Switching is the process of moving data within networks.
- A Switch is a device whose primary purpose is switching.

This is very much a foundational overview of devices as we know there are many different Network Devices such as:
- Access Points
- Firewalls
- Load Balancers
- Layer 3 Switches
- IDS / IPS
- Proxies
- Virtual Switches
- Virtual Routers

Although all of these devices are going to perform Routing and/or Switching.


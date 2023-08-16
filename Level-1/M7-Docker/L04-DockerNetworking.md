# Docker Networking
Using Docker Networking, an isolated package can be communicated. Docker contains the following network drivers -

- Bridge - Bridge is a default network driver for the container. It is used when multiple docker communicates with the same docker host.
- Host - It is used when we don't need for network isolation between the container and the host.
- None - It disables all the networking.
- Overlay - Overlay offers Swarm services to communicate with each other. It enables containers to run on the different docker host.

![](Images/docker20.png)

## For more information refer the following

[Docker Networking Overview](https://docs.docker.com/network/)
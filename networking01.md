# Networking 01

## How the internet works

1. First there is this thing called Optical Fibres that are owned by the tier 1 companies.
2. Then we have tier 2 companies that rent the optical fibre service from the tier 1 companies.
3. The data transfer takes place through these optical fibres (so they are very crucial).
4. Now we have something called WAN, MAN, LAN, PAN.

### WAN
> This refers to Wide Area Network and caters to the global network connection.

### MAN
> This refers to Metropolitan Area Network that takes into account a particular city.
> How connection works in that particular city.

### LAN
> Stands for Local Area Network and takes into account the network of the local environment, such as an office or a particular building.

### PAN
> Stands for Personal Area Network. This is for the connection we create by our pairable devices like bluetooth headphones, mic, screens, etc.

## OSI Model (Open System Interconnection)

> This is theoretically how the internet works.

* Application — S
* Presentation — S
* Session — S
* Transport (TCP/UDP) — S
* Network — H
* Data Link Layer — H
* Physical Layer (Optical Fibres) — H

Above H -> Hardware
Above S -> Software

## TCP/IP Model

> This is practically how the internet works.

* Application
* Transport
* Internet
* Network

## Protocols

> Application Layer -> SMTP, HTTP, HTTPS, FTP
> Transport Layer -> TCP (Transmission Control Protocol, ex: Mail Transfer, Chatting) / UDP (User Datagram Protocol, ex: Video Streaming, Gaming)
> Internet Layer -> Internet Protocol (IP), this brings the internet to our devices, handles routing the movement of data packets from one network to another.

## IP Address and Subnet

> IP Addr -> The unique number assigned to each device is called an IP addr, can be IPv4 or IPv6.
> Subnet -> This solves the problem of limited IP addr by creating a VPC (Virtual Private Cloud).
> To solve the issue of blocked sites on the wifi, we can use — or we must have used — a VPN (Virtual Private Network).

## MAC Address

> On the local level, if we need to block or provide access to a device for certain stuff, or we need to identify the device on the local network, we use MAC Address.
> We cannot use IP addr because it can change if we switch or change the wifi or router, so we rely on the **MAC addr** to uniquely identify the device.

## Router VS Switches

> Router is used when we need to access the internet from the ISP or MAN. The router connects our LAN to other networks and forwards data between them.
> We then connect our devices to the router, either directly or through a switch.
> Switch is used in LAN where we need to connect multiple devices together, often used in offices. It forwards data between devices using their MAC address.

## On-Premise VS Cloud
 
> There is a company that has multiple servers, and on that server we have multiple apps running, be it database, frontend, backend, and so on. On-premise means that the company itself will have the physical hardware, AC, and other blocks that it will maintain.
 
> Whereas big tech giants like GCP, AWS, Microsoft Azure have a lot of servers and data centers and rent us the server, so we don't handle the physical hardware — we just SSH into the server and use it, according to our needs.



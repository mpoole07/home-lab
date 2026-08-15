# Networking Documentation

## 1. Overview

The home lab uses the existing home router as the primary network gateway, DHCP server, and Internet connection.

The HP ProDesk server is connected to the network through Ethernet and hosts several services that are accessible from other devices on the local network and, where required, from the Internet.

The basic network architecture is:

```text
                         INTERNET
                             |
                             v
                       Home Router
                     /      |       
                    /       |        
                   v        v        
                Wi-Fi    Ethernet 
                   |        |     
                   |        |
                   |        v
                   |    HP ProDesk
                   |        |
                   |        +-- Ubuntu
                   |        +-- Minecraft
                   |        +-- Jellyfin
                   |        +-- NAS
                   |        +-- BlueMap
                   |
                   +-- PCs
                   +-- Roku
                   +-- Phones
```

---

# 2. Server Network Connection

The server uses a wired Ethernet connection.

This was chosen over Wi-Fi because the server provides services that benefit from:

* Stable connectivity
* Lower latency
* Higher throughput
* Fewer wireless interruptions
* Reliable long-term operation

The Ethernet interface is managed by Ubuntu.

Useful commands for checking the network interface include:

```bash
ip a
```

and:

```bash
ip link
```

To view routing information:

```bash
ip route
```

---

# 3. Local IP Address

The HP ProDesk was assigned a static local IP address using a DHCP reservation on the home router.

This is preferable to manually configuring a static address inside Ubuntu because the router remains responsible for assigning and tracking the address.

---

# 4. Why the Static Local IP Is Important

Several services depend on the server having a predictable IP address.

For example:

```text
<SERVER_IP>:8097
```

is used to access Jellyfin.

Other services include:

| Service                  | Protocol |  Port |
| ------------------------ | -------- | ----: |
| SSH                      | TCP      |    22 |
| Minecraft Java           | TCP      | 25565 |
| Minecraft Bedrock/Geyser | UDP      | 19132 |
| Jellyfin                 | TCP      |  8097 |

Because the server's local IP does not change, router port-forwarding rules can continue pointing to the correct machine.

---

# 5. DHCP

The home router provides DHCP services for the local network.

The server receives its reserved IP address from the router.

Other devices, such as:

* PCs
* laptops
* phones
* Roku
* other clients

can receive addresses dynamically.

The network therefore uses a combination of:

```text
DHCP
+
DHCP reservations
```

rather than manually configuring every device.

---

# 6. Default Gateway

The home router acts as the default gateway for the server.

Traffic intended for devices outside the local subnet is sent to the router.

The routing structure is conceptually:

```text
Server
  |
  | Local traffic
  v
LAN devices

Server
  |
  | Non-local traffic
  v
Default Gateway
  |
  v
Internet
```

The current routing table can be inspected with:

```bash
ip route
```

---

# 7. DNS

DNS is used to translate hostnames into IP addresses.

The server can use the router for normal Internet name resolution.

For example:

```text
example.com
     |
     v
DNS
     |
     v
IP address
```

DNS is also useful for the Minecraft server because a domain name can be used instead of requiring players to remember a public IP address.

---

# 8. Minecraft Local Networking

Minecraft Java clients on the local network connect directly to the server's local IP.

Example:

```text
<SERVER_IP>:25565
```

The standard Minecraft Java port is:

```text
25565/TCP
```

The local connection path is:

```text
Minecraft Client
      |
      v
Home Router / LAN
      |
      v
HP ProDesk
      |
      v
Paper Minecraft Server
```

No Internet connection is required for players on the same LAN to communicate with the server.

---

# 9. Minecraft Internet Access

The Minecraft server was configured to accept connections from outside the local network.

This required port forwarding on the home router.

The router forwards:

```text
Internet
   |
   v
Public IP
   |
   | TCP 25565
   v
<SERVER_IP>:25565
```

The server therefore acts as the destination for incoming Minecraft connections.

The server was successfully tested from an external network.

---

# 10. Bedrock Networking

Bedrock compatibility was added through Geyser and Floodgate.

The Java server itself continues to run Paper.

The connection architecture is:

```text
Bedrock Client
      |
      | UDP 19132
      v
Home Router
      |
      v
HP ProDesk
      |
      v
Geyser
      |
      v
Paper
      |
      v
Minecraft World
```

The Bedrock port is:

```text
19132/UDP
```

A separate router forwarding rule is required because Java and Bedrock use different network ports/protocols.

---

# 11. Geyser and Floodgate

### Geyser

Geyser provides protocol translation between Bedrock clients and the Java server.

```text
Bedrock protocol
       |
       v
    Geyser
       |
       v
Java protocol
       |
       v
    Paper
```

### Floodgate

Floodgate provides Bedrock authentication support.

This allows Bedrock users to join the Java server without requiring a Java Edition account.

---

# 12. Minecraft Connection Troubleshooting

When external Minecraft connections initially failed, the server reported:

```text
getsockopt
```

The problem was ultimately caused by router changes not being applied.

This demonstrated an important troubleshooting principle:

> Verify that router configuration changes have actually been applied before troubleshooting the server itself.

For external connectivity problems, check in this order:

1. Server is powered on.
2. Server has the expected LAN IP.
3. Minecraft is running.
4. Minecraft is listening on the expected port.
5. Router port forwarding is configured correctly.
6. Router changes have been applied.
7. Test from outside the LAN.

Useful Linux command:

```bash
sudo ss -tulpn
```

This can show whether the server is listening for connections.

---

# 13. Dynamic Public IP

The server's local IP is static, but the public IP provided by the ISP may change.

This creates a problem if friends connect using the numeric public IP:

```text
< PUBLIC_IP >
```

If the ISP changes the public IP, the address shared with friends may stop working.

A Dynamic DNS service can solve this.

Conceptually:

```text
Minecraft Client
       |
       v
mc.example-ddns.com
       |
       v
Current Public IP
       |
       v
Home Router
       |
       v
<SERVER_IP>:25565
```

Potential DDNS providers include:

* DuckDNS
* No-IP
* Cloudflare with an appropriate DNS update mechanism

A DDNS configuration is a planned improvement.

---

# 14. Static Public IP vs. DDNS

A static LAN IP and static public IP solve different problems.

### Static LAN IP

Controls:

```text
Where the server is located inside the home network
```

Example:

```text
<SERVER_IP>
```

This is already configured.

### Static Public IP

Controls:

```text
Which address the Internet uses to reach the home network
```

This generally requires ISP support or a business/static-IP service.

### DDNS

Provides:

```text
A hostname that automatically follows a changing public IP
```

For a home Minecraft server, DDNS is generally more practical than paying for a static public IPv4 address.

---

# 15. Jellyfin Networking

Jellyfin runs inside Docker.

The server exposes Jellyfin's web interface through:

```text
8097/TCP
```

Local clients can connect using:

```text
http://<SERVER_IP>:8097
```

The architecture is:

```text
Client
  |
  v
LAN
  |
  v
<SERVER_IP>:8097
  |
  v
Docker
  |
  v
Jellyfin
```

Jellyfin does not currently need to be exposed to the public Internet for normal local use.

Keeping it LAN-only reduces unnecessary exposure.

---

# 16. BlueMap Networking

BlueMap provides a web interface for the Minecraft world.

The default web server port is:

```text
8100/TCP
```

Local access:

```text
http://<SERVER_IP>:8100
```

The architecture is:

```text
Browser
   |
   v
LAN
   |
   v
<SERVER_IP>:8100
   |
   v
BlueMap
```

---

# 17. NAS Networking

The server also provides network file sharing.

The basic architecture is:

```text
Windows PC
     |
     | SMB
     v
Home LAN
     |
     v
HP ProDesk
     |
     v
/mnt/storage
```

The NAS is intended for local network use.

Authentication is enabled so users do not have unrestricted guest access to the shared storage.

---

# 18. SSH Networking

SSH provides remote command-line access to Ubuntu.

The standard SSH port is:

```text
22/TCP
```

SSH is currently primarily intended for administration from the local network.

Example:

```bash
ssh <USERNAME>@<SERVER_IP>
```

SSH should not be exposed to the public Internet unnecessarily.

If remote SSH access is needed in the future, a VPN or another secure remote-access solution should be considered before simply forwarding port 22.

---

# 19. Firewall

Ubuntu's firewall can be checked with:

```bash
sudo ufw status
```

If UFW is enabled, only required services should be allowed.

Example:

```text
22/TCP      SSH
25565/TCP   Minecraft Java
19132/UDP   Minecraft Bedrock
8096/TCP    Jellyfin
8100/TCP    BlueMap
```

However, not all of these ports necessarily need to be exposed to the Internet.

A useful distinction is:

```text
LAN-only services
-----------------
SSH
NAS
Jellyfin
CasaOS

Internet-facing services
------------------------
Minecraft Java
Minecraft Bedrock
```

CasaOS in particular should generally remain inaccessible from the public Internet unless a secure remote-access method is deliberately configured.

---

# 20. Port Forwarding

Port forwarding allows unsolicited traffic from the Internet to reach a device on the LAN.

The general process is:

```text
Internet
   |
   v
Public IP
   |
   v
Router
   |
   | Port forwarding rule
   v
<SERVER_IP>
   |
   v
Service
```

Port forwarding should only be configured for services that actually need external access.

Current Minecraft forwarding:

```text
TCP 25565 -> <SERVER_IP>:25565
UDP 19132 -> <SERVER_IP>:19132
```

---

# 21. Testing Network Connectivity

### Check local IP

```bash
ip a
```

### Check default gateway

```bash
ip route
```

### Test the gateway

```bash
ping <GATEWAY_IP>
```

### Test Internet connectivity

```bash
ping 1.1.1.1
```

### Test DNS

```bash
ping google.com
```

### Check listening ports

```bash
sudo ss -tulpn
```

### Check firewall

```bash
sudo ufw status
```

### Check network interface status

```bash
ip link
```

---

# 22. Network Troubleshooting Methodology

When a service cannot be reached, troubleshoot from the inside out.

### Step 1 — Verify the service

Is the application running?

```bash
systemctl status <service>
```

or:

```bash
docker ps
```

### Step 2 — Verify the port

Is the service listening?

```bash
sudo ss -tulpn
```

### Step 3 — Test locally

Try connecting from the server itself.

### Step 4 — Test from another LAN device

This determines whether the problem is local to the service or related to the network.

### Step 5 — Check firewall

```bash
sudo ufw status
```

### Step 6 — Check router

Verify:

* Port forwarding
* DHCP reservation
* Applied configuration
* WAN connectivity

### Step 7 — Test externally

Use a device on a different network, such as cellular data.

This methodology helps isolate the problem rather than changing multiple things simultaneously.

---

# 23. Network Security Considerations

The server is an Internet-connected system and should be treated as an exposed network device.

Important principles:

* Do not expose unnecessary ports.
* Keep Ubuntu updated.
* Keep Paper and plugins updated.
* Use strong authentication.
* Do not use unrestricted guest access for sensitive storage.
* Keep backups of important data.
* Do not expose CasaOS unnecessarily.
* Do not expose SSH unnecessarily.
* Use a whitelist for the Minecraft server when appropriate.
* Never commit network credentials or secrets to Git.

---

# 24. Future Networking Improvements

Potential future improvements include:

* [x] Configure Dynamic DNS
* [ ] Create a custom domain/subdomain for Minecraft
* [ ] Add monitoring for network connectivity
* [ ] Add network-wide DNS/ad blocking
* [ ] Add a managed switch for additional networking practice
* [ ] Document the complete IP addressing scheme
* [ ] Monitor bandwidth and network utilization

---

# 25. Current Network Summary

```text
                    INTERNET
                        |
                        |
                 Public IP Address
                        |
                        v
                +---------------+
                | Home Router   |
                | DHCP / NAT    |
                +---------------+
                   |          |
              Wi-Fi          Ethernet
                |               |
                v               v
          Client Devices    HP ProDesk
                              |
                              v
                           Ubuntu
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
      Minecraft            Jellyfin             NAS
          |
      +---+---+
      |       |
     Java   Geyser
             |
          Bedrock

```

The current design uses the home router as the central networking device, a DHCP reservation to provide the server with a predictable LAN address, and selective port forwarding for Internet-facing services.

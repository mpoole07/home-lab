# Home Lab Infrastructure

A small home lab built from repurposed hardware. The primary server provides Minecraft hosting, network file storage, and media streaming through Jellyfin. The environment runs Ubuntu with CasaOS and Docker.

## Overview

### Primary Server

**Hardware:** HP ProDesk
**Purpose:** Home Server / NAS / Minecraft Server / Jellyfin Media Server

| Component      | Specification                 |
| -------------- | ----------------------------- |
| CPU            | Intel Core i5-4590 @ 3.30 GHz |
| CPU cores      | 4 cores / 4 threads           |
| RAM            | 16 GB DDR3                    |
| System storage | 128 GB SSD                    |
| Data storage   | 1 TB HDD                      |
| Network        | Ethernet                      |
| OS             | Ubuntu                        |
| Management     | SSH + CasaOS                  |
| Containers     | Docker                        |

### Services

Currently hosted services:

* Minecraft Java Edition server
* Minecraft Bedrock crossplay
* Network file sharing / NAS
* Jellyfin media server
* CasaOS management dashboard

---

# 1. Operating System

Ubuntu was installed on the 128 GB SSD.

The 1 TB HDD is used primarily for data storage.

The general storage layout is:

```text
SSD
└── Ubuntu / system files

HDD
└── /mnt/storage
    ├── Media
    │   ├── Movies
    │   ├── Shows
    │   └── Music
    └── Minecraft
```

The separation between the system drive and data drive allows the operating system to be reinstalled without unnecessarily affecting the media and Minecraft data.

---

# 2. Storage

The 1 TB HDD was formatted as EXT4 and mounted at:

```text
/mnt/storage
```

The storage drive is intended for:

* Media
* Minecraft server files
* Network file storage
* Other future server data

The server's SSD is reserved primarily for Ubuntu and system files.

## Storage Troubleshooting

During setup, there were several mount-related issues involving:

```text
/run/media/<username>/storage
```

and:

```text
/mnt/storage
```

The final configuration uses:

```text
/mnt/storage
```

as the persistent storage location.

When troubleshooting mounts, useful commands include:

```bash
lsblk -f
```

```bash
df -h
```

```bash
mount
```

```bash
sudo mount -a
```

After modifying `/etc/fstab`, systemd may need to be reloaded:

```bash
sudo systemctl daemon-reload
```

---

# 3. Network File Sharing / NAS

A network share was created on the server so other computers on the local network can access files.

The share requires authentication rather than unrestricted guest access.

This allows Windows computers to access the server's storage through the network.

General workflow:

```text
Windows PC
    |
    | SMB
    v
Ubuntu Server
    |
    v
/mnt/storage
```

The NAS is primarily intended for:

* Moving files between computers
* Storing media
* Accessing server files from Windows
* Managing the media library

---

# 4. SSH

SSH was enabled on the Ubuntu server for remote administration.

This allows the server to be managed without connecting a monitor and keyboard.

Typical connection:

```bash
ssh <username>@<SERVER_IP>
```

SSH is used for:

* Installing software
* Managing services
* Editing configuration files
* Updating Minecraft
* Troubleshooting
* General Linux administration

---

# 5. CasaOS

CasaOS was installed to provide a web-based management interface for the server.

CasaOS makes it easier to:

* Manage Docker applications
* View storage
* Monitor the server
* Install applications
* Manage containers

Docker's data directory was moved from the default location to the server's storage drive.

The Docker data location was configured so application data could use the larger 1 TB HDD.

After changing Docker's storage configuration, Docker needed to be restarted before the change became active.

---

# 6. Minecraft Server

The server hosts a Minecraft Java Edition server using Paper.

### Server software

* Paper
* Geyser
* Floodgate
* ViaVersion
* ViaBackwards
* BlueMap

The Minecraft server runs on the Ubuntu server and is accessible from both the local network and the Internet.

## Java Server

The Minecraft server uses Paper rather than the standard vanilla server.

Paper provides:

* Better server performance
* Plugin support
* Additional configuration options

The server is started using a shell script rather than automatically starting at boot.

Example:

```bash
./start.sh
```

The server can be shut down cleanly using:

```text
stop
```

---

# 7. Minecraft Networking

The server was assigned a static local IP address through the router.

This is important because port forwarding needs to consistently point to the same machine.

Network architecture:

```text
Internet
   |
   v
Router
   |
   | Port forwarding
   v
HP ProDesk
   |
   v
Paper Minecraft Server
```

The standard Java Minecraft port is:

```text
25565 TCP
```

The router forwards this port to the server's local IP.

The server was tested successfully from outside the local network.

---

# 8. Bedrock Crossplay

Bedrock compatibility was added using:

* Geyser
* Floodgate

Geyser translates Bedrock network traffic so Bedrock players can connect to the Java server.

Floodgate allows Bedrock players to authenticate without requiring a Java Edition account.

Bedrock traffic uses:

```text
19132 UDP
```

The router forwards UDP port 19132 to the server.

The result is:

```text
Java Edition ────────┐
                     │
                     v
                 Paper Server
                     ^
                     │
Bedrock ──> Geyser ──┘
```

## Bedrock Chat

Bedrock players initially had problems sending chat messages.

The issue was resolved by changing:

```text
enforce-secure-profile=true
```

to:

```text
enforce-secure-profile=false
```

in `server.properties`.

---

# 9. Multiple Minecraft Versions

ViaVersion and ViaBackwards were added to allow clients using different Java versions to connect to the server.

Current compatibility stack:

```text
Java clients
     |
     v
ViaVersion / ViaBackwards
     |
     v
Paper
     ^
     |
Geyser / Floodgate
     ^
     |
Bedrock clients
```

This allows the server to accommodate different Java client versions without requiring every player to use exactly the same version.

---

# 10. BlueMap

BlueMap was installed as a Paper plugin to provide an interactive web-based 3D map of the Minecraft world.

The BlueMap web interface normally runs on:

```text
8100
```

Local access:

```text
http://<SERVER_IP>:8100
```

BlueMap provides:

* 3D world visualization
* Zooming and panning
* World exploration
* Player/world information

It provides a useful way to explore the Minecraft world without opening Minecraft.

---

# 11. Minecraft Backups

The Minecraft world should be backed up regularly.

Important directories include:

```text
world/
world_nether/
world_the_end/
plugins/
```

A backup should be performed before major changes such as:

* Paper updates
* Plugin updates
* Minecraft version changes
* Major configuration changes

Created new backup folder in:

/mnt/storage/

A backup of the Minecraft world is created every day at 3:00 AM and backup older than 7 days old are automatically deleted.

---

# 12. Jellyfin

Jellyfin was installed through CasaOS/Docker to provide a self-hosted media server.

The media library is stored on the 1 TB HDD rather than inside the Docker container.

Host:

```text
/mnt/storage/Media
```

Container:

```text
/media
```

This distinction is important because Docker containers see the mounted directory using the container path.

For example:

```text
Host:
 /mnt/storage/Media/Movies

Jellyfin container:
 /media/Movies
```

## Media Organization

Movies use:

```text
Movies/
└── Movie Name (Year)/
    └── Movie Name (Year).mp4
```

Example:

```text
Movies/
└── Interstellar (2014)/
    └── Interstellar (2014).mp4
```

TV shows use:

```text
Shows/
└── Show Name/
    └── Season 01/
        ├── Show Name - S01E01.mp4
        ├── Show Name - S01E02.mp4
        └── ...
```

Correct naming allows Jellyfin to automatically identify media and retrieve metadata.

---

# 13. DVD Media

Physical DVDs can be converted into MKV files using MakeMKV and then into MP4 files using HandBrake.

Typical workflow:

```text
DVD
 |
 v
MakeMKV
 |
 v
MKV file
 |
 v
HandBrake
 |
 v
MP4 file
 |
 v
Media folder
 |
 v
Jellyfin
```

MakeMKV was used with an external USB DVD drive.

DVDs may contain AC3/Dolby Digital audio.

---

# 14. Jellyfin Troubleshooting

After reinstalling the Jellyfin container, playback failed with:

```text
Playback failed because the media is not supported by this client
```

FFmpeg logs showed:

```text
Error opening input: No such file or directory
```

The important path was:

```text
file:/Media/Shows/...
```

while the Docker mount was intended to use:

```text
/media
```

Linux paths are case-sensitive.

Therefore:

```text
/Media
```

and:

```text
/media
```

are different paths.

When troubleshooting Jellyfin Docker deployments, verify both:

1. The host path
2. The container path

The media files themselves should remain outside the container so reinstalling Jellyfin does not delete the library.

---

# 15. Hardware and Other Systems

Several older computers were evaluated as potential servers or gaming systems.

The current primary server was selected because the HP ProDesk provided a good balance of:

* Low power consumption
* 4-core Intel CPU
* 16 GB RAM
* SSD system storage
* 1 TB HDD
* Ethernet
* Small physical footprint

The server does not require a dedicated GPU for its current workloads.

---

# 16. Current Architecture

The current home lab can be summarized as:

```text
                         INTERNET
                            |
                            v
                       Home Router
                       /          \
                      /            \
             Minecraft ports      LAN
                    |               |
                    v               v
             HP ProDesk Server  Local Devices
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Paper     Jellyfin    NAS
          |
     +----+----+
     |         |
   Java     Geyser
              |
          Bedrock

```

---

# 17. Current Server Goals

The server is intended to eventually provide:

* [x] Ubuntu server
* [x] SSH administration
* [x] 16 GB RAM upgrade
* [x] 1 TB storage upgrade
* [x] Network file sharing
* [x] CasaOS
* [x] Docker
* [x] Minecraft Java server
* [x] Internet-accessible Minecraft server
* [x] Bedrock crossplay
* [x] Java version compatibility
* [x] Jellyfin
* [x] DVD media workflow
* [x] BlueMap

Future improvements:

* [ ] Better remote access
* [ ] Monitoring
* [ ] UPS
* [ ] Automated updates
* [ ] Improved Jellyfin transcoding
* [ ] Additional Docker services

---

# 18. Lessons Learned

Some of the most useful lessons from building this server:

### Docker containers have their own filesystem view

A host path such as:

```text
/mnt/storage/Media
```

can appear inside a container as:

```text
/media
```

### Static local IPs are important for servers

Port forwarding and other services depend on the server keeping the same LAN address.

### Backups should happen before changes

Always back up important data before:

* OS changes
* Docker changes
* Minecraft updates
* plugin updates
* storage changes

### Logs are extremely useful

Rather than guessing, server logs can often identify the exact cause of a problem.

For example, the Jellyfin FFmpeg error immediately showed that the application was trying to access a nonexistent path.

---

# 19. Security Notes

This server is exposed to the Internet, so security is important.

Current/future security measures include:

* SSH authentication
* Minecraft whitelist
* Limited port forwarding
* Strong passwords
* Regular updates
* Minecraft backups
* Avoiding unnecessary exposed services

---

# 20. Repository Purpose

This repository documents the construction and maintenance of the home lab.

The goal is not only to record the final configuration, but also to document:

* Why components were selected
* How services were configured
* Problems encountered
* Troubleshooting procedures
* Important commands
* Future improvements

This makes the repository useful as both a personal reference and a demonstration of practical IT, networking, Linux, virtualization/container, and server administration skills.

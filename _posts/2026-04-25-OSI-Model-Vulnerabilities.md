---
layout: post
title: "The OSI Model and Vulnerabilities Associated to each Layer"
date: 2026-04-25
categories: [Cybersecurity, Networking]
---

The OSI Model: A Comprehensive Deep Dive & Vulnerability Map
The Open Systems Interconnection (OSI) model is a broad overview of how data moves through a network. It provides a universal language for professionals across different domains and experience levels. Whether you are a helpdesk manager in Europe or a Tier 3 SOC analyst in the USA, "Layer 4" remains a constant.

Each layer utilizes multiple protocols to facilitate data movement. Understanding how these layers function—and how they break—is the foundation of cybersecurity.

1. Physical Layer (Layer 1)
The bottom layer represents the physical components of the network. Here, we deal with signals, cables, and connectors. If a network administrator mentions a "Layer 1 problem," it likely involves hardware—checking connectors, punch downs, or perhaps a mouse literally eating an Ethernet cable.

Cybersecurity Takeaway:
Physical Tapping: Intercepting data by directly accessing copper or fiber optic cables to "copy" the signal.

Signal Jamming: Flooding transmission mediums (like Wi-Fi frequencies) with noise to cause a physical Denial of Service (DoS).

Hardware Implants: Inserting malicious devices, such as network taps or keyloggers, between legitimate hardware.

USB HID Attacks: Using devices like the "Rubber Ducky" that mimic keyboards to inject commands upon connection.

Cable Sabotage: Physically cutting network lines to sever communication.

Side-Channel Attacks: Monitoring electromagnetic interference (EMI) or power consumption to extract sensitive data.

Rogue Access Points: Plugging unauthorized routers into active jacks to bypass perimeter security.

Environmental Manipulation: Tampering with power or cooling systems to cause hardware failure.

Port Stealing: Taking over an unattended, physically active network port.

2. Data Link Layer (Layer 2)
Known as the Switching Layer , Layer 2 handles data transfer strictly within a Local Area Network (LAN). It does not move data across different networks. This layer relies on MAC (Media Access Control) addresses —unique hexadecimal identifiers (eg, 00:1A:2B:3C:4D:5E) assigned to every network module (Wi-Fi, Bluetooth, Ethernet).

Switching Logic:
Switches make decisions based on three primary protocols:

Forward: If the switch knows the destination MAC and its port, it sends the data directly there.

Flood: If the destination is unknown, the switch sends the data to every port except the source, hoping the device responds correctly.

Filter: If the source and destination are on the same port, the switch drops the frame to save power.

MAC Address Breakdown:
OUI (Organizationally Unique Identifier): The first 3 bytes. This identifies the manufacturer (eg, Intel, Apple).

Network Interface Identifier: The last 3 bytes. A unique serial number for that specific chip.

Cybersecurity Breach Vulnerability:
MAC Flood Attack: Attackers send thousands of fake MAC addresses to fill the switch's "MAC Table" (its brain). Once full, the switch enters a "fail-open" state, flooding all frames to all ports. At this point, it acts like a "Hub," allowing an attacker using Wireshark to capture everyone's private data.

3. Network Layer (Layer 3)
Layer 3 is responsible for retrieving data across networks. Routers operate here, ensuring data is packed into packets and moved from the correct source IP to the correct destination IP. Data moves from router to router in "hops"—essentially checkpoints for data.

The Three Flavors of IP Communication:
Unicast (One-to-One): Direct communication between two specific IPs.

Broadcast (One-to-All): Shouting to every device on a local network using a broadcast address (eg, .255).

Multicast (One-to-Many): Sending data to a specific group (common in streaming or routing updates).

Cybersecurity Vulnerabilities:
IP Spoofing: Faking a source IP to impersonate another system.

ICMP Flood (Ping Flood): Overwhelming a target with "Echo Request" packets to crash the system.

Smurf Attack: Sending a spoofed ping to a broadcast address so every device replies to the victim, amplifying the attack.

IP Fragmentation (Teardrop): Sending malformed, overlapping packets that crash the OS during reassembly.

Ping of Death: Sending an oversized ICMP packet (over65 ,535bytes) to cause buffer overflows.

BGP/Route Hijacking: Falsifying routing updates to redirect internet traffic.

Rogue DHCP Server: Providing incorrect network settings to intercept traffic.

IP Address Scanning: Using tools like Nmap to identify "alive" hosts for further attack planning.

4. Transport Layer (Layer 4)
The "Logistics Department." While Layer 3 gets data to the right house (IP), Layer 4 ensures it reaches the right Port Number .

Port 80/443: Web Traffic.

Port 25: Email.

Port 53: DNS.

Primary Protocols:
TCP (Transmission Control Protocol): The "Responsible" protocol. It uses a Three-Way Handshake (SYN -> SYN-ACK -> ACK) to ensure data is properly received. The application doesn't need to check for errors.

UDP (User Datagram Protocol): The "Fast" protocol. It doesn't care about acknowledgments; The application level must manage reliability.

Segmentation: Breaking large data into smaller, manageable packages.

Cybersecurity Concerns:
Port Scanning: Identifying open services to find vulnerabilities.

SYN Flood (DoS): Exhausting server resources by leaving TCP handshakes unfinished.

TCP Session Hijacking: Stealing "Sequence Numbers" to take over an authenticated connection.

UDP Flooding: Overwhelming random ports to exhaust system resources.

Port Knocking Bypass: Discovering hidden ports protected by a specific connection sequence.

Blind Reset Attack: Using spoofed "RST" packets to kill legitimate connections.

SSL/TLS Interception: Targeting encryption occurring just above this layer.

5. Session Layer (Layer 5)
Responsible for initiating, maintaining, and gracefully ending sessions. It ensures that multiple applications (Zoom, a movie download, and YouTube) can run on the same IP without interrupting each other. It also provides Checkpoints ; if a download fails at 70%, it resumes from that point rather than restarting at 0%.

Key Protocols:
RPC (Remote Procedure Call): Executing code on a remote computer as if it were local.

NetBIOS: Used for resource sharing (printers/files) on older Windows networks.

SOCKS: A proxy protocol used for tunneling traffic.

PPTP: A legacy VPN tunneling protocol.

SQL: Maintains the connection between a website's front-end and the database back-end.

Cybersecurity Concerns:
Session Hijacking (Sidejacking): Stealing a Session ID (cookie) to impersonate a user.

Session Fixation: Tricking a user into logging into a session ID known to the attacker.

Session Exhaustion: A DoS attack that fills up all available server session "slots."

Brute-Forcing Session IDs: Guessing weak or predictable IDs (eg, 1001, 1002).

Insecure Session Termination: "Zombie" sessions that remain active after a user logs out.

MitM via RPC/NetBIOS: Intercepting handshakes to inject unauthorized commands.

6. Presentation Layer (Layer 6)
The "Translator and Formatter." It ensures data from Layer 7 of one system is readable by Layer 7 of another, handling syntax and semantics.

Primary Jobs:
Translation: Converting different data representations (ASCII vs. EBCDIC) into a common format.

Encryption: The "Security Guard" layer. It handles SSL/TLS to keep data private.

Compression: Shrinking data to save bandwidth (vital for high-def streaming).

Common Formats:
Images: JPEG, PNG, GIF.

Video: MPEG, QuickTime.

Data: XML, JSON, ASCII, HTML.

Cybersecurity Concerns:
Encryption Flaws: Using cracked or outdated encryption (like SSL v2).

Malicious Code in Files: Hiding malware in JPEGs or XML files.

Cleartext Exposure: Failure to encrypt sensitive data in transit.

Format String Attacks: Exploiting how applications read data formats to execute code.

7. Application Layer (Layer 7)
The "Dashboard." This is the only layer users interact with. It provides the interface and protocols applications use to communicate with the network.

Responsibilities & Protocols:
Resource Sharing: Accessing remote files/databases.

Service Identification: Determining user intent (Web vs. File Upload).

Authentication: Managing logins and passwords.

Big Players: HTTP/HTTPS (Web), SMTP/IMAP (Email), FTP/SFTP (Files), DNS (Phonebook), SSH (Secure Management).

Cybersecurity Concerns:
SQL Injection: Stealing database info via malicious input.

XSS: Injecting scripts into websites to steal user cookies.

Phishing: Using fake interfaces to steal credentials.

L7 DDoS: Overwhelming specific features (like a search bar) to crash a server.

API Attacks: Exploiting app-to-app communication to leak data.

Final Summary & Data Flow
Layer	Name	Data Unit	Core Function
7	Application	Data	User Interface (The Eyes)
6	Presentation	Data	Translation & Encryption (SSL/TLS)
5	Session	Data	Dialogue Management (Tunneling)
4	Transport	Segments	End-to-End Delivery (TCP/UDP)
3	Network	Packets	Routing & IP Addressing
2	Data Link	Frames	Switching & MAC Addressing
1	Physical	Bits	Hardware, Cables, & Signals
The Data Flow Example:
Application: You visit https://mail.google.com.

Presentation: Data is encrypted via SSL.

Session: A link is established between the browser and server.

Transport: Data is encapsulated into TCP segments.

Network: Segments are encapsulated into IP packets.

Data Link: Packets are encapsulated into Ethernet frames.

Physical: Frames are converted into electrical signals.

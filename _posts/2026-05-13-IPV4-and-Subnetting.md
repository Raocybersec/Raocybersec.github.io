# Technical Deep-Dive: Hierarchical Network Design & VLSM
**Category:** Network Engineering / Defensive Security  
**Series:** Proof of Work Labs  
**Author:** Rao Tariq Jameel  

---

## 1. Introduction
In network defense, visibility is everything. You cannot defend what you don't understand. During my recent labs, I moved past simple IP addressing to master **Variable Length Subnet Masking (VLSM)**. This post breaks down how to engineer a network that scales for 500+ hosts while maintaining strict security boundaries.

## 2. The Strategy: Supernetting vs. Subnetting
Most beginners think of a subnet as a single octet (e.g., `192.168.10.x`). However, enterprise security requires a more fluid approach:

*   **Supernetting (The "Left" Shift):** I started with **192.168.10.0/23**. By moving the CIDR to the left, I "swallowed" two `/24` networks (`.10.x` and `.11.x`) to create a single pool of **512 IPs**.
*   **Subnetting (The "Right" Shift):** I then "sliced" that pool into smaller, logical containers (subnets) based on department size.



## 3. Engineering the 5-Department Architecture
The goal was to accommodate 500+ hosts across 5 departments without wasting address space. By utilizing **VLSM**, I created a "Subnet within a Subnet" hierarchy:

### The 512 IP Allocation Table
| Segment | CIDR | Address Range | Total IPs | Department Role |
| :--- | :--- | :--- | :--- | :--- |
| **Subnet A** | `/25` | `10.0 - 10.127` | 128 | Sales (High Density) |
| **Subnet B** | `/25` | `10.128 - 10.255` | 128 | Marketing |
| **Subnet C** | `/25` | `11.0 - 11.127` | 128 | Engineering/Dev |
| **Subnet D** | `/26` | `11.128 - 11.191` | 64 | HR (Lower Density) |
| **Subnet E** | `/26` | `11.192 - 11.255` | 64 | Finance |



## 4. Security Implementation & Lessons Learned
Designing this architecture taught me three critical lessons for my journey into **Cloud Security Engineering**:

### A. The Spanning-Octet Logic
A network boundary is defined by the **Subnet Mask**, not the dots in the IP. In a `/23`, a machine on `.10.255` and `.11.0` are neighbors on the same super-network, even though they look different to the untrained eye.

### B. The Usability Tax
For every slice I created, I "paid" 2 IPs: the **Network ID** and the **Broadcast ID**. In this 5-department design, I reserved **10 IPs** for network overhead, leaving me with precisely **502 usable host addresses**.

### C. Micro-Segmentation & Defense-in-Depth
By carving these subnets, I can now implement **Access Control Lists (ACLs)**. For example, I can configure a firewall to:
1. Allow the *Engineering* subnet to reach the *Production Servers*.
2. Strictly **DROP** any traffic from the *Marketing* subnet trying to reach *Finance*.

## 5. Conclusion: Logic over Memorization
Subnetting isn't about memorizing tables; it's about binary math ($2^n$). 
*   **Need more hosts?** Give bits back to the host side (Decrease CIDR).
*   **Need more security zones?** Borrow bits from the hosts (Increase CIDR).

This mastery allows me to design environments that aren't just functional, but are **defensible by design**.

---
*This post is part of my "Proof of Work" portfolio, documenting hands-on labs in Networking and Cybersecurity.*

# Chapter 2: Fundamentals of Ethernet LANs - CCNA 200-301 Volume 1

## Overview

Chapter 2 covers the fundamental concepts of Ethernet Local Area Networks (LANs), focusing on how modern Ethernet networks operate at the data link layer (Layer 2) and physical layer (Layer 1) of the OSI model. This chapter establishes the foundation for understanding switched Ethernet networks.

## Key Topics

- [x] Ethernet LAN Switching Concepts
- [x] The Ethernet Data-Link Protocol  
- [x] Ethernet Physical Layer Standards
- [x] Building Physical Ethernet LANs with UTP
- [x] Ethernet Frame Structure and Processing
- [x] Half Duplex vs Full Duplex Operation
- [x] Collision Detection and CSMA/CD
- [x] Ethernet Standards Evolution (10BASE-T to 10GBASE-T)

## Important Concepts

### Ethernet LAN Switching

**Key Points:**
- Modern Ethernet LANs use switches instead of hubs
- Switches create separate collision domains for each port
- Each switch port operates in full-duplex mode by default
- Switches maintain MAC address tables to forward frames intelligently
- Eliminates collisions in modern switched networks

**Switch Operation:**
1. **Learning**: Switch learns MAC addresses from source addresses of incoming frames
2. **Flooding**: When destination MAC is unknown, switch floods frame to all ports except the incoming port
3. **Forwarding**: When destination MAC is known, switch forwards frame only to the appropriate port
4. **Filtering**: Switch does not forward frames back to the source port

### Ethernet Frame Structure

**Standard Ethernet II Frame Format:**
```
| Preamble | SFD | Destination MAC | Source MAC | Type/Length | Data | FCS |
|    7     | 1   |        6        |     6      |      2      |46-1500| 4   |
```

**Field Descriptions:**
- **Preamble (7 bytes)**: Alternating 1s and 0s for synchronization
- **SFD (1 byte)**: Start Frame Delimiter (10101011)
- **Destination MAC (6 bytes)**: Target device's hardware address
- **Source MAC (6 bytes)**: Sending device's hardware address  
- **Type/Length (2 bytes)**: Indicates protocol type (IPv4=0x0800) or frame length
- **Data (46-1500 bytes)**: Actual payload data
- **FCS (4 bytes)**: Frame Check Sequence for error detection

### MAC Addresses

**Characteristics:**
- 48-bit (6-byte) hardware addresses
- Expressed in hexadecimal notation (e.g., 00:1A:2B:3C:4D:5E)
- First 24 bits = OUI (Organizationally Unique Identifier)
- Last 24 bits = Device-specific identifier
- Burned into network interface cards (NICs)

**Special MAC Addresses:**
- **Broadcast**: FF:FF:FF:FF:FF:FF (all devices receive)
- **Multicast**: First bit of first byte = 1
- **Unicast**: First bit of first byte = 0

### Physical Layer Standards

**Common Ethernet Standards:**

| Standard | Speed | Cable Type | Max Distance | Duplex |
|----------|-------|------------|--------------|--------|
| 10BASE-T | 10 Mbps | Cat 3 UTP | 100m | Half/Full |
| 100BASE-TX | 100 Mbps | Cat 5 UTP | 100m | Half/Full |
| 1000BASE-T | 1 Gbps | Cat 5e UTP | 100m | Full only |
| 10GBASE-T | 10 Gbps | Cat 6a UTP | 100m | Full only |

**UTP Cable Categories:**
- **Cat 3**: Up to 10 Mbps
- **Cat 5**: Up to 100 Mbps  
- **Cat 5e**: Up to 1 Gbps
- **Cat 6**: Up to 1 Gbps (better performance than 5e)
- **Cat 6a**: Up to 10 Gbps

### CSMA/CD (Carrier Sense Multiple Access with Collision Detection)

**Legacy Half-Duplex Operation:**
- Used in hub-based networks (now obsolete)
- Devices listen before transmitting (Carrier Sense)
- Multiple devices can access the medium (Multiple Access)
- Collisions detected and handled (Collision Detection)

**Collision Detection Process:**
1. Listen to the wire before transmitting
2. If medium is busy, wait for a random time
3. Transmit frame while continuing to listen
4. If collision detected, send jam signal
5. Use exponential backoff algorithm to retry

**Modern Full-Duplex Operation:**
- No collisions possible (separate transmit/receive paths)
- CSMA/CD disabled
- Higher efficiency and throughput
- Standard in all modern switched networks

### Switch MAC Address Table

**Table Contents:**
- MAC Address
- Interface/Port
- VLAN ID
- Timestamp (for aging)

**Aging Process:**
- Default aging time: 300 seconds (5 minutes)
- Entries removed if no frames received from that MAC
- Prevents table overflow with stale entries

## Lab Exercises

- [x] **Lab 2-1**: Analyzing Ethernet Frame Headers with Wireshark
  - Capture and examine Ethernet frames
  - Identify frame fields and their values
  - Observe MAC address learning process

- [x] **Lab 2-2**: Observing Switch MAC Address Table
  - Connect devices to switch
  - View and analyze MAC address table entries
  - Test aging timer functionality

- [x] **Lab 2-3**: Cable Testing and Pinout Verification
  - Test UTP cables with cable tester
  - Verify proper pinout configurations
  - Identify common cable faults

- [x] **Lab 2-4**: Switch Port Configuration
  - Configure switch port speed and duplex
  - Test auto-negotiation behavior
  - Troubleshoot duplex mismatches

## Review Questions

1. **What are the main differences between a hub and a switch in terms of collision domains?**
   - Hub: Single collision domain for all ports
   - Switch: Separate collision domain per port

2. **Describe the three main functions of an Ethernet switch.**
   - Learning: Build MAC address table from source addresses
   - Flooding: Forward unknown unicast frames to all ports
   - Forwarding: Send frames to specific ports based on destination MAC

3. **What is the purpose of the Frame Check Sequence (FCS) field?**
   - Error detection using CRC-32 algorithm
   - Allows receiving device to detect corrupted frames

4. **Why is CSMA/CD not needed in modern switched Ethernet networks?**
   - Full-duplex operation eliminates collisions
   - Separate transmit and receive paths
   - Each port is its own collision domain

5. **What happens when a switch receives a frame with an unknown destination MAC address?**
   - Frame is flooded to all ports except the source port
   - Switch continues learning from source MAC addresses

6. **List three factors that determine the physical Ethernet standard to use.**
   - Required speed/bandwidth
   - Cable type and category available
   - Maximum distance requirements

7. **How does the switch aging timer help manage the MAC address table?**
   - Removes stale entries after timeout period (default 300 seconds)
   - Prevents table overflow
   - Allows for device mobility

8. **What are the advantages of full-duplex over half-duplex Ethernet?**
   - No collisions
   - Doubled effective bandwidth
   - No need for CSMA/CD
   - Better performance and efficiency

## Command Reference

**Cisco Switch Commands:**
```
# View MAC address table
show mac address-table

# View specific interface MAC addresses
show mac address-table interface fastethernet 0/1

# Clear MAC address table
clear mac address-table

# Configure port speed and duplex
interface fastethernet 0/1
speed 100
duplex full

# Enable auto-negotiation
speed auto
duplex auto
```

## Additional Resources

- [IEEE 802.3 Ethernet Standards](https://standards.ieee.org/standard/802_3-2018.html)
- [Cisco Ethernet Technologies White Paper](https://www.cisco.com/c/en/us/products/switches/ethernet-switching.html)
- [Network Cable Pinout Reference](https://www.cisco.com/c/en/us/support/docs/lan-switching/ethernet/10561-83.html)
- [Wireshark Ethernet Analysis Tutorial](https://www.wireshark.org/docs/)
- [RFC 894 - Transmission of IP Datagrams over Ethernet Networks](https://tools.ietf.org/html/rfc894)
- [Cable Categories and Performance Standards](https://www.TIA-942.org)

## Study Tips

1. **Understand the frame format**: Memorize the Ethernet frame structure and field sizes
2. **Practice MAC address analysis**: Learn to read and interpret MAC addresses
3. **Know the standards**: Understand speed/distance relationships for different Ethernet standards
4. **Hands-on practice**: Use packet capture tools to analyze real network traffic
5. **Switch operations**: Understand how switches learn, flood, and forward frames

## Key Formulas and Values

- **Minimum frame size**: 64 bytes (including headers)
- **Maximum frame size**: 1518 bytes (standard) or 1522 bytes (with 802.1Q tag)
- **MAC address aging timer**: 300 seconds (default)
- **UTP cable max distance**: 100 meters
- **Full-duplex advantage**: 2x bandwidth utilization

---
*Last updated: August 18, 2025*
*Based on CCNA 200-301 Official Cert Guide Volume 1*

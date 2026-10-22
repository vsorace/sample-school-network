# Sample School Network

A simplified representation of a school network, demonstrating my ability to configure access points, VLANs, subnets, DHCP, trunk ports, and sub-interfaces. This document will summarize the network configuration and go into detail about how I set it up.

<img width="666" height="452" alt="Sample School Network" src="https://github.com/user-attachments/assets/47ecf45b-bc81-4990-9043-04b655c19242" />

For future revisions, I plan on implementing the following:
- Better Wi-Fi security
- A firewall
- A way to simulate internet access

## Hardware used

- 2911 Router (Router0)
- 2960-24TT Switch (Switch0)
- Three PT-AC Access Points
- Six Laptops, two per access point

## Network Summary

This network is divided into three VLANs
<table>
  <tr>
    <th></th>
    <th>VLAN 10</th>
    <th>VLAN 20</th>
    <th>VLAN 30</th>
  </tr>
  <tr>
    <th>Name</th>
    <td>Guest</td>
    <td>Staff</td>
    <td>Students</td>
  </tr>
  <tr>
    <th>IP Address</th>
    <td>10.0.0.1</td>
    <td>10.0.64.1</td>
    <td>10.0.128.1</td>
  </tr>
   <tr>
    <th>Subnet Mask</th>
    <td>255.255.192.0</td>
    <td>255.255.192.0</td>
    <td>255.255.192.0</td>
  </tr>
   <tr>
    <th>Interface on Switch0</th>
     <td>fa0/1</td>
     <td>fa0/2</td>
     <td>fa0/3</td>
  </tr>
</table>

In this scenario, the IP range of 10.0.192.1-10.0.255.255 is unused

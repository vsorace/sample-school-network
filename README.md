# Sample School Network

A simplified representation of a school network, demonstrating my ability to configure access points, VLANs, subnets, DHCP, trunk ports, and sub-interfaces. This document will summarize the network configuration and go into detail about how I set it up.

<img width="654" height="440" alt="sample school network-rev1" src="https://github.com/user-attachments/assets/b6b3ad76-ea0f-49d9-ab65-1f6929ea5bc5" />

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

This network is divided into three VLANs, as follows:

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
    <td>10.0.0.2</td>
    <td>10.0.64.2</td>
    <td>10.0.128.2</td>
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

Additionally, Router0 is configured as a "router-on-a-stick" configuration using dot1Q encapsulation, with interface g0/0 being divided into three separate sub-interfaces. They are configured as follows:

<table>
  <tr>
    <th>Interface on Router0</th>
     <td>g0/0.10</td>
     <td>g0/0.20</td>
     <td>g0/0.30</td>
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
</table>

# How I did the thing

I started out with my Router0, ensuring that interface `g0/0` is enabled using the following commands:
<br>`en`</br><br>`conf t`</br><br>`int g0/0 no shutdown`</br><br>`exit`</br>

While still in `config` mode, I configured the sub-interfaces on interface `g0/0`:
<br>`int g0/0.10`</br><br>`encap dot1Q 10`</br><br>`ip addr 10.0.0.1 255.255.192.0`</br><br>`no shutdown`</br><br>`exit`</br>

I followed the same process for interfaces `g0/0.20` and `g0/0.30`:
<br>`int g0/0.20`</br><br>`encap dot1Q 20`</br><br>`ip addr 10.0.64.1 255.255.192.0`</br><br>`no shutdown`</br><br>`exit`</br>

<br>`int g0/0.30`</br><br>`encap dot1Q 30`</br><br>`ip addr 10.0.128.1 255.255.192.0`</br><br>`no shutdown`</br><br>`exit`</br>

I then assigned DHCP pools on Router0 for the VLANs that were going to later be configured on Switch0.
<br>`ip dhcp pool GUEST`</br><br>`network 10.0.0.0 255.255.192.0`</br><br>`default-router 10.0.0.1`</br><br>`exit`</br>

<br>`ip dhcp pool STAFF`</br><br>`network 10.0.64.0 255.255.192.0`</br><br>`default-router 10.0.64.1`</br><br>`exit`</br>

<br>`ip dhcp pool STUDENTS`</br><br>`network 10.0.128.0 255.255.192.0`</br><br>`default-router 10.0.128.1`</br><br>`exit`</br>

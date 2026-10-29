# Sample School Network

A simplified representation of a school network, demonstrating my ability to configure access points, VLANs, subnets, DHCP, trunk ports, and sub-interfaces. This document will summarize the network configuration and go into detail about how I set it up.

<img width="607" height="430" alt="updated topology" src="https://github.com/user-attachments/assets/69f80ccc-e53c-4069-a4c3-545e33d23cbb" />

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

### This network is divided into three VLANs, as follows:

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

In this scenario, the IP range of 10.0.192.1-10.0.255.255 is unused.

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

The ultimate goal of this configuration is for the Guest, Staff, and Student networks to be isolated from each other, while still allowing hosts and devices within each network to communicate with each other.

# How I did the thing

## Network segmentation

### Configuring sub-interfaces

I started out with my Router0, ensuring that interface `g0/0` is enabled using the following commands:
<br>`en`<br>`conf t`<br>`int g0/0 no shutdown`<br>`exit`

While still in `config` mode, I configured the sub-interfaces on interface `g0/0`:
<br>`int g0/0.10`<br>`encap dot1Q 10`<br>`ip addr 10.0.0.1 255.255.192.0`<br>`no shutdown`<br>`exit`

I followed the same process for interfaces `g0/0.20` and `g0/0.30`:
<br>`int g0/0.20`<br>`encap dot1Q 20`<br>`ip addr 10.0.64.1 255.255.192.0`<br>`no shutdown`<br>`exit`

<br>`int g0/0.30`<br>`encap dot1Q 30`<br>`ip addr 10.0.128.1 255.255.192.0`<br>`no shutdown`<br>`exit`

### Configuring DHCP pools and exclusions

I then assigned DHCP pools on Router0 for the VLANs that were going to later be configured on Switch0.
<br>`ip dhcp pool GUEST`<br>`network 10.0.0.0 255.255.192.0`<br>`default-router 10.0.0.1`<br>`exit`

<br>`ip dhcp pool STAFF`<br>`network 10.0.64.0 255.255.192.0`<br>`default-router 10.0.64.1`<br>`exit`

<br>`ip dhcp pool STUDENTS`<br>`network 10.0.128.0 255.255.192.0`<br>`default-router 10.0.128.1`<br>`exit`

I then set up DHCP exclusions for IP address ranges 10.0.0.1-2 and 10.0.128.1-2. 10.0.0.1 and 10.0.128.1 are reserved as the default gateways for the GUEST (vlan 10) and STUDENT (vlan 30) DHCP pools, respectively. 
<br>`ip dhcp excluded-address 10.0.0.1 10.0.0.2`<br>`ip dhcp excluded-address 10.0.128.1 10.0.128.2`

I set up DHCP exclusions for the STAFF pool (reserved for the default gateway, vlan 20's address, and other static devices that may be hypothetically added later on, such as printers)
<br>`ip dhcp excluded-address 10.0.64.1 10.0.64.20`

For security purposes, I set up dhcp exclusions for unused addresses on the fourth subnet. These addresses may be included in the future, should the need for expansions arise.
<br>`ip dhcp excluded-address 10.0.192.1 10.0.255.255`
<br>`end`<br>`write memory`

### Configuring VLANs on Switch0

Switching over to Switch0, I assigned interface `g0/1` as a trunk port so that the three VLANs configured in the next steps can all route their traffic through one port. I used the following commands:
<br>`en` <br>`conf t` <br>`int g0/1` <br>`no shutdown` <br>`switchport mode trunk`

I first assigned the Guest VLAN as vlan 10 to fa0/1 at switch0.
<br>`en` <br>`conf t` <br>`int fa0/1` <br>`switchport mode access` <br>`switchport access vlan 10` <br>`int vlan 10` <br>`ip addr 10.0.0.2 255.255.192.0` <br>`no shutdown` <br>`exit`

I then assigned the Staff VLAN as vlan 20 to fa0/2 at switch0.
<br>`en` <br>`conf t` <br>`int fa0/2` <br>`switchport mode access` <br>`switchport access vlan 20` <br>`int vlan 20` <br>`ip addr 10.0.64.2 255.255.192.0` <br>`no shutdown` <br>`exit`

Finally, I assigned the Student VLAN as vlan 30 to fa0/3 at switch0.
<br>`en` <br>`conf t` <br>`int fa0/3` <br>`switchport mode access` <br>`switchport access vlan 30` <br>`int vlan 30` <br>`ip addr 10.0.128.2 255.255.192.0` <br>`no shutdown` <br>`exit` <br>`end` <br>`write memory`

### Adding laptops to the configuration

<br>Guest Laptop 1 – `10.0.0.3` (via DHCP) <br>Guest Laptop 2 – `10.0.0.4` (via DHCP) <br>Staff Laptop 1 – `10.0.64.22` (via DHCP) <br>Staff Laptop 2 – `10.0.64.21` (via DHCP) <br>Student Laptop 1 – `10.0.128.4` (via DHCP) <br>Student Laptop 2 – `10.0.128.3` (via DHCP)

**Note:** IP addresses may change each time the network restarts due to DHCP.

Testing the configuration

In the below screenshot, Guest Laptop 1 (`10.0.0.3

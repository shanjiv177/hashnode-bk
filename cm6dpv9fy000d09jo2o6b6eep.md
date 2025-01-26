---
title: "Installing and Configuring mwan3 in OpenWrt for Failover"
seoTitle: "Installing and Configuring mwan3 in OpenWrt for Failover"
seoDescription: "mwan3 is for multiwan load balancing/failover. The article goes through configuring mwan3 through its members, policies and rules."
datePublished: Sun Jan 26 2025 14:30:13 GMT+0000 (Coordinated Universal Time)
cuid: cm6dpv9fy000d09jo2o6b6eep
slug: installing-and-configuring-mwan3-in-openwrt-for-failover
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1737902393734/f9557141-57da-4fd6-86bc-d37e14d83b4c.webp
tags: networking, routers, openwrt, mwan3

---

## Overview

mwan3 (Multi-WAN Load Balancer) is a package in OpenWrt that enables advanced multi-WAN routing policies, including failover, load balancing, and custom traffic routing. This guide will walk you through the process of flashing OpenWrt, setting up uplinks, and configuring mwan3 for failover purposes.

---

## Prerequisites

1. **Device for OpenWrt**: An OpenWRT compatible device [Compatabile Devices](https://firmware-selector.openwrt.org/)
2. **OpenWrt Image**: Download the appropriate OpenWrt image for your hardware from the [OpenWrt website](https://openwrt.org/).
3. **Multiple WAN Connections**: You need at least two WAN interfaces (e.g., WAN1 and WAN2).
4. **Internet Connectivity**: Ensure you have an active internet connection on at least one interface to install packages.
5. **Additional Hardware**: For Raspberry Pi or similar setups, you may need a USB-to-Ethernet adapter for additional network interfaces.

---

## Step 1: Flashing OpenWrt

### For Raspberry Pi or similar devices

1. **Download the OpenWrt Image**:

   - Visit the [OpenWrt firmware selector](https://firmware-selector.openwrt.org/).
   - Download the appropriate image for your device.

2. **Flash the Image to the SD Card**:

   - Use tools like `balenaEtcher`, `Rufus`, or `dd` (Linux/Mac) to flash the downloaded image to the SD card.

3. **Insert and Boot**:

   - Insert the SD card into the device and power it on.
   - Connect to the router (`192.168.1.1`).

### For Routers

1. **Download the OpenWrt Image**:

   - Visit the [OpenWrt firmware selector](https://firmware-selector.openwrt.org/).
   - Download the firmware for your router model.

2. **Flash OpenWrt**:

   - Access your router's existing firmware GUI.
   - Locate the firmware upgrade section and upload the OpenWrt image.
   - Follow the on-screen instructions to complete the flashing process.

3. **Connect and Access**:

   - Once flashed, connect to the router (`192.168.1.1`).

---

## Step 2: Setting Up Network Interfaces (Uplinks)

1. **Connect WAN Interfaces**:

   - For Raspberry Pi or similar devices: Connect your primary WAN (e.g., your modem) to the built-in Ethernet port and secondary WAN (e.g., USB-to-Ethernet adapter, or USB tethering from a phone) to a USB port.
   - For Routers: Use the WAN and LAN ports as per your device's layout.
   Note: If you plan to use USB tethering with an Android device, ensure you install the `kmod-usb-net-rndis` package. For iPhones, install the appropriate USB tethering kernel modules for compatibility.

2. **Access the OpenWrt Web Interface**:

   - Navigate to `http://192.168.1.1` in your browser.
   - Log in with the default username `root` and no password (set a password immediately).

3. **Configure WAN Interfaces**:

   - Go to **Network > Interfaces**.
   - Edit the default `WAN` interface:
     - Set the protocol to `DHCP`.
     - Save and apply.
   - Add a new interface for the second WAN:
     - Name: `WAN1`.
     - Protocol: `DHCP`.
     - Device: Select the USB-to-Ethernet adapter (e.g., `eth1`) or router's secondary WAN port.
     - Save and apply.

---

## Step 3: Installing mwan3

1. SSH into your device:
   ```bash
   ssh root@192.168.1.1
   ```
2. Update package lists:
   ```bash
   opkg update
   ```
3. Install mwan3:
   ```bash
   opkg install mwan3
   ```
4. Install the LuCI interface for easier configuration (optional):
   ```bash
   opkg install luci-app-mwan3
   ```

---

## Step 4: Configuring mwan3

**Note** : All this could be configured in the luci interface too. Head over to **Network > MultiWAN Manager**.

1. **Define Interfaces**:
   - Navigate to **Network > MultiWAN Manager** in LuCI or edit `/etc/config/mwan3` directly.
   - Add your WAN interfaces. Example:
     ```
     config interface 'wan'                  
        option enabled '1'              
        option family 'ipv4'            
        option reliability '1'          
        option initial_state 'online'   
        option track_method 'ping'      
        option count '1'                
        option size '56'                
        option max_ttl '60'             
        option timeout '4'              
        option interval '10'            
        option failure_interval '5'     
        option recovery_interval '5'    
        option down '5'                 
        option up '5'                   
        list track_ip '8.8.4.4'         
        list track_ip '8.8.8.8'

     config interface 'wan1'                 
        option enabled '1'           
        option initial_state 'online'
        option family 'ipv4'         
        list track_ip '1.0.0.1'      
        list track_ip '1.1.1.1'         
        option track_method 'ping'      
        option reliability '1'          
        option count '1'                
        option size '56'                
        option max_ttl '60'             
        option timeout '4'              
        option interval '10'            
        option failure_interval '5'     
        option recovery_interval '5'    
        option down '5'                 
        option up '5'
     ```

- **Explanation**:
  - `track_ip`: IPs to ping for checking WAN health.
  - `family`: Specify `ipv4` or `ipv6`.
  - `reliability`: Number of successful pings required to mark the interface as up.
  - `interval`: Time in seconds between pings.
  - `down` / `up`: Number of consecutive failures or successes to mark the interface as down or up.
  - `count`: Count of pings to send in one check.
  - `size`: Size of ICMP echo packets. (56 is default)
  - `ttl_max` : Defines the maximum TTL (Time To Live) value for the ICMP packets. This option is used to limit how many hops the packet can make before being discarded.
  - `timeout`: Defines the number of seconds to wait for a reply to the ping request before considering it a failure.
  - `failure_interval/recovery_interval`: Specifies how long must a interface be down or up to be considered failed or recovered respectively.
  
2. **Define Members**:
   ```
   config member 'wan_m1_w1'
       option interface 'wan'
       option metric '1'
       option weight '1'
       
   config member 'wan1_m2_w1'
       option interface 'wan2'
       option metric '2'
       option weight '1'
   ```
- **Explanation**:
  - `interface`: Associated interface's name.
  - `metric`: Priority. An interface with a higher metric will carry the traffic.
  - `weight`: Decides the division of traffic for interfaces on the same metric/priority.
    
3. **Define Failover Policies**:
   ```
   config policy 'failover'
       option last_resort 'unreachable'
       list use_member 'wan_m1'
       list use_member 'wan2_m2'
       
   ```
- **Explanation**:
   - `last_resort`: Determines what happens when no interfaces in the policy are available.
      - `unreachable`: Traffic will not be routed, and the source will receive a "Destination Unreachable" response.
      - `blackhole`: Traffic will be silently dropped without notifying the source.
      - `default`: Traffic will be sent using the system's default routing table.

4. **Set Default Routing Policy**:
   ```
   config rule 'https'                     
        option sticky '1'               
        option dest_port '443'      
        option proto 'tcp'          
        option use_policy 'failover'
                                    
   config rule 'default_rule_v4' 
        option dest_ip '0.0.0.0/0'
        option use_policy 'failover'
        option family 'ipv4'        
        option proto 'all'   
        option sticky '0'         
                                    
   config rule 'default_rule_v6'       
        option dest_ip '::/0'
        option use_policy 'failover'
        option family 'ipv6'        
        option proto 'all'   
        option sticky '0'
   ```
- **Explanation**:
   - `sticky`: Ensures session persistence.
     - `1`: Enabled. All connections from the same source IP will use the same WAN interface during a session.
   - `dest_port`: Specifies the destination port.
   - `proto`: Defines the protocol.
   - `use_policy`: Uses the `failover` policy for routing this traffic.
   - `family`: Specify `ipv4` or `ipv6`.

---

## Step 5: Testing Failover

1. Restart mwan3:
   ```bash
   /etc/init.d/mwan3 restart
   ```
2. Check mwan3 status:
   ```bash
   mwan3 status
   ```
   - Ensure both WAN interfaces are monitored and the failover is active.
3. Simulate a failure:
   - Disconnect the primary WAN interface and check if traffic fails over to the secondary interface.

---

## Troubleshooting

- **Interfaces not switching**: Verify `track_ip` addresses are reachable and accurate.
- **WAN interfaces not up**: Check physical connections and IP configurations.
- **Logs show frequent failures**: Adjust `interval`, `down`, and `up` values to reduce sensitivity.

**Note**: You can check the status of the interfaces at **Status > MultiWAN Manager**

---
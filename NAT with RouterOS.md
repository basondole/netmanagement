# NAT Rules for MikroTik routers

## NAT for Internet access

**Command:**  
`/ip firewall nat add chain=srcnat action=masquerade out-interface=<WAN_INTERFACE>`  

The masquerading will change the source IP address and port of the packets to the IP address of WAN interface when the packet is routed through it. 

You can also specify interface list instead of specific WAN interface  
> Interface-list is a feature in RouterOS where you can group several interfaces together and use the list to reference all interfaces in that group   

Below command does the same thing the first does only using an interface list instead of specific interface  

**Command:**  
`/ip firewall nat add chain=srcnat action=masquerade out-interface-list=<WAN_INTERFACE_LIST>`


### Altenative method
You can also use a specific WAN IP to do NAT with instead of using interface address

**Command:**  
<pre>/ip firewall nat
add chain=srcnat src-address=<LAN_BLOCK> action=src-nat to-addresses=<PUBLIC_IP> out-interface=<WAN_INTERFACE></pre>

Or use an interface list  
**Command:** <pre>/ip firewall nat
add chain=srcnat src-address=<LAN_BLOCK> action=src-nat to-addresses=<PUBLIC_IP> out-interface-list=<WAN_INTERFACE_LIST></pre>

This will change the source IP address and port of the packets from LAN_BLOCK to the IP address PUBLIC_IP



## Port Forwarding WAN to LAN
Below WAN port 5901 is forwarded to LAN ip 192.168.3.100
**Command:**
<pre>
/ip firewall nat
add chain=dstnat protocol=tcp dst-address=<PUBLIC_IP> dst-port=5901 action=dst-nat to-addresses=192.168.3.100
</pre>

Below WAN port 5901 is forwarded to LAN ip 192.168.3.100 port 80  
**Command:**
<pre>
/ip firewall nat
add chain=dstnat protocol=tcp dst-port=5901 action=dst-nat to-addresses=192.168.3.100 to-ports=80
</pre>
This means when an incoming connection requests TCP port 5901 use the DST-NAT action and redirect it to local address 192.168.3.100 and the port 80


## Mapping
### One to One Mapping: IP to IP
**Command:**
```
/ip firewall nat add chain=dstnat dst-address=<PUBLIC_IP> action=dst-nat to-addresses=<PRIVATE_IP>
/ip firewall nat add chain=srcnat src-address=<PRIVATE_IP> action=src-nat to-addresses=<PUBLIC_IP>
```

### One to One Mapping: Subnet to Subnet
**Command:**
```
/ip firewall nat add chain=dstnat dst-address=<SUBNET_1> action=netmap to-addresses=<SUBNET_2>
/ip firewall nat add chain=srcnat src-address=<SUBNET_2> action=src-nat to-addresses=<SUBNET_1>
```

## Port Translation
Below incoming port 2222 is translated to 22

**Command:**  
`/ip firewall nat add action=dst-nat chain=dstnat dst-address=<PUBLIC_IP> dst-port=2222 protocol=tcp to-ports=22`

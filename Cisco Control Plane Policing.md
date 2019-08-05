# Control Plane policing
Control plane policing is used to control traffic destined to the control plane of the router. This reduces strain on our routers CPU by controlling the traffic destined for the router itself.

## Configuring Control Plane Policing on Cisco IOS
Configuration of the control plane policing takes a modular approach as described below
1. Constructing the CoPP Policy 
We use access-list to match the traffic of interest then we create a class-map that matches the access-list and finally we create a class-map to tie them together.
In summary:
   -	access-list to match the traffic
   -	class-map to match the ACL
   -	policy-map to police the class-maps

2. Deploying the CoPP Policy

3. Verifying the CoPP Policy



## Configuration Sample

### Constructing the CoPP Policy  
The policy will be used do administer the below components  
- Allow trusted sources  
><em>This is for all the trusted IPs that are allowed total control to the device</em>
- Remote access control  
><em>This is for what is allowed to access thee vty lines</em>  
- BGP control  
><em>This is for what is allowed to form BGP sessions with the device</em>
- SNMP control  
><em>This is for what can perform SNMP operations on the device</em>
- ICMP rate limiting
><em>This is to limit the ICMP traffic destined to the device</em>
- BFD control
><em>This specifies what is allowed to form BFD neighborship with the device</em>
- LDP control
><em>This specifies what is allowed to form LDP session with the device</em>
- DNS control
><em>This specifies what is allowed to send DNS requests to/from the device</em>
- RIP control
><em>This specifies what is allowed to run RIP with the device</em>
- OSPF control
><em>This specifies what is allowed to run OSPF with the device</em>
- NTP control
><em>This specifies what is allowed to be used for NTP</em>
- GRE control
><em>This specifies what is allowed to form GRE tunnels with the device</em>
- PIM control
><em>This specifies what is allowed to run PIM with the device</em>
- IGMP control
><em>This specifies what is allowed to run IGMP with the device</em>
- Explicit block
><em>This block everything else that is did not match the above and may not be required depending on your design</em>



Allow trusted source
<pre>
ip access-list standard CoPP-ACL-match-trusted-hosts
	permit host 10.21.41.13
  
class-map match-all CoPP-CLASS-allow-trusted-hosts
	match access-group name CoPP-ACL-match-trusted-hosts

policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-trusted-hosts
</pre>

Controlling remote-access only from specified ip blocks
<pre>
ip access-list extended CoPP-ACL-match-telnet  
	permit tcp host 172.18.162.94 any eq telnet
ip access-list extended CoPP-ACL-match-ssh
	permit tcp host 172.18.162.94 any eq 22
	permit tcp 10.62.4.16 0.0.0.15 any eq 22
ip access-list extended CoPP-ACL-match-http
	permit tcp 10.62.4.16 0.0.0.15 any eq www
ip access-list extended CoPP-ACL-match-https
	permit tcp 10.62.4.16 0.0.0.15 any eq 443

ip access-list extended CoPP-ACL-match-telnet-ssh-http	
	permit tcp any any eq 22
	permit tcp any any eq telnet
	permit tcp any any eq www
	permit tcp any any eq 443

class-map match-any CoPP-CLASS-allow-remote-access
	match access-group name CoPP-ACL-match-telnet
	match access-group name CoPP-ACL-match-ssh
	match access-group name CoPP-ACL-match-http	
	match access-group name CoPP-ACL-match-https

class-map match-all CoPP-CLASS-deny-unknown-remote-access
	match access-group name CoPP-ACL-match-telnet-ssh-http

policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-remote-access
  class CoPP-CLASS-deny-unknown-remote-access
     police 8000 conform-action drop  exceed-action drop  violate-action drop
</pre>

Controlling networks allowed to form BGP session  
<pre>
ip access-list extended CoPP-ACL-match-bgp 	
	permit tcp 10.0.0.0 0.255.255.255 gt 1024 any eq bgp
	permit tcp 10.0.0.0 0.255.255.255 eq bgp any gt 1024 established
	permit tcp 172.16.0.0 0.0.15.255 any eq bgp
	permit tcp 172.16.0.0 0.0.15.255 eq bgp any gt 1024 established
	
class-map match-all CoPP-CLASS-allow-bgp
	match access-group name CoPP-ACL-match-bgp

policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-bgp

</pre>

Controlling SNMP access to the device  
<pre>
ip access-list extended CoPP-ACL-match-snmp
	permit udp 10.62.4.16 0.0.0.15 any eq snmp

class-map match-all CoPP-CLASS-allow-snmp
	match access-group name CoPP-ACL-match-snmp
  
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-snmp
</pre>

Limiting ICMP traffic to 1Mbps
<pre>
ip access-list extended CoPP-ACL-match-icmp	
	permit icmp any any

class-map match-all CoPP-CLASS-allow-icmp
	match access-group name CoPP-ACL-match-icmp

policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-icmp
     police 1000000 conform-action transmit  exceed-action drop  violate-action drop	
</pre>

Controlling networks allowed to form BFD
<pre>
ip access-list extended CoPP-ACL-match-bfd
	permit udp any any eq 3784 

class-map match-all CoPP-CLASS-allow-bfd
	match access-group name CoPP-ACL-match-bfd

policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-bfd
</pre>

Controlling networks allowed to form LDP sessions
<pre>
ip access-list extended CoPP-ACL-match-ldp	
	permit udp 10.188.128.0 0.0.1.255 any eq 646
	permit tcp 10.188.128.0 0.0.1.255 any eq 646
	permit tcp 10.188.128.0 0.0.1.255 eq 646 any established
	permit udp 172.16.0.0 0.0.3.255 any eq 646
	permit tcp 172.16.0.0 0.0.3.255 any eq 646
	permit tcp 172.16.0.0 0.0.3.255 eq 646 any established

class-map match-all CoPP-CLASS-allow-ldp
	match access-group name CoPP-ACL-match-ldp

policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-ldp
</pre>

Controlling NTP
<pre>
ip access-list extended CoPP-ACL-match-ntp	
	permit udp host 192.168.41.9 any eq ntp 	
class-map match-all CoPP-CLASS-allow-ntp
	match access-group name CoPP-ACL-match-ntp
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-ntp
</pre>

Controlling DNS
<pre>
ip access-list extended CoPP-ACL-match-dns	
	permit udp host 192.168.1.101 eq domain any
	permit udp host 192.168.2.100 eq domain any
class-map match-all CoPP-CLASS-allow-dns
	match access-group name CoPP-ACL-match-dns
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-dns
</pre>

Controlling RIP
<pre>
ip access-list extended CoPP-ACL-match-rip	
	permit udp any any eq rip
class-map match-all CoPP-CLASS-allow-rip	
	match access-group name CoPP-ACL-match-rip
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-rip
</pre>

Controlling OSPF
<pre>
ip access-list extended CoPP-ACL-match-ospf
	permit ospf any any
class-map CoPP-CLASS-allow-ospf
	match access-group name CoPP-ACL-match-ospf
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-ospf
</pre>

Controlling IGMP
<pre>
ip access-list extended CoPP-ACL-match-igmp
	permit igmp any any
class-map CoPP-CLASS -igmp
	match access-group name CoPP-ACL-match-igmp
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-igmp
    police 8000 conform-action drop  exceed-action drop  violate-action drop
</pre>

Controlling GRE
<pre>
ip access-list extended CoPP-ACL-match-gre
	permit gre any any
class-map match-any CoPP-CLASS-allow-gre
	match access-gro name CoPP-ACL-match-gre
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-allow-gre
</pre>

Controlling PIM
<pre>
ip access-list extended CoPP-ACL-match-pim
 permit pim any any
class-map match-any CoPP-CLASS-pim
 match access-group name CoPP-ACL-match-pim
policy-map CoPP-POLICY-protect-routing-engine
  class CoPP-CLASS-pim
    police 8000 conform-action drop  exceed-action drop  violate-action drop
</pre>

Block everything else  
<pre>
ip access-list extended CoPP-ACL-match-tcp-udp-ip-icmp-garbage
	permit tcp any any
	permit udp any any
	permit ip any any
	permit icmp any any
ip access-list standard CoPP-ACL-match-any-unauthorized-source
  permit any

class-map match-any CoPP-CLASS-drop
	match access-group name CoPP-ACL-match-tcp-udp-ip-icmp-garbage
	match access-group name CoPP-ACL-match-any-unauthorized-source

policy-map CoPP-POLICY-protect-routing-engine
	class CoPP-CLASS-drop
		police 8000 conform-action drop  exceed-action drop  violate-action drop
</pre>


## Deploying the policy map  
Use below command to apply the created policy map to the control plane

<pre>
control-plane
	service-policy input CoPP-POLICY-protect-routing-engine
</pre>

### Verifying the CoPP Policy
To verify run the command
`show policy-map control-plane`

You can also hone in to look at a specific class in the policy map by specifying the class name  
`show policy-map control-plane input class class-name`

### Testing
A hacker tries to access the router via telnet and they get blocked
<pre>
hacker#telnet 41.188.128.46
Trying 41.188.128.46 ... 
% Connection timed out; remote host not responding

hacker#telnet 41.188.128.46
Trying 41.188.128.46 ... 
% Connection timed out; remote host not responding
</pre>

To verify the action taken by the router

<pre>
show policy-map control-plane input class CoPP-CLASS-deny-vty
 Control Plane 
  Service-policy input: CoPP-POLICY-protect-routing-engine
    Class-map: CoPP-CLASS-deny-vty (match-any)  
      4 packets, 256 bytes
      5 minute offered rate 0000 bps, drop rate 0000 bps
      Match: access-group name CoPP-ACL-match-telnet-ssh-http
        4 packets, 256 bytes
        5 minute rate 0 bps
      police:
          cir 8000 bps, bc 1500 bytes, be 1500 bytes
        <b>conformed 4 packets, 256 bytes; actions:
          Drop</b> 
        exceeded 0 packets, 0 bytes; actions:
          drop 
        violated 0 packets, 0 bytes; actions:
          drop 
        conformed 0000 bps, exceeded 0000 bps, violated 0000 bps
</pre>


# Essential Huawei OLT CLI Commands

### Checking OLT Uptime
<pre>
OL01#display sysuptime
System up time: 324 day 20 hour 55 minute 38 second
</pre>

### Checking OLT Board
> SCUN  boards are the Control Module  
> X2CS boards are the uplink ports  
<pre>
OL01#display board 0

————————————————————————-
SlotID  BoardName  Status          SubType0 SubType1    Online/Offline
————————————————————————-
0
1       H805GPFD   Normal
2       H805GPFD   Normal
3       H805GPFD   Normal
4       H805GPFD   Normal
5       H805GPFD   Normal
6       H805GPFD   Normal
7       H802SCUN   Active_normal  
8       H802SCUN   Standby_normal  
9       H805GPFD   Normal
10      H805GPFD   Normal
11      H805GPFD   Normal
12      H805GPFD   Normal
13      H805GPFD   Normal
14      H805GPFD   Normal
15      H805GPFD   Normal
16      H805GPFD   Normal
17      H801X2CS   Normal   
18      H801X2CS   Normal
19      H801PRTE   Normal
20      H801PRTE   Normal
————————————————————————-
</pre>

### Checking OLT processor and Memory per Slot
<pre>
OL01#display mem 0/1
Memory occupancy: 33%
OL01#display cpu 0/1
Send message for inquiring board cpu occupancy successfully, board executing…
CPU occupancy: 15%
</pre>

### Checking OLT Temperature
<pre>
OL01#display temperature 0
SlotID:  1      BoardName: H805      Temperature:   42C( 107F)
SlotID:  2      BoardName: H805       Temperature:   41C( 105F
</pre>

### Finding ONT by SN number
<pre>
OL01#display ont info by-sn ABCDEF
—————————————————————————–
In Huawei port 1 = port 0 at system
F/S/P                   : 0/1/0 
ONT-ID                  : 2
Control flag            : active
Run state               : offline
Config state            : initial
Match state             : initial
DBA type                : SR
ONT distance(m)         : –
ONT battery state       : –
Memory occupation       : –
CPU occupation          : –
Temperature             : –
Authentic type          : SN-auth
SN                      : ABCDEF (HWTC-07930A53)
Management mode         : OMCI
Software work mode      : normal
Isolation state         : normal
ONT IP 0 address/mask   : –
Description             : IRONMAN
Last down cause         : dying-gasp 
Last up time            : 2016-11-11 17:52:08+07:00 
Last down time          : 2016-11-15 09:23:27+07:00
Last dying gasp time    : 2016-11-15 09:23:27+07:00
ONT online duration     : –
Type C support          : Not support
Interoperability-mode   : Unknown
</pre>

### Checking ONT Status
<pre>
OL01#display ont info 0 1 0 all
—————————————————————————–
F/S/P   ONT         SN         Control     Run      Config   Match    Protect
ID                     flag        state    state    state    side
—————————————————————————–
0/ 1/0    1  ABCDEF  deactivated offline  initial  initial  no
0/ 1/0    2  REZABU  active      offline  initial  initial  no 
0/ 1/0    4  BAREZU  active      online   normal   match    no  

</pre>

### Checking ONT History Logs
<pre>
OL01#config
OL01(config)#interface gpon 0/1
OL01(config-if-gpon-0/1)#display ont register-info 0 1
——————————————————————————
Index               : 10
Auth-type           : SN-auth
SN                  : ABCDEF (HWTC-19698F53)
TYPE                : 245H
UpTime              : 2016-11-02 16:56:37+07:00
DownTime            : 2016-11-04 07:52:15+07:00
DownCause           : ONT dying-gasp
——————————————————————————
Index               : 9
Auth-type           : SN-auth
SN                  : ABCDEF (HWTC-19698F53)
TYPE                : 245H
UpTime              : 2016-10-31 22:29:05+07:00
DownTime            : 2016-11-02 16:42:36+07:00
DownCause           : ONT dying-gasp
——————————————————————————
Index               : 8
Auth-type           : SN-auth
SN                  : ABCDEF (HWTC-19698F53)
TYPE                : 245H
UpTime              : 2016-10-29 00:25:35+07:00
DownTime            : 2016-10-31 22:13:03+07:00
DownCause           : ONT dying-gasp
</pre>

### Checking ONT mac address
<pre>
OL01#display mac-address port 0/1/0 ont 2
———————————————————————–
SRV-P BUNDLE TYPE MAC            MAC TYPE F /S /P   VPI  VCI   VLAN ID
INDEX INDEX
———————————————————————–
2389     –  gpon aaaa-bbbb-cccc dynamic  0 /1 /0   2    2         122
2388     –  gpon bbbb-aaaa-cccc dynamic  0 /1 /0   2    1         212
———————————————————————–
Total: 2
Note: F–Frame, S–Slot, P–Port, F/S/P indicates PW Index for PW,
A–The MAC address is learned or configured on the aggregation port,
VPI indicates ONT ID for PON, VCI indicates GEM index for GPON,
v/e–vlan/encap, pritag–priority-tagged,
ppp–pppoe, ip–ipoe, ip4–ipv4oe, ip6–ipv6oe
</pre>

### Checking ONT optical power
<pre>
OL01#config
OL01(config)#interface gpon 0/1
OL01(config-if-gpon-0/1)#display ont optical-info 0 all
—————————————————————————–
ONT  Rx power    Tx power    OLT Rx ONT  Temperature  Voltage     Current
ID   (dBm)       (dBm)       power(dBm)  (C)          (V)         (mA)
—————————————————————————–
4  -23.10      2.30        -24.21      56           3.260       13
6  -21.08      2.28        -21.49      44           3.240       13
</pre>


### Checking ONT Version
<pre>
OL01#display ont version 0 1 0 4
————————————————————————–
F/S/P                    : 0/1/0
ONT-ID                   : 4
Vendor-ID                : HWTC
ONT Version              : 49
Product-ID               : 4
Equipment-ID             : 24
Main Software Version    : V3R0
Standby Software Version : V3R0
OntProductDescription    : EchoLife
Support XML Version      : 1.3.
OL01#
</pre>

### Checking ONT IP (Routing Mode)
<pre>
OL01#display ont wan-info 0/1 0 4
———————————————————————
F/S/P                      : 0/1/0
ONT ID                     : 4
———————————————————————
Index                      : 1
Name                       : OLT_C_INTERNET_DHCP_WAN
Service type               : Internet
Connection type            : IP routed
IPv4 Connection status     : Connected
IPv4 access type           : DHCP
IPv4 address               : 1.2.3.4
Subnet mask                : 255.255.255.0
Default gateway            : 1.2.3.1
Manage VLAN                : 212
Manage priority            : 0
Option60                   : Yes
Switch                     : Enable
MAC address                : BBBB-AAAA-CCCC
Priority policy            : Specified
L2 encap-type              : IPoE
IPv4 switch                : Enable
IPv6 switch                : Disable
———————————————————————
</pre>

### Checking ONT port status
<pre>
OL01#config
OL01(config)#interface gpon 0/1
OL01(config-if-gpon-0/1)#display ont port state 0 4 eth-port all
————————————————————————–
ONT-ID   ONT      ONT       Speed(Mbps)   Duplex   LinkState  RingStatus
port-ID  Port-type
————————————————————————–
4         1         GE –             –        down       –
4         2         GE 100           full     up         –
4         3         GE –             –        down       –
4         4         GE –             –        down       –
————————————————————————–
</pre>

### Checking ONT configuration
> Port 1 and Port 4 are in bridge mode
> Port 2 and Port 3 are in routed mode
<pre>
OL01#display current-configuration ont 0/1/0 4
#
[gpon]
<gpon-0/1>
interface gpon 0/1
ont add 0 4 sn-auth “SN” omci ont-lineprofile-id 5
ont-srvprofile-id 1 desc “SERVICE1”
ont ipconfig 0 4 dhcp vlan 212 priority
ont internet-config 0 4 ip-index 0
ont wan-config 0 4 ip-index 0 profile-id 1
ont port route 0 4 eth 1 enable 
ont port native-vlan 0 4 eth 2 vlan 122 priority 0 
ont port native-vlan 0 4 eth 3 vlan 122 priority 0 
ont port route 0 4 eth 4 enable 
#
[bbs-config]
<bbs-config>
service-port 2677 vlan 212 gpon 0/1/0 ont 4 gemport 1 multi-service user-vlan  212 tag-transform translate inbound traffic-table index 20 outbound
traffic-table index 20
service-port 2678 vlan 122 gpon 0/1/0 ont 4 gemport 2 multi-service user-vlan  122 tag-transform translate inbound traffic-table index 0 outbound traffic-table index 0
return
</pre>

## Checking PON Port Configuration
<pre>
OL01#display current-configuration port 0/1/1
#
[gpon]
<gpon-0/1>
interface gpon 0/1
port 1 ont-auto-find enable
</pre>

### Checking Service Ports in ONT
<pre>
OL01#display service-port port 0/1/0 ont 4
{ <cr>|e2e<K>|gemport<K>|sort-by<K> }:
Command:
display service-port port 0/1/0 ont 4
Switch-Oriented Flow List
—————————————————————————–
INDEX VLAN VLAN     PORT F/ S/ P VPI  VCI   FLOW  FLOW       RX   TX   STATE
ID   ATTR     TYPE                    TYPE  PARA
—————————————————————————–
2677  212 common   gpon 0/1 /0  4    1     vlan  212        20   20   up
2678  122 common   gpon 0/1 /0  4    2     vlan  122        0    0    up
—————————————————————————–
Total : 2  (Up/Down :    2/0)
Note : F–Frame, S–Slot, P–Port,
VPI indicates ONT ID for PON, VCI indicates GEM index for GPON,
</pre>

### Ping Test From ONT
<pre>
OL01#config
OL01(config)#interface gpon 0/1
OL01(config-if-gpon-0/1)#ont remote-ping 0 4 ip-address 10.10.8.8
OL01(config-if-gpon-0/1)#
—————————————–
ONT remote-ping information
—————————————–
F/S/P                : 0/ 1/0
ONT-ID               : 4
ONT IP Index         : 0
IP address of ping   : 8.8.8.8
Transmit packets     : 4
Receive packets      : 4
Lost packets         : 0
Lost packets ratio(%): 0
Min delay(ms)        : 31
Max delay(ms)        : 31
Average delay(ms)    : 31
Error Code           : –
—————————————–
</pre>

### Reboot ONT
<pre>
OL01#config
OL01(config)#interface gpon 0/1
OL01(config-if-gpon-0/1)#ont reset 0 4
</pre>

### Factory Reset  ONT
<pre>
OL01#config
OL01(config)#interface gpon 0/1
OL01(config-if-gpon-0/1)#ont factory-setting-restore 0 4
</pre>

### How To Release Renew ONT as DHCP client
<pre>
OL01#config
OL01(config)#interface gpon 0/1
OL01(config-if-gpon-0/1)#ont ipconfig 0 4 dhcp reset
Checking Rx Level For OLT Uplink
OL01#config
OL01(config)#interface giu 0/18
OL01(config-if-giu-0/18)#display port ddm-info 0
Temperature(C)                       : 38.640000
Supply voltage(V)                    : 3.260000
TX bias current(mA)                  : 38.000000
TX power(dBm)                        : 1.095110
RX power(dBm)                        : -14.659738
OL01(config-if-giu-0/18)#quit
OL01(config)#quit
</pre>

### Reset Slot
<pre>
OL01#config
OL01(config)#board reset 0/1
</pre>

### Locate mac address learnt from ONT
Use case is when you know the mac address but you do not know it comes in via which ONT
<pre>
OL01#display location aaaa-bbbb-cccc
{ <cr>|fabric<K>|ont<K> }:
Command:
display location 4cf9-5dd3-1914
It will take several minutes, and console may be timeout, please use command  idle-timeout to set time limit
Are you sure to query MAC address location ? (y/n)[n]:y
———————————————————————–
SRV-P BUNDLE TYPE MAC            MAC TYPE F /S /P   VPI  VCI   VLAN ID
INDEX INDEX
———————————————————————–
5493     –  gpon aaaa-bbbb-cccc dynamic  0 /6 /6   50   1         203
———————————————————————–
</pre>

### Checking statistic from uplink port
<pre>
OL01-HW#config
OL01-HW(config)#interface giu 0/18
OL01-HW(config-if-giu-0/18)#display port traffic 0
The received traffic of this port(packets/s) =307216
The received traffic of this port(octets/s) =370001834
The received traffic of this port(kbits/s) =2960014.7
The transmitted traffic of this port(packets/s) =125081
The transmitted traffic of this port(octets/s) =21000637
The transmitted traffic of this port(kbits/s) =168005.1
</pre>

<h3> Traffic shapping </h3>
Traffic shaping is a bandwidth management technique used on computer networks which delays some or all datagrams to bring them into compliance with a desired traffic profile


<h4>Junos configuration</h4>

This features is supported from Junos OS Release 10.4 and later

<pre>
set class-of-service interfaces ge-2/0/8 shaping-rate 160k

paul@router# show class-of-service
interfaces {
  ge-2/0/8 {
    shaping-rate 8k;
  }
}
</pre>

Altenatively
<pre>
set class-of-service traffic-control-profiles output shaping-rate 8k
set class-of-service traffic-control-profiles output shaping-rate burst-size 1k
set class-of-service interfaces ge-2/0/8 output-traffic-control-profile output

paul@router# show class-of-service
traffic-control-profiles {
  output {
    shaping-rate 8k burst-size 1k;
  }
}
interfaces {
  ge-2/0/8 {
    output-traffic-control-profile output;
  }
}

</pre>

Verificatuion
<pre>
paul@router> show interfaces extensive ge-2/0/8
</pre>


<h4>Cisco ios configuration</h4>

<pre>
R2#sh ver | i Software
Cisco IOS Software, 7200 Software (C7200-ADVENTERPRISEK9-M), Version 12.4(24)T5, RELEASE SOFTWARE (fc3)
BOOTLDR: 7200 Software (C7200-ADVENTERPRISEK9-M), Version 12.4(24)T5, RELEASE SOFTWARE (fc3)
</pre>

We test with ping to see the transmission rate before traffic shaping
<pre>
R2#config
R2(config)#do ping  192.168.56.1 si 1500 df

Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.1, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = <b>4/11/16 ms</b>
</pre>

Configuring the traffic shaping
<pre>
R2(config)#interface FastEthernet0/0

R2(config-if)#traffic-shape rate ?
  <8000-100000000>  Target Bit Rate (bits per second)

R2(config-if)#traffic-shape rate 8000
</pre>

Verification
<pre>
R2(config-if)#do sh run int f0/0
Building configuration...

Current configuration : 126 bytes
!
interface FastEthernet0/0
 ip address 192.168.56.66 255.255.255.0
 traffic-shape rate 8000 8000 8000 1000
end

R2(config-if)#do sh traffic-shape

Interface   Fa0/0
       Access Target    Byte   Sustain   Excess    Interval  Increment Adapt
VC     List   Rate      Limit  bits/int  bits/int  (ms)      (bytes)   Active
-             8000      2000   8000      8000      1000      1000      -   


R2(config-if)#do sh traffic-shape stat
                  Acc. Queue Packets   Bytes     Packets   Bytes     Shaping
I/F               List Depth                     Delayed   Delayed   Active
Fa0/0                   0     613       61873     0         0         no

R2(config-if)#do ping  192.168.56.1 size 1500 df

Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.1, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = <b>4/923/1984 ms</b>

R2(config-if)#do sh traffic-sha sta           
                  Acc. Queue Packets   Bytes     Packets   Bytes     Shaping
I/F               List Depth                     Delayed   Delayed   Active
Fa0/0                   0     622       69683     <b>4</b>         <b>4602</b>      no
</pre>

We can see the difference between the round trip time before and after applying the shaper

Before
<pre>
R2(config)#do ping  192.168.56.1 si 1500 df

Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.1, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = <b>4/11/16 ms</b>
</pre>

After
<pre>
R2(config-if)#do ping  192.168.56.1 size 1500 df

Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.1, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = <b>4/923/1984 ms</b>
</pre>



We can use an access list to specify the traffic to be shaped
<pre>
R2(config)#access-list 199 permit ip host 192.168.56.66 host 192.168.56.1 
R2(config)#interface FastEthernet0/0
R2(config-if)#traffic-shape group 199 8000
mix of shaping with and without access lists prohibited
R2(config-if)#do sh run interface FastEthernet0/0
Building configuration...

Current configuration : 126 bytes
!
interface FastEthernet0/0
 ip address 192.168.56.66 255.255.255.0
 traffic-shape rate 8000 8000 8000 1000
end
</pre>

We get an error because we already applied another traffic shaper, we have to remove it before applying another
<pre>
R2(config-if)#no traffic-shape rate 8000
R2(config-if)#do sh run interface FastEthernet0/0
Building configuration...

Current configuration : 86 bytes
!
interface FastEthernet0/0
 ip address 192.168.56.66 255.255.255.0
end

R2(config-if)#traffic-shape group 199 8000       
R2(config-if)#do sh run interface FastEthernet0/0
Building configuration...

Current configuration : 131 bytes
!
interface FastEthernet0/0
 ip address 192.168.56.66 255.255.255.0
 traffic-shape group 199 8000 8000 8000 1000
end

R2(config-if)#end                 
</pre>

Verification
We test by ping to the ip <code>192.168.56.1</code> which is matched by the access list
<pre>
R2#ping 192.168.56.1 size 1500 df

Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.1, timeout is 2 seconds:
Packet sent with the DF bit set
!!.!!
Success rate is 80 percent (4/5), round-trip min/avg/max = <b>8/751/1988 ms</b>
R2#
R2#sh traffic-shape stat
                  Acc. Queue Packets   Bytes     Packets   Bytes     Shaping
I/F               List Depth                     Delayed   Delayed   Active
Fa0/0               199 0     5         7570      3         <b>4542</b>      no
</pre>

We can see the shaper is affecting the traffic to the point of causing packet loss,
now we can test to ping an ip that wont be matched by the access-list

<pre>
R2#ping 192.168.56.63 size 1500 df

Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.63, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/24/36 ms


R2#sh traffic-shape stat          
                  Acc. Queue Packets   Bytes     Packets   Bytes     Shaping
I/F               List Depth                     Delayed   Delayed   Active
Fa0/0               199 0     5         7570      3         4542      no
</pre>

We can see there is no increase in the number of delayed bytes and the round trip time is less.


For ios version 15 we use modular QoS to apply traffic shaping.
The Modular QoS CLI is a CLI structure that allows users to create traffic polices and attach these polices to interfaces.
A traffic policy contains a traffic class and one or more QoS features.
A traffic class is used to classify traffic, while the QoS features in the traffic policy determine how to treat the classified traffic. 

<pre>
pycon-ios#sh ver | i Software
Cisco IOS Software, 7200 Software (C7200-ADVIPSERVICESK9-M), Version 15.2(4)S5, RELEASE SOFTWARE (fc1)
BOOTLDR: 7200 Software (C7200-ADVIPSERVICESK9-M), Version 15.2(4)S5, RELEASE SOFTWARE (fc1)

pycon-ios#config)

pycon-ios(config)#class-map shape 
pycon-ios(config-cmap)#match any
pycon-ios(config-cmap)#policy-map shape

pycon-ios(config-pmap)#class shape 
pycon-ios(config-pmap-c)#shape average ?
  <8000-800000000>  Target Bit Rate (bits/sec). (postfix k, m, g optional;
                    decimal point allowed)
  percent           % of interface bandwidth for Committed information rate

pycon-ios(config-pmap-c)#shape average 8k 
</pre>

Verification
<pre>
pycon-ios(config-pmap-c)#int f0/0
pycon-ios(config-if)#do ping 192.168.56.66 df size 1500
Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.66, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = <b>20/32/60 ms</b>
   
pycon-ios(config-if)#service-policy output shape
pycon-ios(config-if)#do sh run int f0/0
Building configuration...

Current configuration : 151 bytes
!
interface FastEthernet0/0
 description management
 ip address 192.168.56.63 255.255.255.0
 <b>service-policy output shape</b>
end

pycon-ios(config-if)#do ping 192.168.56.66 df si 1500
Type escape sequence to abort.
Sending 5, 1500-byte ICMP Echos to 192.168.56.66, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = <b>68/1269/1720 ms</b>

pycon-ios(config-if)# 
pycon-ios(config-if)#end

pycon-ios#sh policy-map shape 
  Policy Map shape
    Class shape
      Average Rate Traffic Shaping
      cir 8000 (bps)

pycon-ios#sh class-map shape 
 Class Map match-all shape (id 1)
   Match any  

pycon-ios#sh policy-map shape class shape 
        Class shape
      Average Rate Traffic Shaping
      cir 8000 (bps)


pycon-ios#sh policy-map interface f0/0 output class shape 
 FastEthernet0/0 

  Service-policy output: shape

    Class-map: shape (match-all)  
      40 packets, 18463 bytes
      5 minute offered rate 0000 bps, drop rate 0000 bps
      Match: any 
      Queueing
      queue limit 64 packets
      (queue depth/total drops/no-buffer drops) 0/0/0
      (pkts output/bytes output) 40/18463
      shape (average) cir 8000, bc 32, be 32
      target shape rate 8000
</pre>



<h4>Cisco iosxr configuration</h4>
<pre>
RP/0/0/CPU0:pycon-iosxr(config)#show conf 
Tue Jan 21 07:36:06.774 UTC
Building configuration...
!! IOS XR Configuration 5.3.0
!
class-map match-any shape-class
 match protocol ipv4 
 end-class-map
!
!
policy-map shape-map
 class shape
  shape average 8 kbps 
 !
 class class-default
 !
 end-policy-map
!
interface GigabitEthernet0/0/0/0.600
 service-policy output shape-map
</pre>

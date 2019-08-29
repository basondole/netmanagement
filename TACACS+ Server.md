# Installation of TACACS+ Server on Ubuntu

Install the package
<pre>
1606:05 pbasondole:baggy@box ~ ./rw
sudo apt install tacacs+
Reading package lists... Done
Building dependency tree
Reading state information... Done
tacacs+ is already the newest version (4.0.4.27a-3).
0 upgraded, 0 newly installed, 0 to remove and 252 not upgraded.
</pre>

Backup the config file
<pre>
1606:05 pbasondole:baggy@box ~ ./rw
$ sudo cp /etc/tacacs+/tac_plus.conf /etc/tacacs+/tac_plus.conf.old
</pre>

Create file for accounting information
<pre>
1606:10 pbasondole:baggy@box ~ ./rw
$ sudo touch /var/log/tac_plus.acct
</pre>

Edit the config file and add line below specifying the tacacs secret key used for server and client packet encryption
<pre>
1621:05 pbasondole:baggy@box ~ ./rw
$ sudo nano /etc/tacacs+/tac_plus.conf

accounting file = /var/log/tac_plus.acct
key = tac123
</pre>

Create a hash of the password of the user. In this case the user is ‘fisi’ and users password is ‘fisi123’ also another test user with password test123  
Note: Usernames ARE case sensitive.
<pre>
1621:51 pbasondole:baggy@box ~ ./rw
$ tac_pwd
Password to be encrypted: fisi123
ZeJw2VrB55m82

1636:29 pbasondole:baggy@box ~ ./rw
$ tac_pwd
Password to be encrypted: test123
LXTi1.xQsU1V2
</pre>

Edit the config file to add the users
<pre>
1636:38 pbasondole:baggy@box ~ ./rw
$ sudo nano /etc/tacacs+/tac_plus.conf
user = fisi {
    name = "Fisi Hyena"
    member = admin
    login = des ZeJw2VrB55m82
    service = junos-exec {
               local-username = remote-admin
    }
}
user = test {
    name = "Test User"
    member = read-only
    login = des LXTi1.xQsU1V2
    service = junos-exec {
               local-username = remote-read-only
    }
}
</pre>

The member parameter defines the group the user is part of. There is an additional stanza service = junos-exec that defines an additional group. This is Juniper specific. When authenticating users against a TACACS+ server on juniper devices and you’ll need to apply Juniper Networks Vendor-Specific TACACS+ Attributes.  
These attributes can be either:  
1. Specified in the tac_plus.conf file by using regular expressions to list all the commands that the user has permitted or denied. A user will need to be created on the device with that user being referred under the local-user-name statement. The stanza would look something: 
<pre>
service = junos-exec {
	local-user-name = xxx
	allow-commands =  .*
	allow-configurations = .*
	deny-commands = 
	deny-configuration = 
	user-permissions = 
	}  
</pre>  
2. Configure a class that has states all the permitted or denied permissions, this class will be linked to a user. Both need to be configured on the device. Once this has been created you’ll need to refer, said user, under the local-user-name. In this example we will use this method and configure the users remote-admin and remote-read-only in the junos device.


### Group parameters
- default service: defines the default permission that the user will have. By default, if this statement isn’t used or left blank, it’s denied. Meaning that each permitted command users of this group will have to be listed. If you want the default permission to allow, then the statement permit is needed  
- service: define services which the group is authorised to execute, these could be commands that the group is authorised to execute. Authorisation must be configured on both the client and the daemon to operate correctly.  
- cmd: This is where you list a command and set an action, it will be either be a permit or deny. Additionally by having the .* this means that any command after the first word is affected. i.e my example below, all show commands will be permitted  
  
In this example there are two groups, admin and read-only, the admin group will have full access permitted and the read-only group will have read-only access and will be denied from any configuration, clear or restart commands. This may not work with juniper but works with cisco.  

Edit the config file to add the groups as we have associated our users with admin and read-only groups those are the groups we will define in the config file
<pre>
1636:38 pbasondole:baggy@box ~ ./rw
$ sudo nano /etc/tacacs+/tac_plus.conf
group = admin {
	default service = permit
	service = exec {
		priv-lvl = 15
		}
	}
group = read-only {
	service = exec {
		priv-lvl = 15
		}
	cmd = show {
		permit .*
		}
	cmd = write {
		permit term
		}
	cmd = dir {
		permit .*
		}
	cmd = admin {
		permit .*
		}
	cmd = terminal {
		permit .*
		}
	cmd = more {
		permit .*
		}
	cmd = exit {
		permit .*
		}
	cmd = logout {
		permit .*
		}
}
</pre>

For more in-depth detail and additional parameters that can be configured in this file, you can find them via the man pages using the command man tac_plus  

Run service tacacs_plus check to make sure the syntax is correct and if you get any errors you can fix and sometimes will need to restart the daemon using service tacacs_plus restart
<pre>
1649:03 pbasondole:baggy@box ~ ./rw
$ sudo service tacacs_plus check
</pre>
If there are no errors the contents of the config file will be displayed and the bottom line will say  
* Checking TACACS+ authentication daemon configuration files successful tacacs+  

## Configuring the junos device
Creating the users  
<pre>
fisi@big# show | compare
[edit system login]
+    user remote-admin {
+        full-name "TACACS RW";
+        class super-user;
+    }
+    user remote-read-only {
+        full-name "TACACS RO";
+        class read-only;
+    }
</pre>

Configuring tacacs plus
<pre>
fisi@big# show | compare
[edit system]
+  authentication-order [ tacplus password ];
+  tacplus-server {
+      192.168.56.1 {
+          secret "$9$/YQh90IEhrvMXREdb2gJZ"; ## SECRET-DATA
+          single-connection;
+          source-address 192.168.56.36;
+      }
+  }
+  accounting {
+      events [ login change-log interactive-commands ];
+      destination {
+          tacplus;
+      }
+  }
</pre>

To see the logs on the server

`less /var/log/tac_plus.acct`

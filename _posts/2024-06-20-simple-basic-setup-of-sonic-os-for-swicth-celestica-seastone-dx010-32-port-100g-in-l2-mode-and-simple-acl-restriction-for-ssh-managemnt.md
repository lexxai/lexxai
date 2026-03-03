---
layout: post
title: "Simple basic setup of SONiC OS for swicth Celestica Seastone DX010, 32-port 100G in L2 mode, and simple ACL restriction for SSH managemnt."
date: 2024-06-20 01:16:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/06/simple-basic-setup-of-sonic-os-for.html
---

In continuing to
[Hardware switch with stateless firewall rules. Seastone DX010. SONiC OS](/2024/06/hardware-switch-with-stateless-firewall.html), [SONiC Notebook](/2024/04/sonic-notebook.html).

Conditions. Two Celestica DX010 switches (Celestica1, Celestica2) are
connected via a 100G (4x25G) uplink connection located in different data
centres.  
  
Task. Manage the settings of the Celestica2 from the
console of the Celestica1 switch, but not vice versa.

#### Upgrade firmware from WiKi

|  |
| --- |
|  |
| [SONiC / Supported devices platforms](https://github.com/sonic-net/SONiC/blob/sonic_image_md_update/supported_devices_platforms.md) - Celestica DX010 #37 |

```
$ scp ./sonic-broadcom.bin scp://admin@sonic_ip  
$ ssh admin@sonic_ip
admin@sonic:~$ sudo sonic-installer install sonic-broadcom.bin  
New image will be installed, continue? [y/N]: y  
efi not supported - exiting without verification  
  
Installing image SONiC-OS-master.576207-99e0e1ade and setting it as default...  
Setup SWAP memory  
Command: mkswap /host/swapfile  
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)  
no label, UUID=c7e49ce7-321d-4a01-9af6-09123874b547  
  
Command: swapon /host/swapfile  
  
Command: bash ./sonic-broadcom.bin  
Verifying image checksum ... OK.  
Preparing image archive ... OK.  
Installing SONiC in SONiC  
ONIE Installer: platform: x86_64-broadcom-r0  
onie_platform: x86_64-cel_seastone-r0  
Removing old SONiC installation /host/image-master.571486-a2b3ca87a  
Installing SONiC to /host/image-master.576207-99e0e1ade  
....  
Command: sync  
  
Command: sync  
  
Command: sync  
  
Command: sleep 3  
  
Done  

admin@sonic:~$ sudo reboot
```

Now the user is admin, password: "YourPaSsWoRd".  
Managemnet sonic\_ip is
obtained from the local DHCP server.  
  
You can back up the sonic
configuration on your local PC   

```
$ scp -r admin@sonic_ip:/etc/sonic/ ./sonic
```

#### Apply the L2 mode preset to all ports. Create VLAN 1000 and set all ports (Trunk mode) as members of VLAN 1000 (untagged).

```
$ ssh admin@sonic_ip
admin@sonic:~$ cp /etc/sonic/config_db.json ./config_db.json-backup
admin@sonic:~$ sudo sonic-cfggen --preset l2 -p -H -k Seastone-DX010 > config_db.json
admin@sonic:~$ less config_db.json
{
    "DEVICE_METADATA": {
        "localhost": {
            "hwsku": "Seastone-DX010",
            "platform": "x86_64-cel_seastone-r0",
            "mac": "................."
        }
    },
    "PORT": {
        "Ethernet0": {
            "alias": "Eth1",
            "lanes": "65,66,67,68",
            "speed": "100000",
            "index": "1",
            "subport": "0",
            "admin_status": "up"
        },
        "Ethernet4": {
            "alias": "Eth2",
            "lanes": "69,70,71,72",
            "speed": "100000",
            "index": "2",
            "subport": "0",
            "admin_status": "up"
        },
:q
admin@sonic:~$ 
admin@sonic:~$ sudo cp config_db.json /etc/sonic/config_db.json
admin@sonic:~$ sudo config load -y
Running command: /usr/local/bin/sonic-cfggen -j /etc/sonic/config_db.json --write-to-db
admin@sonic:~$ sudo reboot
requested COLD shutdown
/var/log: 238.9 MiB (250482688 bytes) trimmed on /dev/loop1
/host: 475.2 MiB (498249728 bytes) trimmed on /dev/sda3
Watchdog armed for 180 seconds
Wed Jun 19 11:19:53 PM UTC 2024 Issuing OS-level reboot ...
```

#### Change password

```
$ ssh admin@sonic_ip

admin@sonic:~$ passwd
Changing password for admin.
Current password: .....
New password: .....
Retype new password: .....
```

#### Show interfaces status

```
admin@sonic:~$ show interfaces status
  Interface            Lanes    Speed    MTU    FEC    Alias    Vlan    Oper    Admin             Type    Asym PFC
-----------  ---------------  -------  -----  -----  -------  ------  ------  -------  ---------------  ----------
  Ethernet0      65,66,67,68     100G   9100    N/A     Eth1   trunk    down       up              N/A         N/A
  Ethernet4      69,70,71,72     100G   9100    N/A     Eth2   trunk    down       up              N/A         N/A
  Ethernet8      73,74,75,76     100G   9100    N/A     Eth3   trunk    down       up              N/A         N/A
 Ethernet12      77,78,79,80     100G   9100    N/A     Eth4   trunk    down       up              N/A         N/A
 Ethernet16      33,34,35,36     100G   9100    N/A     Eth5   trunk    down       up              N/A         N/A
 Ethernet20      37,38,39,40     100G   9100    N/A     Eth6   trunk    down       up              N/A         N/A
 Ethernet24      41,42,43,44     100G   9100    N/A     Eth7   trunk    down       up              N/A         N/A
 Ethernet28      45,46,47,48     100G   9100    N/A     Eth8   trunk    down       up              N/A         N/A
 Ethernet32      49,50,51,52     100G   9100    N/A     Eth9   trunk    down       up              N/A         N/A
 Ethernet36      53,54,55,56     100G   9100    N/A    Eth10   trunk    down       up              N/A         N/A
 Ethernet40      57,58,59,60     100G   9100    N/A    Eth11   trunk    down       up              N/A         N/A
 Ethernet44      61,62,63,64     100G   9100    N/A    Eth12   trunk    down       up              N/A         N/A
 Ethernet48      81,82,83,84     100G   9100    N/A    Eth13   trunk    down       up              N/A         N/A
 Ethernet52      85,86,87,88     100G   9100    N/A    Eth14   trunk    down       up              N/A         N/A
 Ethernet56      89,90,91,92     100G   9100    N/A    Eth15   trunk    down       up              N/A         N/A
 Ethernet60      93,94,95,96     100G   9100    N/A    Eth16   trunk    down       up              N/A         N/A
 Ethernet64     97,98,99,100     100G   9100    N/A    Eth17   trunk    down       up              N/A         N/A
 Ethernet68  101,102,103,104     100G   9100    N/A    Eth18   trunk    down       up              N/A         N/A
 Ethernet72  105,106,107,108     100G   9100    N/A    Eth19   trunk    down       up              N/A         N/A
 Ethernet76  109,110,111,112     100G   9100    N/A    Eth20   trunk    down       up              N/A         N/A
 Ethernet80          1,2,3,4     100G   9100    N/A    Eth21   trunk    down       up              N/A         N/A
 Ethernet84          5,6,7,8     100G   9100    N/A    Eth22   trunk    down       up              N/A         N/A
 Ethernet88       9,10,11,12     100G   9100    N/A    Eth23   trunk    down       up              N/A         N/A
 Ethernet92      13,14,15,16     100G   9100    N/A    Eth24   trunk    down       up              N/A         N/A
 Ethernet96      17,18,19,20     100G   9100    N/A    Eth25   trunk    down       up              N/A         N/A
Ethernet100      21,22,23,24     100G   9100    N/A    Eth26   trunk    down       up              N/A         N/A
Ethernet104      25,26,27,28     100G   9100    N/A    Eth27   trunk    down       up              N/A         N/A
Ethernet108      29,30,31,32     100G   9100    N/A    Eth28   trunk    down       up              N/A         N/A
Ethernet112  113,114,115,116     100G   9100    N/A    Eth29   trunk    down       up              N/A         N/A
Ethernet116  117,118,119,120     100G   9100    N/A    Eth30   trunk    down       up              N/A         N/A
Ethernet120  121,122,123,124     100G   9100    N/A    Eth31   trunk    down       up              N/A         N/A
Ethernet124  125,126,127,128     100G   9100    N/A    Eth32   trunk    down       up  QSFP28 or later         N/A

admin@sonic:~$ show vlan config
Name        VID  Member       Mode
--------  -----  -----------  --------
Vlan1000   1000  Ethernet0    untagged
Vlan1000   1000  Ethernet4    untagged
Vlan1000   1000  Ethernet8    untagged
Vlan1000   1000  Ethernet12   untagged
Vlan1000   1000  Ethernet16   untagged
Vlan1000   1000  Ethernet20   untagged
Vlan1000   1000  Ethernet24   untagged
Vlan1000   1000  Ethernet28   untagged
Vlan1000   1000  Ethernet32   untagged
Vlan1000   1000  Ethernet36   untagged
Vlan1000   1000  Ethernet40   untagged
Vlan1000   1000  Ethernet44   untagged
Vlan1000   1000  Ethernet48   untagged
Vlan1000   1000  Ethernet52   untagged
Vlan1000   1000  Ethernet56   untagged
Vlan1000   1000  Ethernet60   untagged
Vlan1000   1000  Ethernet64   untagged
Vlan1000   1000  Ethernet68   untagged
Vlan1000   1000  Ethernet72   untagged
Vlan1000   1000  Ethernet76   untagged
Vlan1000   1000  Ethernet80   untagged
Vlan1000   1000  Ethernet84   untagged
Vlan1000   1000  Ethernet88   untagged
Vlan1000   1000  Ethernet92   untagged
Vlan1000   1000  Ethernet96   untagged
Vlan1000   1000  Ethernet100  untagged
Vlan1000   1000  Ethernet104  untagged
Vlan1000   1000  Ethernet108  untagged
Vlan1000   1000  Ethernet112  untagged
Vlan1000   1000  Ethernet116  untagged
Vlan1000   1000  Ethernet120  untagged
Vlan1000   1000  Ethernet124  untagged
```

#### Set the HOSTNAME. For the uplink interface, set the MTU and FEC

```
admin@sonic:~$ sudo -i
root@sonic:~# 
root@sonic:~# config hostname Celestica1
Please note loaded setting will be lost after system reboot. To preserve setting, run `config save`.
root@sonic:~# config interface mtu Ethernet124 9216
root@sonic:~# config interface fec Ethernet124 rs
root@sonic:~# show interface  status  Ethernet124
  Interface            Lanes    Speed    MTU    FEC    Alias    Vlan    Oper    Admin             Type    Asym PFC
-----------  ---------------  -------  -----  -----  -------  ------  ------  -------  ---------------  ----------
Ethernet124  125,126,127,128     100G   9216     rs    Eth32   trunk      up       up  QSFP28 or later         N/A
root@sonic:~# config save -y
Running command: /usr/local/bin/sonic-cfggen -d --print-data > /etc/sonic/config_db.json
root@sonic:~# logout
admin@sonic:~$ logout

$ ssh admin@sonic_ip
Linux Celestica1 6.1.0-11-2-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.38-4 (2023-08-08) x86_64
You are on
  ____   ___  _   _ _  ____
 / ___| / _ \| \ | (_)/ ___|
 \___ \| | | |  \| | | |
  ___) | |_| | |\  | | |___
 |____/ \___/|_| \_|_|\____|

-- Software for Open Networking in the Cloud --

Unauthorized access and/or use are prohibited.
All access and/or use are subject to monitoring.

Help:    https://sonic-net.github.io/SONiC/

Last login: Wed Jun 19 23:23:35 2024 from .......
admin@Celestica1:~$
```

#### Set statcic management sonic\_ip

```
admin@Celestica1:~$ show management_interface address
admin@Celestica1:~$ sudo config interface ip add eth0 192.168.10.101/24 192.168.10.1
admin@Celestica1:~$ show management_interface address
Management IP address = 192.168.10.101/24
Management Network Default Gateway = 192.168.10.1
```

#### Create management VLAN 100

```
admin@Celestica1:~$ show vlan brief
+-----------+--------------+-------------+----------------+-------------+-----------------------+
|   VLAN ID | IP Address   | Ports       | Port Tagging   | Proxy ARP   | DHCP Helper Address   |
+===========+==============+=============+================+=============+=======================+
|      1000 |              | Ethernet0   | untagged       | disabled    |                       |
|           |              | Ethernet4   | untagged       |             |                       |
...
|           |              | Ethernet124 | untagged       |             |                       |
+-----------+--------------+-------------+----------------+-------------+-----------------------+
admin@Celestica1:~$ sudo config vlan add 100
admin@Celestica1:~$ show vlan brief
+-----------+--------------+-------------+----------------+-------------+-----------------------+
|   VLAN ID | IP Address   | Ports       | Port Tagging   | Proxy ARP   | DHCP Helper Address   |
+===========+==============+=============+================+=============+=======================+
|       100 |              |             |                | disabled    |                       |
+-----------+--------------+-------------+----------------+-------------+-----------------------+
|      1000 |              | Ethernet0   | untagged       | disabled    |                       |
|           |              | Ethernet4   | untagged       |             |                       |
...
```

#### Assign IP for management VLAN 100

```
admin@Celestica1:~$ sudo -i
root@Celestica1:~# show ip interface
Interface    Master    IPv4 address/mask    Admin/Oper    BGP Neighbor    Neighbor IP
-----------  --------  -------------------  ------------  --------------  -------------
docker0                240.127.1.1/24       up/down       N/A             N/A
eth0                   192.168.10.101/24    up/up         N/A             N/A
lo                     127.0.0.1/16         up/up         N/A             N/A
root@Celestica1-DC:~# config interface ip add Vlan100 10.254.100.1/30
root@Celestica1-DC:~# show ip interface
Interface    Master    IPv4 address/mask    Admin/Oper    BGP Neighbor    Neighbor IP
-----------  --------  -------------------  ------------  --------------  -------------
Vlan100                10.254.100.1/30      up/up         N/A             N/A
docker0                240.127.1.1/24       up/down       N/A             N/A
eth0                   192.168.10.101/24    up/up         N/A             N/A
lo                     127.0.0.1/16         up/up         N/A             N/A
root@Celestica1C:~# show vlan brief
+-----------+-----------------+-------------+----------------+-------------+-----------------------+
|   VLAN ID | IP Address      | Ports       | Port Tagging   | Proxy ARP   | DHCP Helper Address   |
+===========+=================+=============+================+=============+=======================+
|       100 | 10.254.100.1/30 |             |                | disabled    |                       |
+-----------+-----------------+-------------+----------------+-------------+-----------------------+
|      1000 |                 | Ethernet0   | untagged       | disabled    |                       |
|           |                 | Ethernet4   | untagged       |             |                       |
...
```

#### Add member port for management VLAN 100

```
root@Celestica1:~# config vlan member add 100 Ethernet124
root@Celestica1:~# show vlan brief
+-----------+-----------------+-------------+----------------+-------------+-----------------------+
|   VLAN ID | IP Address      | Ports       | Port Tagging   | Proxy ARP   | DHCP Helper Address   |
+===========+=================+=============+================+=============+=======================+
|       100 | 10.254.100.1/30 | Ethernet124 | tagged         | disabled    |                       |
+-----------+-----------------+-------------+----------------+-------------+-----------------------+
|      1000 |                 | Ethernet0   | untagged       | disabled    |                       |
|           |                 | Ethernet4   | untagged       |             |                       |
...
```

#### Save changes form live Redis-DB to static config file /etc/sonic/config\_db.json

```
root@Celestica1-DC:~# config save -y
Running command: /usr/local/bin/sonic-cfggen -d --print-data > /etc/sonic/config_db.json
```

#### Connect from "Celestica1" with other switch "Celestica2" via VLAN 100 to interface IP.

```
root@Celestica1:~# ping 10.254.100.2
PING 10.254.100.2 (10.254.100.2) 56(84) bytes of data.
64 bytes from 10.254.100.2: icmp_seq=1 ttl=64 time=0.237 ms
64 bytes from 10.254.100.2: icmp_seq=2 ttl=64 time=0.270 ms

root@Celestica1-DC:~# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route, q - queued route, r - rejected route

C>*10.254.100.0/30 is directly connected, Vlan100, 00:16:25
C>*192.168.10.101/24 is directly connected, eth0, 00:30:58


root@Celestica1:~# ssh admin@10.254.100.2
Debian GNU/Linux 12 \n \l

admin@10.254.100.2's password:
Linux celestica2 6.1.0-11-2-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.38-4 (2023-08-08) x86_64
You are on
  ____   ___  _   _ _  ____
 / ___| / _ \| \ | (_)/ ___|
 \___ \| | | |  \| | | |
  ___) | |_| | |\  | | |___
 |____/ \___/|_| \_|_|\____|

-- Software for Open Networking in the Cloud --

Unauthorized access and/or use are prohibited.
All access and/or use are subject to monitoring.

Help:    https://sonic-net.github.io/SONiC/

Last login: Wed Jun 19 23:54:58 2024 from 10.254.100.1
```

#### Connect from "Celestica2" with other switch "Celestica1" via VLAN 100 to interface IP.

```
root@Celestica1:~# ping 10.254.100.1
PING 10.254.100.1 (10.254.100.1) 56(84) bytes of data.
64 bytes from 10.254.100.1: icmp_seq=1 ttl=64 time=0.231 ms
64 bytes from 10.254.100.1: icmp_seq=2 ttl=64 time=0.273 ms

root@Celestica1:~# ssh admin@10.254.100.1
Debian GNU/Linux 12 \n \l

admin@10.254.100.1's password:
Linux celestica1 6.1.0-11-2-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.38-4 (2023-08-08) x86_64
You are on
  ____   ___  _   _ _  ____
 / ___| / _ \| \ | (_)/ ___|
 \___ \| | | |  \| | | |
  ___) | |_| | |\  | | |___
 |____/ \___/|_| \_|_|\____|

-- Software for Open Networking in the Cloud --

Unauthorized access and/or use are prohibited.
All access and/or use are subject to monitoring.

Help:    https://sonic-net.github.io/SONiC/

Last login: Wed Jun 19 23:58:52 2024 from 10.254.100.2
```

#### Simple ACL restriction for SSH managemnt.

Now we plan to use an ssh connection from "Celestica1" to "Celestica2", but
not in the opposite direction. Therefore, you need to restrict the ssh service
in "Celestica1" to connect only from the management IP. Creating ACL rules
does not work from CLI commands, only the .json configuration file is used.

#### Create ACL rules in the acl\_ssh.json file on "Celestica1".

Note. vi commands : i - Insert mode, esc : wq - write quit

```
admin@Celestica1:~# vim acl-ssh.json
{
    "ACL_TABLE": {
	"SSH_ONLY": {
          "services": [
            "SSH"
          ],
          "type": "CTRLPLANE",
          "policy_desc": "SSH_ONLY"
        }
    },
    "ACL_RULE": {
        "SSH_ONLY|MANAGMENT_NET": {
            "PRIORITY": "999",
            "PACKET_ACTION": "ACCEPT",
            "SRC_IP": "192.168.10.0/24",
            "IP_PROTOCOL": "6"
        },
        "SSH_ONLY|DEFAULT_DENY": {
            "PRIORITY": "1",
            "PACKET_ACTION": "DROP",
            "ETHER_TYPE": "2048"
        }
     }
}

admin@Celestica1:~$ sudo config load ./acl-ssh.json -y
Running command: /usr/local/bin/sonic-cfggen -j ./acl-ssh.json --write-to-db

admin@Celestica1:~$  show runningconfiguration acl
{
    "SSH_ONLY|DEFAULT_DENY": {
        "ETHER_TYPE": "2048",
        "PACKET_ACTION": "DROP",
        "PRIORITY": "1"
    },
    "SSH_ONLY|MANAGMENT_NET": {
        "IP_PROTOCOL": "6",
        "PACKET_ACTION": "ACCEPT",
        "PRIORITY": "999",
        "SRC_IP": "192.168.10.0/24"
    }
}

admin@Celestica1:~$ show acl table   # (acl-loader show table)
Name      Type       Binding    Description    Stage    Status
--------  ---------  ---------  -------------  -------  --------
SSH_ONLY  CTRLPLANE  SSH        SSH_ONLY       ingress  Active

admin@Celestica1:~$ show acl rule    # (acl-loader show rule)
Table     Rule           Priority    Action    Match                  Status
--------  -------------  ----------  --------  ---------------------  --------
SSH_ONLY  MANAGMENT_NET  999         ACCEPT    IP_PROTOCOL: 6         N/A
                                               SRC_IP: 192.168.10.0/24
SSH_ONLY  DEFAULT_DENY   1           DROP      ETHER_TYPE: 2048       N/A

admin@Celestica1:~$ aclshow -a
RULE NAME      TABLE NAME      PRIO  PACKETS COUNT    BYTES COUNT
-------------  ------------  ------  ---------------  -------------
MANAGMENT_NET  SSH_ONLY         999  N/A              N/A
DEFAULT_DENY   SSH_ONLY           1  N/A              N/A

admin@Celestica1:~$ sudo config save -y
Running command: /usr/local/bin/sonic-cfggen -d --print-data > /etc/sonic/config_db.json
```

#### Test to establish an ssh connection from "Celestica2" to "Celestica1"

```
admin@celestica2:~$ ping 10.254.100.1
PING 10.254.100.1 (10.254.100.1) 56(84) bytes of data.
64 bytes from 10.254.100.1: icmp_seq=1 ttl=64 time=0.270 ms
64 bytes from 10.254.100.1: icmp_seq=2 ttl=64 time=0.821 ms
^C
--- 10.254.100.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1027ms
rtt min/avg/max/mdev = 0.270/0.545/0.821/0.275 ms
admin@celestica2:~$ ssh admin@10.254.100.1
ssh: connect to host 10.254.100.1 port 22: Connection timed out
admin@celestica2:~$
```

#### Add descriptipon for interlace Ethernet124

```
admin@Celestica1:~$ sudo vi /etc/sonic/config_db.json

     "Ethernet124": {
            "admin_status": "up",
            "alias": "Eth32",
            "fec": "rs",
            "index": "32",
            "lanes": "125,126,127,128",
            "mtu": "9216",
            "speed": "100000",
            "subport": "0",
            "description": "Uplink to Celestica2 Ethernet0[65..68]"
        }

admin@Celestica1:~$ sudo config load -y
admin@Celestica1:~$ show runningconfiguration ports
...
    "Ethernet124": {
        "admin_status": "up",
        "alias": "Eth32",
        "description": "Uplink to Celestica2 Ethernet0[65..68]",
        "fec": "rs",
        "index": "32",
        "lanes": "125,126,127,128",
        "mtu": "9216",
        "speed": "100000",
        "subport": "0"
    },
...


admin@Celestica1:~$ show interface description
  Interface    Oper    Admin    Alias                                   Description
-----------  ------  -------  -------  --------------------------------------------
  Ethernet0    down       up     Eth1
  Ethernet4    down       up     Eth2
...
Ethernet124      up       up    Eth32  Uplink to Celestica2 Ethernet0[65..68]
```

Or via sonic-cli

```
admin@Celestica1:~$ sonic-cli
Celestica1# configure terminal
Celestica1(config)# interface Ethernet 124
Celestica1(conf-if-Ethernet124)#  description "Uplink_to_Celestica2_Ethernet0[65..68]"
Celestica1(conf-if-Ethernet124)# end
Celestica1# show interface status | grep 124
Ethernet124   Uplink_to_Celestica2_Ethernet0[65..68]up      100GB          9216
Celestica1# exit
admin@Celestica1:~$ sudo config save -y
```

---
layout: post
title: "Hardware switch with stateless firewall rules. Seastone DX010. SONiC OS."
date: 2024-06-16 15:26:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/06/hardware-switch-with-stateless-firewall.html
---

⌛ Нові часи швидкостей 📶 і нове обладнання приходиться опановувати.

Так опановую налаштування 100Gb обладнання 🔨 на прикладі "[Celestica](https://www.celestica.com/)
[Seastone DX010](https://www.servethehome.com/inside-a-celestica-seastone-dx010-32x-100gbe-switch/), 32-Port 100G QSFP28
[Onie](https://community.fs.com/article/what-is-onie-open-network-install-environment.html)
Switch" з операційною системою
[SONiC OS](https://sonicfoundation.dev/).

🚆 Великі швидкості передачі - великі вимоги до часу ⛳ обробки мережевих
пакетів. Для пришвидшення обробки використовуються спеціальні апаратні
акселератори ([ASIC](https://uk.wikipedia.org/wiki/ASIC)), наприклад, 
[Broadcom Tomahawk ASIC](https://www.broadcom.com/info/switching/merchant-silicon).

🚀 ASIC своєю чергою можуть мати свої обмеження 🚧, такі, наприклад як
stateless firewall правила в
[ACL](https://uk.wikipedia.org/wiki/Access_control_list)

|  |
| --- |
|  |
| stateless firewall |

|  |
| --- |
|  |
| statefull firewall |

Чи так і є що більшість апаратних роутерів мають stateless firewall правила ?

🔌 І чи розумієте що треба контролювати самому і правила відповіді ⬅.

Наприклад, далі описано одностопній ➡ [SSH](https://uk.wikipedia.org/wiki/SSH)
доступ 🔥.

---

⌛ New times of speeds 📶 and new equipment have to be mastered.  
  
This
is how I master the configuration of 100Gb equipment 🔨 on the example of the
"[Celestica](https://www.celestica.com/) [SeastoneDX010](https://www.servethehome.com/inside-a-celestica-seastone-dx010-32x-100gbe-switch/), 32-Port 100G QSFP28 Onie Switch" with the
[SONiC OS](https://sonicfoundation.dev/) operating
system.  
  
🚆 High transmission speeds mean high time requirements for
network packet processing. To speed up processing, special hardware accelerators
(ASICs) are used.  
  
ASICs, in turn, may have their limitations, such as
stateless firewall rules in ACLs.  
  
Is it true that most hardware
routers have stateless firewall rules?  
  
🔌 And do you realize that you
need to control the response rules yourself ⬅.  
  
For example, the
following describes one-way ➡ SSH access 🔥.  
  

#### Create 'acl\_ssh.json' file for example

```
vim acl_ssh.json
```

```
{
  "ACL_TABLE": {
  	"SSH_ONLY": {
       "services": [
           "SSH"
        ],
       "type": "CTRLPLANE"
       "policy_desc": "SSH_ONLY"
    },
    "VLAN100_ACL": {
      "policy_desc": "VLAN100 Ingress ACL",
      "type": "L3",
      "stage": "INGRESS",
      "ports": [
        "Vlan100"
      ]
    }
  },
  "ACL_RULE": {
  	"SSH_ONLY|DEFAULT_RULE": {
        "PRIORITY": "1",
        "PACKET_ACTION": "DROP",
        "ETHER_TYPE": "2048"
    },
    "SSH_ONLY|RULE_1": {
        "PRIORITY": "99",
        "PACKET_ACTION": "ACCEPT",
        "SRC_IP": "192.168.0.1/32",
        "IP_PROTOCOL": "6"
    },
	"SSH_ONLY|RULE_2": {
         "PRIORITY": "98",
         "PACKET_ACTION": "ACCEPT",
         "SRC_IP": "192.168.1.1/32",
         "IP_PROTOCOL": "6"
    },
    "VLAN100_ACL|PERMIT_SSH_REQ": {
      "priority": "20",
      "PACKET_ACTION": "ACCEPT",
      "SRC_IP": "10.0.0.1/32",
      "DST_IP": "10.0.0.2/32",
      "IP_PROTOCOL": "6",
      "L4_DST_PORT": "22"
    },
    "VLAN100_ACL|PERMIT_SSH_ANSWER": {
      "priority": "10",
      "PACKET_ACTION": "ACCEPT",
      "SRC_IP": "10.0.0.2/32",
      "DST_IP": "10.0.0.1/32",
      "IP_PROTOCOL": "6",
      "L4_SRC_PORT": "22",
      "L4_DST_PORT": "1024-65535",
      "TCP_FLAGS": "0x10/0x10"
    },
    "VLAN100_ACL|DENY_ALL": {
      "priority": "2",
      "PACKET_ACTION": "DROP"
    }
  }
}
```

These rules allow:

* SSH initiation from switch 1 to switch 2 (traffic to TCP port 22).
* Reverse traffic from switch 2 to switch 1 for established TCP connections if
  the `ACK` checkbox is selected.
* Default Denial.

#### Add ACL config to global config

```
sudo config load -y acl_ssh.json
```

#### Show tables:

```
show acl table
Name         Type       Binding    Description          Stage    Status
-----------  ------     ---------  -------------------  -------  --------
SSH_ONLY     CTRLPLANE  SSH        SSH_ONLY             ingress  Active
VLAN100_ACL  L3         Vlan100    VLAN100 Ingress ACL  ingress  Active
```

#### Show rules:

```
show acl rule
Table        Rule               Priority    Action    Match                    Status
-----------  -----------------  ----------  --------  -----------------------  --------
VLAN100_ACL  DENY_ALL           99          DROP      N/A                      Inactive
VLAN100_ACL  PERMIT_SSH_ANSWER  20          ACCEPT    DST_IP: 10.0.0.1/32      Inactive
                                                      IP_PROTOCOL: 6
                                                      L4_DST_PORT: 1024-65535
                                                      L4_SRC_PORT: 22
                                                      SRC_IP: 10.0.0.2/32
                                                      TCP_FLAGS: 0x10/0x10
VLAN100_ACL  PERMIT_SSH_REQ     10          ACCEPT    DST_IP: 10.0.0.2/32      Active
                                                      IP_PROTOCOL: 6
                                                      L4_DST_PORT: 22
                                                      SRC_IP: 10.0.0.1/32
SSH_ONLY     RULE_1             99          ACCEPT    IP_PROTOCOL: 6           N/A
                                                      SRC_IP: 192.168.0.1/24
SSH_ONLY     RULE_2             98          ACCEPT    IP_PROTOCOL: 6           N/A
                                                      SRC_IP: 192.168.1.1/24
SSH_ONLY     DEFAULT_RULE        1          DROP      ETHER_TYPE: 2048         N/A
```

#### This command is used to display: ACL rules, tables and their priority, ACL packets counters, and bytes counters

```
aclshow -a
RULE NAME          TABLE NAME      PRIO  PACKETS COUNT    BYTES COUNT
-----------------  ------------  ------  ---------------  -------------
RULE_1             SSH_ONLY          99  N/A              N/A
RULE_2             SSH_ONLY          98  N/A              N/A
DEFAULT_RULE       SSH_ONLY           1  N/A              N/A
PERMIT_SSH_REQ     VLAN100_ACL       20  N/A              N/A
PERMIT_SSH_ANSWER  VLAN100_ACL       10  N/A              N/A
DENY_ALL           VLAN100_ACL        2  N/A              N/A
```

За матеріалами:

* sonic-notebook:
  [https://lexxai.blogspot.com/2024/04/sonic-notebook.html](https://lexxai.github.io/2024/04/sonic-notebook.html)
* [Understanding Stateful vs Stateless Firewalls](https://www.whizlabs.com/labs/understanding-stateful-vs-stateless-firewalls/)

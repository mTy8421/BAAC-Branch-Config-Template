#  SWITCH 01
Switch IOS Global Configuration Mode
```
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
!
!
hostname  SW_dep_ratchaburi_04
!
vtp mode transparent
!
```
Basic Service and Feature should be disabled
```
memory reserve critical 4096
!
no service tcp-small-servers
no service udp-small-servers
service nagle
service tcp-keepalives-in
service tcp-keepalives-out
service password-encryption
lldp run
no ip domain lookup
ip http server
ip http secure-server
ip http secure-active-session-modules none
ip http active-session-modules none
no ip finger
no ip host-routing
no ip icmp redirect
ip domain name it.baac.or.th
!
errdisable recovery cause udld
errdisable recovery cause bpduguard
errdisable recovery cause security-violation
errdisable recovery cause pagp-flap
errdisable recovery cause dtp-flap
errdisable recovery cause link-flap
errdisable recovery cause sfp-config-mismatch
errdisable recovery cause gbic-invalid
!
errdisable recovery cause psecure-violation
errdisable recovery cause port-mode-failure
errdisable recovery cause dhcp-rate-limit
errdisable recovery cause pppoe-ia-rate-limit
errdisable recovery cause mac-limit
errdisable recovery cause storm-control
errdisable recovery cause inline-power
errdisable recovery cause arp-inspection
errdisable recovery cause loopback
errdisable recovery cause small-frame
errdisable recovery cause psp
```
Shutdown VLAN 1
```
interface vlan 1
no ip address
shutdown
!
!
!
!
!
```
Config Archiving
```
archive
 log config
  logging enable
  logging size 200
  hidekeys
  notify syslog
!
ip domain name it.baac.or.th
!
enable secret inservice
username inbaac privilege 15 secret install
!
clock timezone BKK 7
!
crypto key generate rsa modulus 2048
!
ip ssh time-out 60
ip ssh authentication-retries 3
!
ip ssh version 2
!
vlan 200
name FRONT_Back_OFFICE
!
vlan 300
name ATM
!
vlan 400
name PBX_VOICE
!
vlan 401
name PBX_DATA
!
vlan 600
name NON_IT
!
vlan 700
name PRINTER
!
```
Syslog Setting
```
!
ip route 0.0.0.0 0.0.0.0 10.96.123.126
!
logging host 172.29.20.23
logging host 172.26.160.28
logging host 172.29.20.21
logging buffered
logging trap 6
logging buffered 6
no logging console
no logging monitor
logging buffered 100000
logging source-interface vlan 200
!
```
SNMP Community
```
!
snmp-server community baacwan RO snmp_server
snmp-server community baacbkk RW snmp_server
snmp-server group network-admin v3 auth
snmp-server user oper network-admin v3
snmp-server user oper enforcePriv v3
snmp-server group network-admin v3 priv context vlan-200
snmp-server user oper network-admin v3 auth md5 BAAC@noc priv aes 128 BAAC@noc
!
```
Banner 
```
!
banner login ^C
Warning!

*******************************************************************************
*  *
*  ***  BAAC ONLY!! ***  *
*  *
*  *
*  *
*  This system is for the use of authorized users only.  *
*  Individuals using this computer system without authority, or in  *
*  excess of their authority, are subject to having all of their  *
*  activities on this system monitored and recorded by system  *
*  personnel.  *
*  *
*  In the course of monitoring individuals improperly using this  *
*  system, or in the course of system maintenance, the activities  *
*  of authorized users may also be monitored.  *
*  *
*  Anyone using this system expressly consents to such monitoring  *
*  and is advised that if such monitoring reveals possible  *
*  evidence of criminal activity, system personnel may provide the  *
*  evidence of such monitoring to law enforcement officials.  *
*  *
*******************************************************************************
You have entered *** $(hostname) *** on line $(line)
^C
```
NTP 
```
ntp server 172.19.1.7
ntp server 172.19.1.8
!
!
```
Virtual Terminal Console
```
line con 0
 exec-timeout 10
!
line vty 0 15
 exec-timeout 10
 transport input ssh
login local
!
```
Access-list
```
!
ip access-list extended ISE-CWA-REDIRECT-BAAC
 deny udp any any eq domain
 deny ip any host 172.26.168.3
 deny ip any host 172.26.168.4
 deny ip any host 172.26.168.8
 deny ip any host 172.26.168.9
 permit tcp any any eq www
 permit tcp any any eq 443
 permit tcp any any eq 8443
!
```
Interface Configuration Mode (Access Host)
```
!
int vlan 200
ip address 10.96.123.4 255.255.255.128
no shut
!
interface range GigabitEthernet 1/0/3-41
description ## Connected to Front&Back Client ##
!
Switchport mode access
switchport access  vlan 200
 no cdp enable
authentication event fail action next-method
authentication event server dead action authorize vlan 200
authentication host-mode multi-auth
 authentication order mab dot1x
 authentication priority dot1x mab
 authentication port-control auto
 authentication periodic
 authentication timer reauthenticate server
 authentication timer restart 21
 authentication timer inactivity server
 authentication violation restrict
 mab
 dot1x pae authenticator
 dot1x timeout tx-period 10
 dot1x max-req 1
no shut
!
!
interface range GigabitEthernet 1/0/42-46
description ## Connected to Printer ##
!
switchport access vlan 700
switchport mode access
no cdp enable
no shut
!
interface range GigabitEthernet 1/0/47-48
description ## Connected to PBX-VOICE ##
switchport access vlan 400
 switchport mode access
 switchport voice vlan 400
 authentication event fail action next-method
authentication event server dead action authorize vlan 400
 authentication event server dead action authorize voice
 authentication event server alive action reinitialize
 authentication host-mode multi-auth
 authentication order mab
 authentication priority mab
 authentication port-control auto
 authentication periodic
 authentication timer reauthenticate 3600
 authentication timer restart 21
 authentication timer inactivity server
 authentication violation restrict
 mab
 dot1x pae authenticator
 dot1x timeout tx-period 10
 dot1x max-req 1
no shut
!
ip default-gateway 10.96.123.126
!
```
Create Port channel
```
interface po10
description ## Connected to Switch_1 ##
Switchport mode trunk
switchport trunk all vlan 200,300,400,401,600,700,998
no cdp enable
no shut
spanning-tree portfast trunk
```
Conect to Sw1
```
interface range Gi1/0/1-2
description ## Connected to Switch_1 ##
Switchport mode trunk
switchport trunk all vlan 200,300,400,401,600,700,998
channel-group 10 mode active
 no cdp enable
no shut
spanning-tree portfast trunk
!
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree extend system-id
!
!
```
ISE&AAA
```
!
aaa new-model
!
radius server bkn-baac-psn01
 address ipv4 172.26.168.3 auth-port 1812 acct-port 1813
 key BAAC&AIT
radius server bkn-baac-psn02
 address ipv4 172.26.168.4 auth-port 1812 acct-port 1813
 key BAAC&AIT
radius server sri-baac-psn03
 address ipv4 172.26.168.8 auth-port 1812 acct-port 1813
 key BAAC&AIT
radius server sri-baac-psn04
 address ipv4 172.26.168.9 auth-port 1812 acct-port 1813
 key BAAC&AIT
!
aaa group server radius NAC
 server name bkn-baac-psn01
 server name bkn-baac-psn02
 server name sri-baac-psn03
 server name sri-baac-psn04
!
aaa authentication dot1x default group NAC
aaa authorization network default group NAC
aaa authorization auth-proxy default group NAC
aaa accounting dot1x default start-stop group NAC
aaa accounting system default start-stop group NAC
aaa accounting network default start-stop group NAC
aaa accounting update newinfo periodic 2880
!
aaa session-id common
authentication mac-move permit
dot1x system-auth-control
!
ip device tracking
device-sensor accounting
mac address-table notification change
mac address-table notification mac-move
!
ip radius source-interface Vlan 200
radius-server deadtime 5
radius-server dead-criteria time 30 tries 3
radius-server vsa send accounting
radius-server vsa send authentication
radius-server attribute 6 on-for-login-auth
radius-server attribute 8 include-in-access-req
radius-server attribute 25 access-request include
radius-server attribute 31 mac format ietf upper-case
radius-server attribute 31 send nas-port-detail
!
aaa server radius dynamic-author
 client 172.26.168.3 server-key BAAC&AIT
 client 172.26.168.4 server-key BAAC&AIT
 client 172.26.168.8 server-key BAAC&AIT
 client 172.26.168.9 server-key BAAC&AIT
 auth-type any
!
snmp-server trap-source Vlan200
!
```
TACACS+
```
aaa new-model
!
!
aaa authentication login default local
aaa authentication login default group tacacs+ local
aaa authentication login tacvty group tacacs+ local
aaa authorization config-commands
aaa authorization exec default group tacacs+ local
aaa authorization exec tacvty group tacacs+ local
aaa authorization commands 15 default group tacacs+ local
aaa authorization commands 15 tacvty group tacacs+ local
aaa accounting exec tacvty
 action-type start-stop
 group tacacs+
!
aaa accounting commands 0 tacvty
 action-type start-stop
 group tacacs+
!
aaa accounting commands 1 tacvty
 action-type start-stop
 group tacacs+
!
aaa accounting commands 15 default
 action-type start-stop
 group tacacs+
!
aaa accounting commands 15 tacvty
 action-type start-stop
 group tacacs+
!
line con 0
 password inservice
line vty 0 4
 password cisco
 authorization commands 15 tacvty
 authorization exec tacvty
 accounting commands 0 tacvty
 accounting commands 1 tacvty
 accounting commands 15 tacvty
 accounting exec tacvty
 login authentication tacvty
 transport preferred ssh
 transport input ssh
line vty 5 15
 password cisco
 authorization commands 15 tacvty
 authorization exec tacvty
 accounting commands 0 tacvty
 accounting commands 1 tacvty
 accounting commands 15 tacvty
 accounting exec tacvty
 login authentication tacvty
 transport preferred ssh
 transport input ssh
!
tacacs-server host 172.26.168.5 key ciscobaac
tacacs-server host 172.26.168.10 key ciscobaac
tacacs-server timeout 1
!
service password-encryption
```
Shut and no shut interface range
```
interface range GigabitEthernet 1/0/1-48
 shut
 no shut
!
exit
exit
wr
!
```

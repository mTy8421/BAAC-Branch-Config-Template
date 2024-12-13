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

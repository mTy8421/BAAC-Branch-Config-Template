# 🏢 BAAC Branch Switch Configuration Template

[![Cisco](https://img.shields.io/badge/Cisco-Catalyst%20Switch-1BA0D7?logo=cisco&logoColor=white)](#)
[![Network](https://img.shields.io/badge/Architecture-Branch%20Access%20Switch-green)](#)
[![Security](https://img.shields.io/badge/Security-802.1X%20%7C%20MAB%20%7C%20Cisco%20ISE-red)](#)
[![AAA](https://img.shields.io/badge/AAA-TACACS%2B%20%7C%20RADIUS-orange)](#)

เอกสารแม่แบบการตั้งค่าสวิตช์เครือข่ายสำหรับสาขา **ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร (ธ.ก.ส. / BAAC)**  
ออกแบบสำหรับ **Cisco Catalyst Switch 48-Port** เพื่อใช้งานเป็น Access Switch ประจำสาขา รองรับการยืนยันตัวตนผ่าน Cisco ISE (802.1X / MAB), การจัดการอุปกรณ์รวมศูนย์ผ่าน TACACS+, Syslog, SNMP และการรักษาความปลอดภัยตามมาตรฐานความปลอดภัยไอทีของธนาคาร

---

## 📑 สารบัญ (Table of Contents)

1. [ข้อมูลภาพรวมระบบและผังพอร์ต (System Overview & Port Mapping)](#1-ข้อมูลภาพรวมระบบและผังพอร์ต-system-overview--port-mapping)
2. [ตารางตัวแปรที่ต้องแก้ไขเฉพาะสาขา (Site Parameter Checklist)](#2-ตารางตัวแปรที่ต้องแก้ไขเฉพาะสาขา-site-parameter-checklist)
3. [ขั้นตอนการตั้งค่าแยกตามหมวดหมู่ (Step-by-Step Configuration)](#3-ขั้นตอนการตั้งค่าแยกตามหมวดหมู่-step-by-step-configuration)
   - [3.1 Global & System Settings](#31-global--system-settings)
   - [3.2 Errdisable Recovery](#32-errdisable-recovery)
   - [3.3 VLAN & L2 Spanning-Tree](#33-vlan--l2-spanning-tree)
   - [3.4 Security, Admin & Management Access](#34-security-admin--management-access)
   - [3.5 Network Services (Syslog, NTP, Default Route)](#35-network-services-syslog-ntp-default-route)
   - [3.6 SNMP Monitoring](#36-snmp-monitoring)
   - [3.7 Console & VTY Line Access](#37-console--vty-line-access)
   - [3.8 Access-List (ISE CWA Redirect ACL)](#38-access-list-ise-cwa-redirect-acl)
   - [3.9 Interface Configuration (Access, Voice, Printer, Uplink)](#39-interface-configuration-access-voice-printer-uplink)
   - [3.10 Cisco ISE & 802.1X / MAB / CoA](#310-cisco-ise--8021x--mab--coa)
   - [3.11 TACACS+ Authentication & Authorization](#311-tacacs-authentication--authorization)
   - [3.12 Restart Ports & Save Configuration](#312-restart-ports--save-configuration)
4. [สคริปต์การตั้งค่าฉบับเต็ม (Complete One-Click Script)](#4-สคริปต์การตั้งค่าฉบับเต็ม-complete-one-click-script)
5. [คำสั่งตรวจสอบการทำงาน (Verification & Troubleshooting)](#5-คำสั่งตรวจสอบการทำงาน-verification--troubleshooting)
6. [ข้อควรระวังสำคัญ (Important Considerations)](#6-ข้อควรระวังสำคัญ-important-considerations)

---

## 1. ข้อมูลภาพรวมระบบและผังพอร์ต (System Overview & Port Mapping)

### 📌 ตารางข้อมูล VLAN (VLAN Mapping)

| VLAN ID | VLAN Name | วัตถุประสงค์การใช้งาน | หมายเหตุ |
| :---: | :--- | :--- | :--- |
| **1** | Default | ไม่ใช้งาน (Default VLAN) | ปิดใช้งาน (`shutdown` & `no ip address`) |
| **200** | `FRONT_Back_OFFICE` | เครื่องคอมพิวเตอร์พนักงานหน้าเคาน์เตอร์ และหลังเคาน์เตอร์ / SVI Mgmt | รองรับ 802.1X / MAB ผ่าน Cisco ISE |
| **300** | `ATM` | เครื่องบริการอัตโนมัติ / ตู้ ATM | Trunk ไปยัง Switch หลัก |
| **400** | `PBX_VOICE` | โทรศัพท์ไอพี (IP Phone / VoIP) | รองรับ MAB Authentication |
| **401** | `PBX_DATA` | อุปกรณ์ระบบโทรศัพท์ (Data) | Trunk ไปยัง Switch หลัก |
| **600** | `NON_IT` | อุปกรณ์ส่วนสนับสนุนอื่นๆ นอกระบบไอทีหลัก | Trunk ไปยัง Switch หลัก |
| **700** | `PRINTER` | เครื่องพิมพ์เครือข่าย (Network Printers) | Static Access Port |
| **998** | `Transit / Native` | ทราฟฟิกเฉพาะและระบบเชื่อมโยง | อยู่ใน Trunk Allowed VLAN |

---

### 🔌 ผังการจัดสรรพอร์ตสวิตช์ 48 พอร์ต (Port Allocation Table)

```
[Gi1/0/1 - Gi1/0/2]  ==> Uplink to SW1 (Port-Channel 10: Trunk)
[Gi1/0/3 - Gi1/0/41] ==> Front & Back Office Workstations (VLAN 200 + 802.1X / MAB)
[Gi1/0/42 - Gi1/0/46] ==> Network Printers (VLAN 700 Access)
[Gi1/0/47 - Gi1/0/48] ==> IP Phone / PBX Voice (VLAN 400 + MAB)
```

| พอร์ต | ประเภท | VLAN | การยืนยันตัวตน | รายละเอียด |
| :--- | :--- | :---: | :---: | :--- |
| **Gi1/0/1 - Gi1/0/2** | Trunk (Po10) | 200, 300, 400, 401, 600, 700, 998 | - | Uplink เชื่อมต่อ Switch 1 (LACP Active) |
| **Gi1/0/3 - Gi1/0/41** | Access | 200 | 802.1X + MAB (Multi-Auth) | สำหรับเครื่อง Client หน้างานและหลังงาน |
| **Gi1/0/42 - Gi1/0/46** | Access | 700 | None | เครื่องพิมพ์เครือข่าย (Printers) |
| **Gi1/0/47 - Gi1/0/48** | Access + Voice | 400 | MAB | อุปกรณ์โทรศัพท์ PBX / Voice |

---

## 2. ตารางตัวแปรที่ต้องแก้ไขเฉพาะสาขา (Site Parameter Checklist)

> [!IMPORTANT]
> ก่อนนำคอนฟิกไปใช้งาน **โปรดตรวจสอบและแทนที่ค่าตัวแปรตามข้อมูลสาขาที่ติดตั้งจริง**

| ตัวแปรในเทมเพลต | ค่าตัวอย่างในเทมเพลต | คำอธิบายและข้อกำหนด |
| :--- | :--- | :--- |
| `<HOSTNAME>` | `SW_dep_ratchaburi_04` | ตั้งชื่อตามมาตรฐานสาขา เช่น `SW_dep_<ชื่อสาขา>_<หมายเลข>` |
| `<VLAN200_IP>` | `10.96.123.4` | IP Address ของ SVI VLAN 200 ประจำสวิตช์สาขา |
| `<SUBNET_MASK>` | `255.255.255.128` | Subnet Mask ของ VLAN 200 ประจำสาขา |
| `<DEFAULT_GATEWAY>` | `10.96.123.126` | IP Default Gateway ของสาขา (สำหรับ IP Route และ Default Gateway) |
| `<ENABLE_SECRET>` | `inservice` | รหัสผ่าน Enable Secret ประจำอุปกรณ์ |
| `<ADMIN_USER>` | `inbaac` / `install` | Local Administrator Account สำรองกรณีฉุกเฉิน |

### 🌐 เซิร์ฟเวอร์ส่วนกลาง (ไม่ต้องเปลี่ยนแปลง)

- **Cisco ISE (RADIUS):** `172.26.168.3`, `172.26.168.4`, `172.26.168.8`, `172.26.168.9` (Key: `BAAC&AIT`)
- **TACACS+ Servers:** `172.26.168.5`, `172.26.168.10` (Key: `ciscobaac`)
- **Syslog Hosts:** `172.29.20.23`, `172.26.160.28`, `172.29.20.21`
- **NTP Servers:** `172.19.1.7`, `172.19.1.8`

---

## 3. ขั้นตอนการตั้งค่าแยกตามหมวดหมู่ (Step-by-Step Configuration)

### 3.1 Global & System Settings

กำหนดเวลาของระบบ, Hostname, โหมด VTP และปิดฟังก์ชันบริการที่ไม่ปลอดภัย

```cisco
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
!
hostname SW_dep_ratchaburi_04
!
vtp mode transparent
!
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
```

---

### 3.2 Errdisable Recovery

เปิดใช้งานการกู้คืนพอร์ตแบบอัตโนมัติ (Automatic Recovery) ในกรณีที่พอร์ตถูกสั่งปิดด้วยเหตุการณ์ต่างๆ

```cisco
errdisable recovery cause udld
errdisable recovery cause bpduguard
errdisable recovery cause security-violation
errdisable recovery cause pagp-flap
errdisable recovery cause dtp-flap
errdisable recovery cause link-flap
errdisable recovery cause sfp-config-mismatch
errdisable recovery cause gbic-invalid
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
!
```

---

### 3.3 VLAN & L2 Spanning-Tree

ปิดใช้งาน Default VLAN 1 เพื่อความปลอดภัย และสร้าง VLAN Database ตามโครงสร้างสาขา

```cisco
interface Vlan1
 no ip address
 shutdown
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
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree extend system-id
!
```

---

### 3.4 Security, Admin & Management Access

ตั้งค่าระบบเก็บบันทึกประวัติการเปลี่ยนแปลงคำสั่ง (Archive), บัญชีผู้ดูแลระบบฉุกเฉิน, เขตเวลา (Timezone), กุญแจเข้ารหัส RSA และ SSH Version 2

```cisco
archive
 log config
  logging enable
  logging size 200
  hidekeys
  notify syslog
!
clock timezone BKK 7
!
enable secret inservice
username inbaac privilege 15 secret install
!
crypto key generate rsa modulus 2048
!
ip ssh time-out 60
ip ssh authentication-retries 3
ip ssh version 2
!
```

---

### 3.5 Network Services (Syslog, NTP, Default Route)

ตั้งค่าเส้นทางออก (Default Route/Gateway), จุดส่ง Syslog และแม่ข่ายเวลา NTP

```cisco
ip route 0.0.0.0 0.0.0.0 10.96.123.126
!
logging host 172.29.20.23
logging host 172.26.160.28
logging host 172.29.20.21
logging trap 6
logging buffered 100000 6
no logging console
no logging monitor
logging source-interface Vlan200
!
ntp server 172.19.1.7
ntp server 172.19.1.8
!
```

---

### 3.6 SNMP Monitoring

กำหนดค่า SNMP v2c และ v3 สำหรับระบบติดตามและเฝ้าระวังเครือข่าย

```cisco
snmp-server community baacwan RO snmp_server
snmp-server community baacbkk RW snmp_server
snmp-server group network-admin v3 auth
snmp-server user oper network-admin v3
snmp-server user oper enforcePriv v3
snmp-server group network-admin v3 priv context vlan-200
snmp-server user oper network-admin v3 auth md5 BAAC@noc priv aes 128 BAAC@noc
snmp-server trap-source Vlan200
!
```

---

### 3.7 Console & VTY Line Access

กำหนดข้อความแจ้งเตือนความปลอดภัย (Banner) และความปลอดภัยของพอร์ต Console / VTY

```cisco
banner login ^C
Warning!

*******************************************************************************
*                                                                             *
*                             ***  BAAC ONLY!! ***                            *
*                                                                             *
*                                                                             *
*                                                                             *
*  This system is for the use of authorized users only.                       *
*  Individuals using this computer system without authority, or in            *
*  excess of their authority, are subject to having all of their              *
*  activities on this system monitored and recorded by system                 *
*  personnel.                                                                 *
*                                                                             *
*  In the course of monitoring individuals improperly using this              *
*  system, or in the course of system maintenance, the activities              *
*  of authorized users may also be monitored.                                 *
*                                                                             *
*  Anyone using this system expressly consents to such monitoring             *
*  and is advised that if such monitoring reveals possible                    *
*  evidence of criminal activity, system personnel may provide the            *
*  evidence of such monitoring to law enforcement officials.                 *
*                                                                             *
*******************************************************************************
You have entered *** $(hostname) *** on line $(line)
^C
!
line con 0
 exec-timeout 10 0
 password inservice
!
line vty 0 15
 exec-timeout 10 0
 transport preferred ssh
 transport input ssh
 login local
!
```

---

### 3.8 Access-List (ISE CWA Redirect ACL)

Access-List สำหรับส่งต่อผู้ใช้งานไปยังหน้าเว็บยืนยันตัวตน Cisco ISE Central Web Authentication (CWA)

```cisco
ip access-list extended ISE-CWA-REDIRECT-BAAC
 deny   udp any any eq domain
 deny   ip any host 172.26.168.3
 deny   ip any host 172.26.168.4
 deny   ip any host 172.26.168.8
 deny   ip any host 172.26.168.9
 permit tcp any any eq www
 permit tcp any any eq 443
 permit tcp any any eq 8443
!
```

---

### 3.9 Interface Configuration (Access, Voice, Printer, Uplink)

#### 1) SVI Management (VLAN 200)

```cisco
interface Vlan200
 ip address 10.96.123.4 255.255.255.128
 no shutdown
!
ip default-gateway 10.96.123.126
!
```

#### 2) Client Ports (Gi1/0/3 - 41) - 802.1X & MAB

```cisco
interface range GigabitEthernet1/0/3-41
 description ## Connected to Front&Back Client ##
 switchport mode access
 switchport access vlan 200
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
 no shutdown
!
```

#### 3) Printer Ports (Gi1/0/42 - 46)

```cisco
interface range GigabitEthernet1/0/42-46
 description ## Connected to Printer ##
 switchport access vlan 700
 switchport mode access
 no cdp enable
 no shutdown
!
```

#### 4) PBX / Voice Ports (Gi1/0/47 - 48) - MAB

```cisco
interface range GigabitEthernet1/0/47-48
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
 no shutdown
!
```

#### 5) Uplink to Switch 1 (Port-Channel 10)

```cisco
interface Port-channel10
 description ## Connected to Switch_1 ##
 switchport mode trunk
 switchport trunk allowed vlan 200,300,400,401,600,700,998
 no cdp enable
 spanning-tree portfast trunk
 no shutdown
!
interface range GigabitEthernet1/0/1-2
 description ## Connected to Switch_1 ##
 switchport mode trunk
 switchport trunk allowed vlan 200,300,400,401,600,700,998
 channel-group 10 mode active
 no cdp enable
 spanning-tree portfast trunk
 no shutdown
!
```

---

### 3.10 Cisco ISE & 802.1X / MAB / CoA

เชื่อมต่อไปยัง Cisco Identity Services Engine (ISE) สำหรับการยืนยันตัวตนระดับพอร์ต

```cisco
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
ip radius source-interface Vlan200
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
```

---

### 3.11 TACACS+ Authentication & Authorization

ระบบยืนยันตัวตนและบันทึกการใช้งานของผู้ดูแลระบบผ่าน TACACS+ Server

```cisco
aaa authentication login default group tacacs+ local
aaa authentication login tacvty group tacacs+ local
aaa authorization config-commands
aaa authorization exec default group tacacs+ local
aaa authorization exec tacvty group tacacs+ local
aaa authorization commands 15 default group tacacs+ local
aaa authorization commands 15 tacvty group tacacs+ local
!
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
!
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
!
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
!
```

---

### 3.12 Restart Ports & Save Configuration

รีสตาร์ทพอร์ตเชื่อมต่อเพื่อให้รับค่านโยบายใหม่ และบันทึกการตั้งค่าลง NVRAM

```cisco
interface range GigabitEthernet1/0/1-48
 shutdown
 no shutdown
!
end
write memory
```

---

## 4. สคริปต์การตั้งค่าฉบับเต็ม (Complete One-Click Script)

<details>
<summary><b>คลิกเพื่อขยายดูสคริปต์ฉบับเต็มสำหรับคัดลอก (Full Configuration Script)</b></summary>

```cisco
! ==============================================================================
! BAAC Branch Switch Configuration Template
! Switch: SW_dep_ratchaburi_04
! ==============================================================================

service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
!
hostname SW_dep_ratchaburi_04
!
vtp mode transparent
!
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
!
interface Vlan1
 no ip address
 shutdown
!
archive
 log config
  logging enable
  logging size 200
  hidekeys
  notify syslog
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
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree extend system-id
!
ip route 0.0.0.0 0.0.0.0 10.96.123.126
!
logging host 172.29.20.23
logging host 172.26.160.28
logging host 172.29.20.21
logging trap 6
logging buffered 100000 6
no logging console
no logging monitor
logging source-interface Vlan200
!
snmp-server community baacwan RO snmp_server
snmp-server community baacbkk RW snmp_server
snmp-server group network-admin v3 auth
snmp-server user oper network-admin v3
snmp-server user oper enforcePriv v3
snmp-server group network-admin v3 priv context vlan-200
snmp-server user oper network-admin v3 auth md5 BAAC@noc priv aes 128 BAAC@noc
snmp-server trap-source Vlan200
!
banner login ^C
Warning!

*******************************************************************************
*                                                                             *
*                             ***  BAAC ONLY!! ***                            *
*                                                                             *
*                                                                             *
*                                                                             *
*  This system is for the use of authorized users only.                       *
*  Individuals using this computer system without authority, or in            *
*  excess of their authority, are subject to having all of their              *
*  activities on this system monitored and recorded by system                 *
*  personnel.                                                                 *
*                                                                             *
*  In the course of monitoring individuals improperly using this              *
*  system, or in the course of system maintenance, the activities              *
*  of authorized users may also be monitored.                                 *
*                                                                             *
*  Anyone using this system expressly consents to such monitoring             *
*  and is advised that if such monitoring reveals possible                    *
*  evidence of criminal activity, system personnel may provide the            *
*  evidence of such monitoring to law enforcement officials.                 *
*                                                                             *
*******************************************************************************
You have entered *** $(hostname) *** on line $(line)
^C
!
ntp server 172.19.1.7
ntp server 172.19.1.8
!
ip access-list extended ISE-CWA-REDIRECT-BAAC
 deny   udp any any eq domain
 deny   ip any host 172.26.168.3
 deny   ip any host 172.26.168.4
 deny   ip any host 172.26.168.8
 deny   ip any host 172.26.168.9
 permit tcp any any eq www
 permit tcp any any eq 443
 permit tcp any any eq 8443
!
interface Vlan200
 ip address 10.96.123.4 255.255.255.128
 no shutdown
!
ip default-gateway 10.96.123.126
!
interface range GigabitEthernet1/0/3-41
 description ## Connected to Front&Back Client ##
 switchport mode access
 switchport access vlan 200
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
 no shutdown
!
interface range GigabitEthernet1/0/42-46
 description ## Connected to Printer ##
 switchport access vlan 700
 switchport mode access
 no cdp enable
 no shutdown
!
interface range GigabitEthernet1/0/47-48
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
 no shutdown
!
interface Port-channel10
 description ## Connected to Switch_1 ##
 switchport mode trunk
 switchport trunk allowed vlan 200,300,400,401,600,700,998
 no cdp enable
 spanning-tree portfast trunk
 no shutdown
!
interface range GigabitEthernet1/0/1-2
 description ## Connected to Switch_1 ##
 switchport mode trunk
 switchport trunk allowed vlan 200,300,400,401,600,700,998
 channel-group 10 mode active
 no cdp enable
 spanning-tree portfast trunk
 no shutdown
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
ip radius source-interface Vlan200
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
aaa authentication login default group tacacs+ local
aaa authentication login tacvty group tacacs+ local
aaa authorization config-commands
aaa authorization exec default group tacacs+ local
aaa authorization exec tacvty group tacacs+ local
aaa authorization commands 15 default group tacacs+ local
aaa authorization commands 15 tacvty group tacacs+ local
!
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
 exec-timeout 10 0
 password inservice
!
line vty 0 4
 exec-timeout 10 0
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
line vty 5 15
 exec-timeout 10 0
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
!
interface range GigabitEthernet1/0/1-48
 shutdown
 no shutdown
!
end
write memory
```

</details>

---

## 5. คำสั่งตรวจสอบการทำงาน (Verification & Troubleshooting)

หลังการตั้งค่าเสร็จสิ้น สามารถใช้คำสั่งเหล่านี้ในการทดสอบสถานะระบบ:

### 1) ตรวจสอบการเชื่อมต่อและพอร์ต
```cisco
show ip interface brief
show interfaces status
show etherchannel summary
show vlan brief
```

### 2) ตรวจสอบสถานะการยืนยันตัวตน 802.1X / MAB บน Cisco ISE
```cisco
show authentication sessions
show authentication sessions interface Gi1/0/3 details
show dot1x all summary
show dot1x statistics
```

### 3) ตรวจสอบ AAA, RADIUS และ TACACS+
```cisco
show aaa servers
show tacacs
show radius server-group NAC
```

### 4) ตรวจสอบเวลาและบันทึกระบบ
```cisco
show ntp status
show ntp associations
show logging
```

---

## 6. ข้อควรระวังสำคัญ (Important Considerations)

> [!WARNING]
> **การคอนฟิกจากระยะไกล (Remote Configuration via SSH/Telnet):**  
> ในส่วนขั้นตอนสุดท้าย `interface range GigabitEthernet1/0/1-48` มีคำสั่ง `shutdown` ซึ่งรวมถึงพอร์ต Uplink (`Gi1/0/1-2`) ด้วย  
> หากดำเนินการผ่าน Remote SSH จะทำให้การเชื่อมต่อหลุดทันที  
> **แนะนำ:** หากตั้งค่าจากระยะไกล ให้เว้นพอร์ต Uplink ออกไป เช่น รันเฉพาะ `interface range GigabitEthernet1/0/3-48` แทน

> [!NOTE]
> คำสั่ง `crypto key generate rsa modulus 2048` ในบางรุ่นอาจถามยืนยันการเขียนทับกุญแจเดิม (`Do you really want to replace them? [yes/no]:`) ให้ตอบ `yes`

# 🚀 High Availability Slurm Cluster using Pacemaker, Corosync & DRBD

## 📌 Project Overview
This project implements a production-grade High Availability HPC cluster using Slurm with automated failover, secure remote access, centralized authentication, web-based job management, and full monitoring/alerting.
This project implements a **Highly Available (HA) Slurm Workload Manager cluster** using:
- Highly available job scheduling
- Secure remote HPC access
- Centralized user authentication
- Web-based user interaction
- Automated failover
- Pacemaker (Cluster Resource Manager)
- Corosync (Cluster Communication Layer)
- DRBD (Block-Level Data Replication)
- Virtual IP Failover
- MariaDB for Slurm Accounting
- Shared Slurm State Storage
- Full observability and alerting

The objective is to provide **automatic failover of Slurm Controller services** with minimal downtime while ensuring data consistency across controller nodes.

---

## 🎯 Objectives

- Achieve High Availability for Slurm Controller
- Prevent Single Point of Failure
- Automatic Service Failover
- Replicated Slurm State Data
- Seamless Job Scheduling Continuity
- Fast Recovery during Node Failure

---

## 🧱 Architecture
Users
 ↓
WireGuard VPN
 ↓
OnDemand Web Portal
 ↓
Login Node
 ↓
Slurm VIP
 ↓
Active Controller (Pacemaker)
 ↓
DRBD Replicated Storage
 ↓
Compute Nodes

Monitoring Layer:

Prometheus ← exporters on all nodes
     ↓
Alertmanager
     ↓
Grafana Dashboards


---

### 🔗 Core Components

| Component | Purpose |
|-----------|---------|
| Corosync | Cluster Communication |
| Pacemaker | Resource Management |
| DRBD | Block Storage Replication |
| Virtual IP | Floating Controller Access |
| Slurmctld | Scheduler Controller |
| Slurmdbd | Accounting Database |
| MariaDB | Job Accounting Storage |

---
## ⚙️ Cluster Resource Flow

### Pacemaker controls:
- Dual Slurm Controllers
- DRBD replicated spool & database storage
- Pacemaker resource orchestration
- Automatic failover
- Virtual IP migration
### Failover Targets -
1. slurmctld
2. slurmdbd
3. MariaDB
4. DRBD storage
5. Filesystem mount
6. Virtual IP

---
## ⚙️ Configurations


#### Deployment Steps 
##### Infrastructure Setup

1. Install Linux on all nodes
2. Configure networking & DNS
3. Setup LDAP server
4. Configure WireGuard VPN

##### Cluster Setup
5. Install Slurm on all nodes
6. Setup DRBD replication
7. Configure Pacemaker + Corosync
8. Create Virtual IP resource
9. Configure Slurm HA services

##### Access Layer

10. Configure Login Node
11. Install Open OnDemand
12. Integrate LDAP authentication

##### Monitoring

13. Install Prometheus
14. Configure exporters
15. Setup Alertmanager
16. Deploy Grafana dashboards
---
### 🌐 1. DNS Server Setup

```bash
apt install bind9 -y
nano /etc/bind/named.conf.local
systemctl restart bind9
```

### 👥 2. LDAP Authentication
Centralized user authentication for HPC users.
##### LDAP provides:
- Centralized user accounts
- Shared UID/GID
- Secure authentication
- Consistent login across nodes
##### LDAP integrated with:

1. Login Node
2. Open OnDemand
```bash
apt install slapd ldap-utils -y
dpkg-reconfigure slapd
apt install libnss-ldap libpam-ldap nscd -y
```
Set:
1. Domain: hpccluster.com
2. Org: HPCCluster

##### Create Base LDAP Structure
```
ldapadd -x -D cn=admin,dc=hpccluster,dc=com -W -f base.ldif
```
Example:
```
ou=People
ou=Groups
ou=HPCUsers
```
##### Add LDAP Client to ALL Nodes

Install:
```
apt install libnss-ldap libpam-ldap ldap-utils nscd
```

Configure:
```
/etc/ldap/ldap.conf
```
```
BASE dc=hpccluster,dc=com
```
##### Enable LDAP Authentication
```
pam-auth-update
```
Enable:
1. LDAP Authentication
2. Create Home Directory
### 💾 3. DRBD Storage Replication
Replicated Slurm spool and database storage.
```bash
apt install drbd-utils -y
```

#### Filesystem Setup
```bash
mkfs.ext4 /dev/drbd0
mount /dev/drbd0 /var/spool/slurm
nano /etc/drbd.d/slurm_data.res
drbdadm create-md slurm_data
drbdadm up slurm_data
drbdadm primary --force slurm_data
```
#### 📊 DRBD Sync Monitoring
```
cat /proc/drbd
watch -n1 drbdadm status
```
Output:
```
Primary/Secondary
SyncSource / SyncTarget
UpToDate/Inconsistent
```

### 🧠 4. Pacemaker + Corosync Cluster
High availability cluster manager.
```bash
apt install pacemaker corosync pcs -y
systemctl enable pcsd --now
pcs host auth MasterNode PassiveMaster
pcs cluster setup SlurmHA MasterNode PassiveMaster
pcs cluster start --all
```


#### Pacemaker Resource Creation
```bash
pcs resource create slurm_data_res ocf:linbit:drbd drbd_resource=slurm_data
pcs resource create slurm_fs ocf:heartbeat:Filesystem device=/dev/drbd0 directory=/var/spool/slurm fstype=ext4
pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=<VIP> cidr_netmask=24
pcs resource create mariadb systemd:mariadb
pcs resource create slurmdbd_res systemd:slurmdbd
pcs resource create slurm_ctld_res systemd:slurmctld
```
```
Pacemaker
   |
   +-- DRBD (Primary/Secondary)
          |
          +-- /var/spool/slurm
                  |
                  +-- Slurmctld
                  +-- Slurmdbd
                  +-- MariaDB
                  +-- SlurmVIP
```
### 🔐 5. WireGuard VPN Setup
Secure communication between user & cluster nodes.
##### WireGuard provides:
- Encrypted remote HPC access
- Private cluster network exposure
- Secure admin access
- Reduced public attack surface
- Users connect via VPN before accessing:
- Login Node
- Web Portal
- Monitoring
  
```
apt install wireguard -y
wg genkey | tee privatekey | wg pubkey > publickey
nano /etc/wireguard/wg0.conf
```
##### On Controller:
```
[Interface]
Address = 10.10.10.1/24
PrivateKey = <key>
ListenPort = 51820

[Peer]
PublicKey = <clientkey>
AllowedIPs = 10.10.10.2/32
```

Start:
```
systemctl enable wg-quick@wg0 --now

```
Test:
```
wg
ping 10.10.10.2
```

### 🌐 6. Open OnDemand Web Portal
Web-based HPC job submission interface.
##### Features:
- Web-based shell
- Job submission interface
- Interactive apps
- File browser
- HPC dashboard
- Authentication handled via LDAP.
- Users access cluster through browser without SSH.

##### Start OOD

```
apt install ondemand -y
nano /etc/ood/config/clusters.d/hpc.yml
systemctl restart ondemand
/opt/ood/ood-portal-generator/sbin/update_ood_portal
```
##### Authentication Integration

```
/etc/ood/config/ood_portal.yml
```
```
auth:
  - 'AuthType Basic'
  - 'AuthName "HPC Portal"'
  - 'AuthBasicProvider ldap'
```

### 🚨 7. Alertmanager
Alerting system for failures and thresholds.

##### Triggers alerts on:

- Controller failure
- Node down
- DRBD sync failure
- Slurmctld crash
- Database failure
- High CPU/memory
- Service downtime
  
```bash
apt install prometheus-alertmanager -y
nano /etc/alertmanager/alertmanager.yml
systemctl restart prometheus-alertmanager
```
### 📊 8. Monitoring – Prometheus
Cluster metrics collection.
### Collects metrics from:

- Slurm services
- DRBD replication
- Pacemaker cluster
- System resources
- Network interfaces
- VPN

```
apt install prometheus -y
apt install prometheus-node-exporter -y
systemctl enable prometheus --now
```
### 📈 9. Grafana Dashboard
Visualization for cluster monitoring.
```
apt install grafana -y
systemctl enable grafana-server --now
```
Access
```
http://monitoring-node:3000
```
### Resource Group Example
DRBD
 → Filesystem
   → MariaDB
     → SlurmDBD
       → SlurmCTLD
         → VIP

### 🧪 10. Failover Testing
```
pcs node standby MasterNode
pcs status
scontrol ping
```
##### Failover Testing
- Move Resources
- pcs node standby PassiveMaster
- Check Cluster
- pcs status
- Slurm Health
- scontrol ping
- sinfo

### 🧰 11. Validation Commands
```
pcs status
drbdadm status
cat /proc/drbd
sinfo
squeue
sacct
```

---

## 🚨 Common Issues Faced


---

## 📈 Project Outcomes

Fully functional HA Slurm cluster

Successful automatic failover

Replicated scheduler state

Stable cluster communication

Production-grade HPC control plane

---

## 🧪 Validation

Controller Failover Successful

DRBD Replication Verified

Job Scheduling Survived Node Failure

VIP Migration Confirmed

Resource Ordering Enforced

---

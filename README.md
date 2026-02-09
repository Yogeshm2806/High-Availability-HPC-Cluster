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
## 3️⃣ Configurations

### 🌐 1. DNS Server Setup

```bash
apt install bind9 -y
nano /etc/bind/named.conf.local
systemctl restart bind9
```

### 👥 2. LDAP Authentication
```bash
apt install slapd ldap-utils -y
dpkg-reconfigure slapd
apt install libnss-ldap libpam-ldap nscd -y
```

### 💾 3. DRBD Storage Replication
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
```bash
apt install pacemaker corosync pcs -y
systemctl enable pcsd --now
pcs host auth MasterNode PassiveMaster
pcs cluster setup SlurmHA MasterNode PassiveMaster
pcs cluster start --all
```

###🗄️ 5. MariaDB Accounting Database

```bash
apt install mariadb-server -y
mysql_secure_installation
mysql -u root -p
```

#### 🔁 6. Pacemaker Resource Creation
```bash
pcs resource create slurm_data_res ocf:linbit:drbd drbd_resource=slurm_data
pcs resource create slurm_fs ocf:heartbeat:Filesystem device=/dev/drbd0 directory=/var/spool/slurm fstype=ext4
pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=<VIP> cidr_netmask=24
pcs resource create mariadb systemd:mariadb
pcs resource create slurmdbd_res systemd:slurmdbd
pcs resource create slurm_ctld_res systemd:slurmctld
```
### 🚨 7. Alertmanager
Alerting system for failures and thresholds.
```bash
apt install prometheus-alertmanager -y
nano /etc/alertmanager/alertmanager.yml
systemctl restart prometheus-alertmanager
```
### 🌐 8. Open OnDemand Web Portal
Web-based HPC job submission interface.
```
apt install ondemand -y
nano /etc/ood/config/clusters.d/hpc.yml
systemctl restart ondemand
```
### 📊 9. Monitoring – Prometheus
Cluster metrics collection.
```
apt install prometheus -y
apt install prometheus-node-exporter -y
systemctl enable prometheus --now
```
### 📈 10. Grafana Dashboard
Visualization for cluster monitoring.
```
apt install grafana -y
systemctl enable grafana-server --now
```

### Resource Group Example
DRBD
 → Filesystem
   → MariaDB
     → SlurmDBD
       → SlurmCTLD
         → VIP

###🧪 11. Failover Testing
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

### 🧰 12. Validation Commands
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

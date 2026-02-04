# 🚀 High Availability Slurm Cluster using Pacemaker, Corosync & DRBD

## 📌 Project Overview
This project implements a **Highly Available (HA) Slurm Workload Manager cluster** using:

- Pacemaker (Cluster Resource Manager)
- Corosync (Cluster Communication Layer)
- DRBD (Block-Level Data Replication)
- Virtual IP Failover
- MariaDB for Slurm Accounting
- Shared Slurm State Storage

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


## 🧩 Key Features

✅ Automatic Failover  
✅ Controller Redundancy  
✅ Data Replication  
✅ Virtual IP Switching  
✅ DRBD Synchronous Mode  
✅ Pacemaker Resource Groups  
✅ Service Ordering Constraints  

---
## ⚙️ Cluster Resource Flow

Pacemaker controls:

- DRBD Promotion/Demotion
- Filesystem Mount
- Database Service
- Slurm Services
- VIP Movement

---
3️⃣ Configure Corosync Cluster

apt install pacemaker corosync pcs drbd-utils mariadb-server slurm-wlm

pcs cluster auth MasterNode PassiveMaster
pcs cluster setup SlurmHA MasterNode PassiveMaster
pcs cluster start --all

4️⃣ Configure DRBD
Create resource:

/etc/drbd.d/slurm_data.res
Initialize:

drbdadm create-md slurm_data
drbdadm up slurm_data
Promote Primary:

drbdadm primary --force slurm_data

5️⃣ Filesystem Setup
mkfs.ext4 /dev/drbd0
mount /dev/drbd0 /var/spool/slurm

6️⃣ Create Pacemaker Resources
pcs resource create slurm_data_res ocf:linbit:drbd ...
pcs resource create slurm_fs Filesystem ...
pcs resource create mariadb systemd:mariadb
pcs resource create slurmdbd_res systemd:slurmdbd
pcs resource create slurm_ctld_res systemd:slurmctld
pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ...

7️⃣ Resource Group Example
DRBD
 → Filesystem
   → MariaDB
     → SlurmDBD
       → SlurmCTLD
         → VIP

🔄 Failover Testing
Move Resources
pcs node standby PassiveMaster
Check Cluster
pcs status
Slurm Health
scontrol ping
sinfo

📊 DRBD Sync Monitoring
cat /proc/drbd
watch -n1 drbdadm status

Output:

Primary/Secondary
SyncSource / SyncTarget
UpToDate/Inconsistent

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

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
Pacemaker controls:

- DRBD Promotion/Demotion
- Filesystem Mount
- Database Service
- Slurm Services
- VIP Movement

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


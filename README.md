# Multi-Node HPC Test Cluster (FreeIPA + Slurm + NFS)

A functional 3-node High-Performance Computing (HPC) testbed featuring centralized identity management via **FreeIPA**, job orchestration via **Slurm**, and shared storage via **NFS**. Automated using **Vagrant** and **Ansible** on **Rocky Linux 9**.

---

## 1. Cluster Architecture & Network Topology

| Hostname | Role | Example IP | Operating System | Core Services |
| :--- | :--- | :--- | :--- | :--- |
| **`ctrl.hpc.test`** | Controller / Head Node | `192.168.56.10` | Rocky Linux 9 | FreeIPA Server, `slurmctld`, NFS Server |
| **`node1.hpc.test`** | Compute Node 1 | `192.168.56.11` | Rocky Linux 9 | SSSD Client, `slurmd`, NFS Client |
| **`node2.hpc.test`** | Compute Node 2 | `192.168.56.12` | Rocky Linux 9 | SSSD Client, `slurmd`, NFS Client |

* **Network Subnet:** `192.168.56.0/24` (Private Host-Only Subnet)
* **Shared Storage:** `/home` exported via NFSv4 from `ctrl` to `node1` and `node2`
* **Authentication Domain:** `HPC.TEST` (FreeIPA Kerberos Realm)

---

## 2. Core Subsystems Overview

### Identity & Access Management (FreeIPA / SSSD)
* **Central Directory:** Hosted on `ctrl.hpc.test` for unified LDAP/Kerberos account management.
* **Client Integration:** SSSD runs on all compute nodes, ensuring user POSIX UIDs, GIDs, and Kerberos tickets stay synchronized across the cluster.

### Shared Storage (NFS)
* `/home` is exported via NFSv4 on `ctrl`.
* Compute nodes automatically mount `/home` so user scripts and job output logs (`job_%j.out`) remain synchronized across all nodes.

### Workload Orchestration (Slurm)
* **Control Daemon (`slurmctld`):** Runs on `ctrl.hpc.test` over TCP port `6817`.
* **Execution Daemon (`slurmd`):** Runs on compute nodes over TCP port `6818`.
* **Munge:** Handles secure authentication for all internal Slurm communications.

---

## 3. Quick Start & Job Submission Guide

### Step 1: Authenticate as a Cluster User
```bash
kinit <username>@HPC.TEST

## 📄 AKS Sizing & Compute Calculation Cheatsheet

### 1. Fundamental Cluster Capacity Formula

$$\text{Total Cluster Capacity} = \text{Node Count} \times \text{VM Size Capacity}$$

* **Total vCPUs:** $\text{Nodes} \times \text{vCPU per VM}$
* **Total RAM:** $\text{Nodes} \times \text{RAM per VM}$

---

### 2. Available Capacity Per Node (Allocatable Resources)

Azure reserves a portion of CPU and RAM on every node for OS, Kubelet, and System Services.

$$\text{Allocatable Resource} = \text{Total Node Resource} - \text{Azure System Overhead}$$

#### Standard Overhead Estimate (for a 2 vCPU / 8 GB RAM Node like `Standard_D2s_v4`)

* **Allocatable CPU:** $\sim 1.9 \text{ vCPU } (1900\text{m})$
*(where $1\text{ vCPU} = 1000\text{m}$ cores)*
* **Allocatable RAM:** $\sim 6.2 \text{ GB } (\sim 6350\text{ MB})$

---

### 3. Maximum Pod Capacity Formula Per Node

To determine how many pods can run on a single worker node, compute both CPU-bound and RAM-bound limits. **The smaller result determines the actual maximum capacity.**

#### A. CPU-based Capacity

$$\text{Max Pods (CPU)} = \left\lfloor \frac{\text{Allocatable CPU (in millicores)}}{\text{Pod CPU Request (in millicores)}} \right\rfloor$$

#### B. RAM-based Capacity

$$\text{Max Pods (RAM)} = \left\lfloor \frac{\text{Allocatable RAM (in MB)}}{\text{Pod RAM Request (in MB)}} \right\rfloor$$

#### C. Real Node Capacity (Bottleneck Rule)

$$\text{Max Pods per Node} = \min\left(\text{Max Pods (CPU)}, \text{Max Pods (RAM)}\right)$$

---

### 4. Total Cluster Pod Capacity Formula

For User Workload Node Pools (excluding System Node Pool):

$$\text{Total Pod Capacity} = \text{Total User Nodes} \times \text{Max Pods per Node}$$

---

### 5. Practical Example Calculation

#### **Scenario Setup:**

* **VM Size:** `Standard_D2s_v4` (2 vCPU, 8 GB RAM)
* **User Nodes:** 2 Nodes
* **Pod Spec (Small Pod):**
* **CPU Request:** `250m`
* **RAM Request:** `512 MB`

#### **Step-by-Step Calculation:**

1. **CPU Limit per Node:**

$$\frac{1900\text{m}}{250\text{m}} = 7.6 \longrightarrow \mathbf{7 \text{ Pods}}$$

1. **RAM Limit per Node:**

$$\frac{6350\text{ MB}}{512\text{ MB}} = 12.4 \longrightarrow \mathbf{12 \text{ Pods}}$$

1. **Max Guaranteed Pods per Node:**

$$\min(7, 12) = \mathbf{7 \text{ Pods per node}}$$

1. **Total Pod Capacity across 2 User Nodes:**

$$\text{Guaranteed (Full Load): } 2 \text{ Nodes} \times 7 \text{ Pods} = \mathbf{14 \text{ Pods}}$$

$$\text{Realistic (Normal Load with Bursting): } 2 \text{ Nodes} \times 10 \text{ to } 11 \text{ Pods} = \mathbf{20 \text{ to } 22 \text{ Pods}}$$

---

### 6. Quick Rules of Thumb for Azure AKS

* **System Pool Sizing:** Minimum **1–2 Nodes** (`Standard_D2s_v4`) reserved strictly for system pods (`CoreDNS`, `metrics-server`, etc.).
* **Azure Default Limit:** Max **110 pods per node** (or up to 250 with Azure CNI configuration).
* **Resource Over-commit Strategy:** Always define `requests` for scheduling and `limits` for safety in your Pod YAML.

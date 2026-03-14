

## **Project: Automated VM Metrics Collection & Email Report Using Ansible**

### **1. Project Objective**

* Monitor multiple VMs (100–500) for:

  * CPU usage
  * RAM usage
  * Disk availability
* Consolidate metrics into a report and send it via email automatically.
* Avoid manual intervention by automating with Ansible.

---

### **2. Environment Setup**

#### **2.1 Ansible Master**

* Create a VM for Ansible master:

  * Ubuntu 24.04 LTS
  * Instance type: t2.medium (or smaller for demo)
  * Open ports: 22 (SSH), 587 (SMTP email)
* Install Ansible:

  ```bash
  sudo apt update
  sudo apt install ansible -y
  ansible --version
  ```

#### **2.2 Target Servers**

* Launch multiple VMs (e.g., 10 for demo, 100–500 in real scenario).
* Tag VMs with `Environment=dev` (or relevant tag for filtering in dynamic inventory).

---

### **3. Dynamic Inventory**

* **Why:** Automatically fetch IP addresses of VMs without manually updating static inventory.
* **File:** `inventory/aws_ec2.yaml`
* **Key points:**

  * Use AWS plugin
  * Filter by tags and running state
  * Compose group names for hosts
* **Example:**

  ```yaml
  plugin: aws_ec2
  regions:
    - ap-south-1
  filters:
    "tag:Environment": "dev"
    instance-state-name: running
  compose:
    ansible_host: public_ip_address
  ```

---

### **4. SSH Key Setup**

1. Generate key pair on Ansible master:

   ```bash
   ssh-keygen -t rsa -b 4096 -C "ansible-master"
   ```
2. Copy public key to all target servers using a script:

   * Ensure `.ssh` folder permissions are correct
   * Use `chmod 400` for private key
3. Disable host key checking in `ansible.cfg`:

   ```ini
   [defaults]
   host_key_checking = False
   ```

---

### **5. Project Directory Structure**

```
VM-monitor/
├─ ansible.cfg
├─ inventory/
│   └─ aws_ec2.yaml
├─ group_vars/
│   └─ all.yaml
├─ templates/
│   └─ report_template.j2
├─ playbooks/
│   ├─ collect_metrics.yaml
│   └─ send_report.yaml
└─ main_playbook.yaml
```

---

### **6. Group Variables (`all.yaml`)**

* Store variables like:

  * SMTP server: `smtp.gmail.com`
  * Port: `587`
  * Email user & app password
  * Recipient email
* Keeps credentials and configurable info in one place.

---

### **7. Playbooks**

#### **7.1 Collect Metrics (`collect_metrics.yaml`)**

* Hosts: `env_dev` (all target VMs)
* Tasks:

  1. Install monitoring packages (`sysstat`)
  2. Collect CPU usage using `mpstat` and `awk`
  3. Collect memory usage using `free` and `awk`
  4. Collect disk usage using `df` and `awk`
  5. Store metrics in dictionary `vm_metrics` using `set_fact`

#### **7.2 Send Report (`send_report.yaml`)**

* Hosts: `localhost` (email is sent from master)
* Tasks:

  1. Render report using Jinja2 HTML template
  2. Send email using `mail` module (SMTP)

#### **7.3 Main Playbook (`main_playbook.yaml`)**

* Imports both `collect_metrics.yaml` and `send_report.yaml` so the full workflow can run with one command:

  ```bash
  ansible-playbook main_playbook.yaml
  ```

---

### **8. Email Report**

* HTML formatted table using Jinja2 template
* Columns:

  * VM Name
  * CPU Usage %
  * Memory Usage %
  * Disk Usage %
* Average metrics included at the bottom.

---


# Enterprise Linux Administration & Multi-Server Infrastructure Lab (RHCSA)

A practical implementation of enterprise-grade Red Hat Enterprise Linux (RHEL) system administration and infrastructure deployment based on Red Hat Certified System Administrator (RHCSA) requirements.

---

## 🛠️ Architecture & Setup Overview
- **Domain:** `lab.example.com`
- **Network Subnet:** `172.25.250.0/24`
- **Nodes:**
  - `servera.lab.example.com` (172.25.250.10)
  - `serverb.lab.example.com` (172.25.250.11)
  - `utility.lab.example.com` (Container Registry & Services)

---

## 📌 Key Implementations & Tasks

### 1. Network & System Configuration
- Configured static IPv4 networking, DNS, and gateway settings using `nmcli`.
- Synchronized system clocks across nodes via Chrony (`chronyd`) against an NTP server.
- Set up custom local AppStream and BaseOS YUM/DNF repositories.

### 2. Storage & Logical Volume Management (LVM)
- Partitioned storage drives (`fdisk`) and configured dedicated persistent Swap partitions (`mkswap`, `swapon`, `/etc/fstab`).
- Created Volume Groups with custom Physical Extent sizes (`vgcreate -s 8M`).
- Provisioned Logical Volumes (`lvcreate`), formatted with `ext3`, and executed live online filesystem resizing (`lvextend`, `resize2fs`).

### 3. Security, Hardening & Access Control
- **SELinux:** Resolved non-standard port issues by adjusting SELinux port labeling (`semanage port -a -t http_port_t`) and applying file context restoration (`restorecon`).
- **Firewall:** Granular traffic control and service allowances via `firewall-cmd`.
- **User & Group Security:** Configured collaborative SGID directories (`chmod 2770`), secure `umask` defaults, password aging policies (`login.defs`), and passwordless administrative escalation via `/etc/sudoers.d/`.

### 4. Centralized Network Storage & Automation
- **NFS & Autofs:** Configured an NFS server on `servera` and dynamic home directory automounting using `autofs` on `serverb`.
- **Scheduling & Scripting:** Automated recurring jobs using `crontab` and created Bash utility scripts for automated file system inspection.

### 5. Rootless Containers & Service Integration
- Built container images using `podman`.
- Deployed systemd user services (`~/.config/systemd/user/`) for persistent container execution across reboots.

---

## 🚀 Tech Stack
- **OS:** Red Hat Enterprise Linux (RHEL 9)
- **Tools & Services:** NetworkManager (`nmcli`), LVM2, SELinux, Firewalld, Autofs, NFS, Podman, Systemd, Chrony, Bash.

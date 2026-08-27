# Red Hat Enterprise Linux (RHEL) - Infrastructure Administration & Exam Lab

A comprehensive, hands-on implementation and walkthrough of multi-server enterprise administration tasks on Red Hat Enterprise Linux (RHEL), covering storage management, SELinux policies, automounting, rootless containers, security hardening, and automation.

---

## 📄 Lab Documentation Reference
* Complete lab execution output, terminal records, and validation evidence are documented in: **[RHCSA EXAM.pdf](./RHCSA%20EXAM.pdf)**

---

## 🌐 Environment Specifications
* **Domain:** `lab.example.com`
* **Subnet:** `172.25.250.0/24`
* **Infrastructure Nodes:**
  * `servera.lab.example.com` (`172.25.250.10`)
  * `serverb.lab.example.com` (`172.25.250.11`)
  * Gateway / DNS / NTP / Registry: `172.25.250.254`

---

## 🖥️ Node: serverb.lab.example.com

### 1. Network Configuration
```bash
hostnamectl set-hostname serverb.lab.example.com
nmcli connection modify enp1s0 ipv4.addresses 172.25.250.11/24
nmcli connection modify enp1s0 ipv4.gateway 172.25.250.254
nmcli connection modify enp1s0 ipv4.dns 172.25.250.254
nmcli connection modify enp1s0 ipv4.method manual
nmcli connection down enp1s0
nmcli connection up enp1s0
```

### 2. Local Repository Configuration
```bash
cat << 'EOF' > /etc/yum.repos.d/local.repo
[local-BaseOS]
name=BaseOS
baseurl=[http://classroom.example.com/content/rhel9.0/x86_64/dvd/BaseOS](http://classroom.example.com/content/rhel9.0/x86_64/dvd/BaseOS)
enabled=1
gpgcheck=0

[local-AppStream]
name=AppStream
baseurl=[http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream](http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream)
enabled=1
gpgcheck=0
EOF
```

### 3. SELinux & Firewall Web Service Remediation
```bash
semanage port -a -t http_port_t -p tcp 82
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload
restorecon -Rv /var/www/html
systemctl restart httpd
```

### 4. User & Group Management
```bash
groupadd admin
useradd -G admin harry
useradd -G admin natasha
useradd -s /sbin/nologin sarah

echo 'harry:password' | chpasswd
echo 'natasha:password' | chpasswd
echo 'sarah:password' | chpasswd
```

### 5. Collaborative SGID Directory
```bash
mkdir -p /common/admin
chown :admin /common/admin
chmod 2770 /common/admin
```

### 6. Autofs NFS Home Directories Automounting
```bash
useradd -u 2005 -d /localhome/production5 production5
echo "/localhome /etc/auto.localhome" > /etc/auto.master.d/localhome.autofs
echo "* -rw,sync servera.lab.example.com:/user-homes/&" > /etc/auto.localhome
systemctl daemon-reload
systemctl restart autofs
systemctl enable autofs
```

### 7. Scheduled Tasks (Cron)
```bash
crontab -u harry -e
# Add the following entry:
30 12 * * * /bin/echo "hello"
```

### 8. NTP Client Configuration (Chrony)
```bash
sed -i '1i server classroom.example.com iburst' /etc/chrony.conf
systemctl restart chronyd
chronyc makestep
```

### 9. File Search, Filtering & Archiving
```bash
# Locate and copy user sarah files
mkdir -p /root/find.user
find / -user sarah -exec cp -a {} /root/find.user/ \; 2>/dev/null

# String extraction
grep "home" /etc/passwd > /root/search.txt

# Archive generation
tar -czvf /root/test.tar.gz /var/tmp
```

### 10. User ID & Login Banner Configuration
```bash
useradd -u 1326 alies
echo 'echo "Welcome to Advantage Pro"' >> /home/alies/.bashrc
chown alies:alies /home/alies/.bashrc
```

### 11. Rootless Podman Container & Systemd User Service
```bash
# Create shared storage paths and set permissions
mkdir -p /opt/files /opt/processed
chmod 777 /opt/files /opt/processed
chcon -R -t container_file_t /opt/files /opt/processed

# Build and run container as student user
su - student
podman build -t monitor [http://classroom.example.com/Containerfile](http://classroom.example.com/Containerfile)
podman run -d --name ascii2pdf \
  -v /opt/files:/opt/incoming:Z \
  -v /opt/processed:/opt/outgoing:Z \
  monitor

# Create systemd user service for auto-start
mkdir -p ~/.config/systemd/user/
cd ~/.config/systemd/user/
podman generate systemd --name ascii2pdf --files --new
systemctl --user daemon-reload
systemctl --user enable --now container-ascii2pdf.service
```

### 12. System Hardening & Security Policies
```bash
# Default umask for user natasha (Files: -r-------- / Dirs: dr-x------)
echo "umask 0277" >> /home/natasha/.bashrc

# Password expiration baseline (20 days max)
sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS 20/' /etc/login.defs

# Administrative sudo privilege without password
echo "%admin ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/admin
chmod 0440 /etc/sudoers.d/admin
```

### 13. File Management Automation Script
```bash
cat << 'EOF' > /usr/local/bin/mysearch
#!/bin/bash
mkdir -p /root/myfiles
find /usr/share -type f -size -1M -exec cp -a {} /root/myfiles/ \; 2>/dev/null
EOF

chmod +x /usr/local/bin/mysearch
/usr/local/bin/mysearch
```

---

## 🖥️ Node: servera.lab.example.com

### 1. Root Password & NFS Share Configuration
```bash
passwd

# Export NFS user homes
systemctl enable --now nfs-server
mkdir -p /user-homes/production5
chmod -R 777 /user-homes
chown -R root:root /user-homes
echo "/user-homes *(rw,sync,no_root_squash)" > /etc/exports
exportfs -rva

# Firewall and SELinux boolean configuration
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
setsebool -P nfs_export_all_rw 1
setsebool -P nfs_export_all_ro 1
```

### 2. Local Repository Configuration
```bash
cat << 'EOF' > /etc/yum.repos.d/local.repo
[local-BaseOS]
name=RHEL Local BaseOS
baseurl=[http://classroom.example.com/content/rhel9.0/x86_64/dvd/BaseOS](http://classroom.example.com/content/rhel9.0/x86_64/dvd/BaseOS)
enabled=1
gpgcheck=0

[local-AppStream]
name=RHEL Local AppStream
baseurl=[http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream](http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream)
enabled=1
gpgcheck=0
EOF
```

### 3. Persistent Swap Space Configuration
```bash
# Partitioning with fdisk (/dev/vdc2 -> 512MiB, type 82: Linux swap)
fdisk /dev/vdc
mkswap /dev/vdc2
swapon /dev/vdc2
echo "/dev/vdc2 swap swap defaults 0 0" >> /etc/fstab
```

### 4. Logical Volume Management (LVM) & Dynamic Expansion
```bash
# Create Volume Group with custom 8MiB Physical Extent
vgcreate -s 8M datastore /dev/vdc3

# Allocate Logical Volume (50 extents = 400MiB) and format with ext3
lvcreate -l 50 -n database datastore
mkfs.ext3 /dev/datastore/database
mkdir -p /mnt/database
mount /dev/datastore/database /mnt/database
echo "/dev/datastore/database /mnt/database ext3 defaults 0 0" >> /etc/fstab

# Dynamically extend Logical Volume to 100 extents (800MiB) and resize filesystem online
lvextend -l 100 -r /dev/datastore/database
```

### 5. System Performance Tuning
```bash
tuned-adm profile $(tuned-adm recommend)
tuned-adm active
```

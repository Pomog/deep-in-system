### Tasks
- Set up an Ubuntu Server (latest LTS) as a Virtual Machine (VM) with specific disk partitions.
- Configure networking with a static private IP.
- Harden security by disabling remote root login, changing the SSH port, and setting up a firewall.
- Create user accounts with different SSH authentication methods.
- Install and configure several services (FTP server for a read-only user, MySQL server with restricted remote access, and WordPress).
- Automate backups of your WordPress database via a cron job.
- Extend project with bonus features such as a persistent Minecraft server, Ansible automation, and SSL for services.

### Actions
1. Download [The latest LTS version of Ubuntu Server](https://ubuntu.com/download/server)
2. Open VMware Workstation
3. Create a new virtual machine
4. Configure the VM:
- Allocate at least 30 GB of disk space.
- Assign a username that will serve as your login name.
- VM’s disk: Swap: 4 GB; Root (/): 15 GB; Home (/home): 5 GB; Backup (/backup): 6 GB
![image](https://github.com/Pomog/deep-in-system/blob/main/partition.png)
```
When installing Ubuntu, choose "Custom Storage Layout" or "Manual" or "Custom" partitioning.
Create a swap partition of 4 GB.
Create a primary partition mounted as / of 15 GB.
Create a separate partition for /home of 5 GB.
Create another partition for /backup of 6 GB.
Complete the installation.
```
5. Set the Hostname
```
sudo hostnamectl set-hostname yourusername-host
hostnamectl
```
![image](https://github.com/Pomog/deep-in-system/blob/main/hostname.png)

6. Network Configuration
On Ubuntu, networking is managed by Netplan. Open the Netplan configuration file (the filename might be 01-netcfg.yaml or similar in /etc/netplan/
```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```
![image](https://github.com/Pomog/deep-in-system/blob/main/50-cloud-init-old.png)
Replace <interface> with your network interface (e.g., ens33), and choose a static IP (for example, 192.168.1.100/24) and proper gateway:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    <interface>:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
or as gateway4 is deprecated in newer Netplan versions:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
```
Apply the Configuration
```bash
sudo netplan apply
```
** CHECK that VMware works in Bridged mode,  because in NAT mode, VMware creates a separate virtual network with its own gateway  **
Verify you can reach the Internet:
```bash
ping -c 5 google.com
```

6. Security Configuration
- Install OpenSSH server
```bash
sudo apt update
sudo apt install openssh-server
```
- Disable Remote Root Login & Change SSH Port
```
sudo ls /etc/ssh/
sudo vim /etc/ssh/sshd_config
```
Make these changes:
Disable root login: PermitRootLogin no
Change the SSH port: Port 2222

- start or restart ssh, or maybe VM restart in needed
```
sudo systemctl restart ssh
```
- Test SSH on the new port from another terminal or machine
```
Test-NetConnection 192.168.1.100 -Port 2222
or
ssh -p 2222 yourusername@yourserver-ip
ssh -p 2222 yurii@192.168.1.100 
```
![image](https://github.com/Pomog/deep-in-system/blob/main/testSSH.png)
![image](https://github.com/Pomog/deep-in-system/blob/main/testSSH_2.png)

- Configure the Firewall
Use UFW (Uncomplicated Firewall) to restrict access.
Allow only necessary ports (e.g., SSH on 2222, HTTP if using Apache for WordPress, FTP if needed)
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw allow 80/tcp
sudo ufw allow 21/tcp

sudo ufw enable
sudo ufw status verbose
```


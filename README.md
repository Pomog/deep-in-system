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
6. Network Configuration
On Ubuntu, networking is managed by Netplan. Open the Netplan configuration file (the filename might be 01-netcfg.yaml or similar in /etc/netplan/
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
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
Apply the Configuration
```bash
sudo netplan apply
```
Verify you can reach the Internet:
```bash
ping -c 5 google.com
```

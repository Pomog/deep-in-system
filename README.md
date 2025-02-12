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
![image](https://github.com/Pomog/deep-in-system/blob/main/ufw.png)

- User Management
To check the UID_MIN and UID_MAX values on your system, you can use the following command:
```
grep -E '^UID_MIN|^UID_MAX' /etc/login.defs
```
list all normal users in our Linux system:
```
getent passwd {1000..60000}
```
Create User “luffy” (Public Key Authentication)
```
sudo adduser fluffy
sudo usermod -aG sudo fluffy
```
#### Set up SSH keys for fluffy
- Generate an SSH Key Pair (on Your Local Machine)
```
ssh-keygen -t rsa -b 4096 -C "thoryur@gmail.com" 
```
su - fluffy
mkdir -p ~/.ssh && chmod 700 ~/.ssh
vim ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
Create User “zoro” (Password Authentication Only)
```
sudo adduser zoro
```
- FTP Server Installation (for User “nami”), use vsftpd for the FTP server
```
sudo apt update
sudo apt install vsftpd
```
Backup the original configuration file:
```
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
```
Edit vsftpd.conf:
```
sudo nano /etc/vsftpd.conf
```
```
anonymous_enable=NO
local_enable=YES
write_enable=NO
chroot_local_user=YES
```
To restrict user nami to /backup, you might set her home directory to /backup 
```
sudo useradd -d /backup -s /usr/sbin/nologin nami
sudo passwd nami
```
Restart vsftpd
```
sudo systemctl restart vsftpd
```
- Install MySQL Server
```
sudo apt install mysql-server
```
Ensure MySQL binds only to localhost:
Edit the MySQL configuration file (commonly /etc/mysql/mysql.conf.d/mysqld.cnf):
```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Confirm or set:
```
bind-address = 127.0.0.1
```
Restart MySQL:
```
sudo systemctl restart MySQL
```
- Create a WordPress database and dedicated user
Log into MySQL:
```
sudo mysql -u root -p
```
```
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'your_custom_password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
-  WordPress Installation
Install a web server (Apache):
```
sudo apt install apache2
```
- Download and extract WordPress:
```
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
sudo rsync -av wordpress/ /var/www/html/
```
Set permissions for the web server:
```
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```
Configure WordPress:
```
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sudo vim wp-config.php
```
```
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'your_custom_password');
define('DB_HOST', 'localhost');
```
![image](https://github.com/Pomog/deep-in-system/blob/main/php.png)
![image](https://github.com/Pomog/deep-in-system/blob/main/wp.png)

- SSL for Web and FTP Servers:
Configure self-signed SSL certificates for your Apache (or Nginx) web server and your FTP server. For Apache, you can enable SSL with:
```
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo systemctl reload apache2
```
- Exporting the Virtual Machine
Export the VM to a secure location using your virtualization software.
Generate a SHA1 checksum:
```
sha1sum deep-in-system.ova > deep-in-system.sha1
cat deep-in-system.sha1
```

### Generate SHA-1
This method involves directly copying the VM’s folder, which contains all the necessary files.
Navigate to the folder containing your VM files
```
Compress-Archive -Path * -DestinationPath Ubuntu.zip
Get-FileHash -Algorithm SHA1 Ubuntu.zip | Out-File deep-in-system.sha1
```

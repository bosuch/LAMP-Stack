# LAMP Stack Setup in Oracle VirtualBox

A step-by-step guide to setting up a fully functional **LAMP stack** (Linux, Apache, MySQL, PHP) inside an Oracle VirtualBox virtual machine. This is a useful local development environment for web projects before deploying to production.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Create the Virtual Machine](#2-create-the-virtual-machine)
3. [Install Ubuntu Server](#3-install-ubuntu-server)
4. [Configure the Network](#4-configure-the-network)
5. [Update the System](#5-update-the-system)
6. [Install Apache](#6-install-apache)
7. [Install MySQL](#7-install-mysql)
8. [Install PHP](#8-install-php)
9. [Test the LAMP Stack](#9-test-the-lamp-stack)
10. [Optional: Install phpMyAdmin](#10-optional-install-phpmyadmin)
11. [Tips and Troubleshooting](#11-tips-and-troubleshooting)

---

## 1. Prerequisites

Before you begin, make sure you have the following:

- **Oracle VirtualBox** installed on your host machine — [Download here](https://www.virtualbox.org/wiki/Downloads)
- **Ubuntu Server ISO** (LTS recommended) — [Download here](https://ubuntu.com/download/server)
- At least **2 GB RAM** and **20 GB disk space** to allocate to the VM
- A stable internet connection inside the VM

---

## 2. Create the Virtual Machine

1. Open VirtualBox and click **New**.
2. Set the following:
   - **VM Name:** `LAMP-Server` (or any name you prefer)
   - **ISO Image:** Select the Ubuntu ISO file you downloaded.
   - **OS:** Linux
   - **OS Version:** Ubuntu (64-bit)
   - **Proceed with Unattended Installation:** NOT checked.
3. Allocate **at least 2048 MB (2 GB)** of RAM and 2 CPUs.
4. Create a new virtual hard disk:
   - Type: **VDI (VirtualBox Disk Image)**
   - Storage: **Dynamically allocated**
   - Size: **20 GB** or more
5. Click **Finish**.

---

## 3. Install Ubuntu Server

1. Start the VM — it will boot from the ISO.
2. Select **Install Ubuntu Server** from the menu.
3. Follow the installation prompts:
   - Choose your **language** and **keyboard layout**
   - Use the default **network settings** (you'll adjust these later if needed)
   - Accept the **default disk partitioning** (use entire disk)
   - Create a **username and password** — remember these
   - When prompted, install **OpenSSH Server** (recommended for remote access)
4. Complete the installation and reboot.
5. Remove the ISO from the virtual drive when prompted, then press **Enter**.

---

## 4. Configure the Network

For the VM to be reachable from your host browser, configure a **Host-Only Adapter** alongside NAT.

1. Shut down the VM if it is running.
2. Go to **Settings → Network**:
   - **Adapter 1:** Attached to **NAT** (for internet access)
   - **Adapter 2:** Attached to **Host-Only Adapter** (for host-to-VM communication)
3. Start the VM and find the Host-Only IP address:

```bash
ip addr show
```

Note the `inet` address on the host-only interface (often `192.168.56.xxx`). You will use this to access Apache from your host browser.

---

## 5. Update the System

Log in to your VM and run a full system update before installing anything:

```bash
sudo apt update && sudo apt upgrade -y
```

This ensures you are working with the latest package lists and security patches.

---

## 6. Install Apache

Apache is the web server component of the LAMP stack.

```bash
sudo apt install apache2 -y
```

### Start and Enable Apache

```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```

### Verify Apache is Running

```bash
sudo systemctl status apache2
```

You should see `active (running)` in the output.

### Allow Apache Through the Firewall

If UFW is active, allow HTTP and HTTPS traffic:

```bash
sudo ufw allow in "Apache Full"
```

### Test in Browser

On your **host machine**, open a browser and navigate to your VM's Host-Only IP address (from Step 4):

```
http://192.168.56.xxx
```

You should see the **Apache2 Ubuntu Default Page**.

---

## 7. Install MySQL

MySQL is the database component of the stack.

```bash
sudo apt install mysql-server -y
```

### Start and Enable MySQL

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

### Run the Security Script

This removes insecure defaults and locks down the MySQL installation:

```bash
sudo mysql_secure_installation
```

Follow the prompts — it is recommended to:
- Set a strong **root password**
- Remove **anonymous users**
- Disallow **remote root login**
- Remove the **test database**
- Reload privilege tables

### Test MySQL Login

```bash
sudo mysql -u root -p
```

Enter your root password. You should see the MySQL prompt. Type `exit` to leave.

---

## 8. Install PHP

PHP is the scripting language that connects Apache and MySQL.

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

### Verify the PHP Version

```bash
php -v
```

### Restart Apache

Apache needs to be restarted to load the PHP module:

```bash
sudo systemctl restart apache2
```

### Configure Apache to Prefer PHP Files (Optional)

Edit the directory index file so Apache serves `index.php` before `index.html`:

```bash
sudo nano /etc/apache2/mods-enabled/dir.conf
```

Change the line to:

```
DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
```

Save with `Ctrl+O`, then exit with `Ctrl+X`. Restart Apache again:

```bash
sudo systemctl restart apache2
```

---

## 9. Test the LAMP Stack

Create a PHP info page to confirm all three components are working together.

```bash
sudo nano /var/www/html/info.php
```

Paste the following content:

```php
<?php
phpinfo();
?>
```

Save and exit (`Ctrl+O`, `Ctrl+X`).

### View the Test Page

In your host browser, navigate to:

```
http://192.168.56.xxx/info.php
```

You should see the full **PHP Information** page, which confirms Apache is serving PHP correctly.

> **Security Note:** Remove this file when you are done testing — it exposes sensitive server configuration information.

```bash
sudo rm /var/www/html/info.php
```

---

## 10. Optional: Install phpMyAdmin

phpMyAdmin provides a web-based GUI for managing MySQL databases.

```bash
sudo apt install phpmyadmin -y
```

During installation:
- Select **apache2** as the web server (press `Space` to select, then `Enter`)
- Choose **Yes** to configure the database with `dbconfig-common`
- Set a password for phpMyAdmin

### Create a Symbolic Link (if needed)

```bash
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```

### Access phpMyAdmin

```
http://192.168.56.xxx/phpmyadmin
```

Log in with your MySQL root credentials.

---

## 11. Tips and Troubleshooting

| Issue | Solution |
|---|---|
| Apache page not loading in browser | Confirm the Host-Only Adapter IP with `ip addr show`; ensure the VM firewall allows port 80 |
| MySQL won't start | Check logs with `sudo journalctl -u mysql` |
| PHP not rendering (shows as text) | Restart Apache: `sudo systemctl restart apache2` |
| phpMyAdmin 404 error | Verify the symlink: `ls -la /var/www/html/phpmyadmin` |
| Can't connect VM to internet | Confirm Adapter 1 is set to NAT in VirtualBox network settings |

### Useful Commands Reference

```bash
# Apache
sudo systemctl start|stop|restart|status apache2

# MySQL
sudo systemctl start|stop|restart|status mysql

# Check UFW firewall status
sudo ufw status

# View Apache error log
sudo tail -f /var/log/apache2/error.log

# Web root directory
/var/www/html/
```

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

# Ubuntu Server Setup in VirtualBox with PHP 4.3, Apache, MySQL, Laravel, Vue.js, Git, and DNSMasq

## Prerequisites
- **Host Machine:** Windows
- **Guest OS:** Ubuntu Server in VirtualBox
- **Network Configuration:** Bridged Adapter or Host-Only Network
- **Access Method:** SSH from Windows to Ubuntu Server

---

## 1. Create a Non-Root SSH User

1. Log in to your Ubuntu Server:
   ```bash
   sudo su
   ```
2. Create a new user (replace `devuser` with your preferred username):
   ```bash
   adduser devuser
   ```
3. Grant sudo privileges:
   ```bash
   usermod -aG sudo devuser
   ```
4. Enable SSH access:
   ```bash
   nano /etc/ssh/sshd_config
   ```
   - Find `PermitRootLogin` and set it to `no`
   - Save and exit
5. Restart SSH service:
   ```bash
   systemctl restart ssh
   ```
6. Find your server's IP:
   ```bash
   ip a | grep inet
   ```
7. Connect from Windows using PowerShell or an SSH client:
   ```powershell
   ssh devuser@your-server-ip
   ```

---

## 2. Install Apache, PHP 4.3, and MySQL

1. Update system packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. Install Apache:
   ```bash
   sudo apt install apache2 -y
   ```
3. Install PHP 8.3:
   ```bash
   sudo apt install php8.3-cli php8.3-common php8.3-mysql -y
   ```
4. Install MySQL:
   ```bash
   sudo apt install mysql-server -y
   ```
5. Secure MySQL installation:
   ```bash
   sudo mysql_secure_installation
   ```
6. Restart services:
   ```bash
   sudo systemctl restart apache2
   sudo systemctl restart mysql
   ```

---

## 3. Install Laravel, Vue.js, Git, and Other Dependencies

1. Install Composer:
   ```bash
   curl -sS https://getcomposer.org/installer | php
   sudo mv composer.phar /usr/local/bin/composer
   ```
2. Install Node.js and npm:
   ```bash
   sudo apt install nodejs npm -y
   ```
3. Install Vue CLI:
   ```bash
   npm install -g @vue/cli
   ```
4. Install Git:
   ```bash
   sudo apt install git -y
   ```
5. Verify installations:
   ```bash
   php -v
   mysql --version
   composer -V
   node -v
   vue --version
   git --version
   ```

---

## 4. Setup Laravel Project

1. Create a Laravel project:
   ```bash
   composer create-project --prefer-dist laravel/laravel api_project
   ```
2. Configure Apache for Laravel:
   ```bash
   sudo nano /etc/apache2/sites-available/api.domain.local.conf
   ```
   - Add the following:
   ```apache
   <VirtualHost *:80>
       ServerName api.domain.local
       DocumentRoot /home/devuser/api_project/public

       <Directory /home/devuser/api_project>
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
3. Enable the site and restart Apache:
   ```bash
   sudo a2ensite api.domain.local.conf
   sudo systemctl restart apache2
   ```

---

## 5. Install and Configure DNSMasq for Dynamic DNS

### Install DNSMasq
```bash
sudo apt install dnsmasq -y
```

### Configure DNSMasq
```bash
sudo nano /etc/dnsmasq.conf
```
- Add the following line at the end:
```conf
address=/.domain.local/192.168.56.101
```
- Replace `192.168.56.101` with your **Ubuntu Server's IP Address**

### Restart DNSMasq
```bash
sudo systemctl restart dnsmasq
```

### Configure Ubuntu to Use DNSMasq as Default Resolver
```bash
sudo nano /etc/resolv.conf
```
- Add this line:
```conf
nameserver 127.0.0.1
```

- Prevent `resolv.conf` from being overwritten:
```bash
sudo chattr +i /etc/resolv.conf
```

### Configure Windows to Use Ubuntu Server as DNS

1. Open **Control Panel** > **Network and Sharing Center**
2. Select your active network > **Properties**
3. Double-click on **Internet Protocol Version 4 (TCP/IPv4)**
4. Select **Use the following DNS server addresses**
5. Enter `192.168.56.101` (Ubuntu Server's IP)
6. Click **OK** and restart your connection

### Test DNS Resolution from Windows
```powershell
nslookup api.domain.local
nslookup test.vue_project.domain.local
```

---

## 6. Setup Vue.js Project with Wildcard DNS

1. Create a Vue.js project:
   ```bash
   vue create vue_project
   ```
2. Serve Vue.js project:
   ```bash
   cd vue_project
   npm run serve -- --host 0.0.0.0
   ```
3. Access Vue.js project from Windows:
   ```
   http://sub.vue_project.domain.local:8080
   ```

---

## 7. Access Projects from Windows

- Open browser and visit:
  - Laravel API: `http://api.domain.local`
  - Vue.js Project: `http://sub.vue_project.domain.local:8080`

- Access Ubuntu via SSH:
  ```powershell
  ssh devuser@192.168.56.101
  ```

---

## 8. Deployment Considerations

- Use Apache for Laravel (`DocumentRoot` pointing to `public/`)
- Use Nginx or Apache as a reverse proxy for Vue.js
- Use `pm2` for Vue.js production build:
  ```bash
  npm run build
  pm2 serve dist 8080 --spa
  ```
- Set up SSL certificates using Let's Encrypt for production

---

Now, your Windows machine can dynamically resolve domain names via DNSMasq, without manually editing the hosts file. 🚀


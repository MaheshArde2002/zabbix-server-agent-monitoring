# Zabbix 7.0 Server & Agent Monitoring Lab

A hands-on Linux monitoring project demonstrating the installation, configuration, and deployment of **Zabbix 7.0 Server** with **MySQL**, **Apache**, and **Zabbix Agent** on Ubuntu.

The project also demonstrates how to configure a remote Linux client and add it to the Zabbix Server for monitoring.

---

## 📌 Project Overview

In this project, I configured a Zabbix monitoring environment with:

* Zabbix Server
* Zabbix Agent
* MySQL database
* Apache2 web server
* Zabbix Web Frontend
* Linux client monitoring
* Host and template configuration
* Service management using `systemctl`

The project was completed in a Linux virtual-machine lab environment.

---

## 🏗️ Project Architecture

```text
                    Zabbix Web Interface
                           │
                           │ HTTP
                           ▼
                ┌─────────────────────┐
                │    Zabbix Server    │
                │    192.168.0.105    │
                │                     │
                │  Zabbix Server      │
                │  MySQL              │
                │  Apache2            │
                │  Zabbix Frontend    │
                └──────────┬──────────┘
                           │
                    Zabbix Agent
                    Port: 10050
                           │
                           ▼
                ┌─────────────────────┐
                │    Linux Client     │
                │    192.168.0.108    │
                │                     │
                │    Zabbix Agent     │
                └─────────────────────┘
```

### Environment

| Component         | Details         |
| ----------------- | --------------- |
| Monitoring Server | `192.168.0.105` |
| Monitored Client  | `192.168.0.108` |
| Monitoring Server | Zabbix 7.0      |
| Database          | MySQL           |
| Web Server        | Apache2         |
| Agent             | Zabbix Agent    |
| Agent Port        | `10050`         |
| Server Port       | `10051`         |
| OS                | Ubuntu Linux    |

---

# 🚀 Installation and Configuration

## Phase 1 — Install Zabbix Server

### 1. Add Zabbix Repository

Download and install the Zabbix repository package.

```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb

sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb

sudo apt update
```

---

### 2. Install Zabbix Server and Required Packages

```bash
sudo apt install zabbix-server-mysql \
zabbix-frontend-php \
zabbix-apache-conf \
zabbix-sql-scripts \
zabbix-agent -y
```

Installed components include:

* Zabbix Server
* Zabbix Web Frontend
* Apache configuration
* MySQL database scripts
* Zabbix Agent

---

## Phase 2 — Configure MySQL Database

### 3. Log in to MySQL

```bash
sudo mysql
```

Check existing databases:

```sql
SHOW DATABASES;
```

---

### 4. Create Zabbix Database

```sql
CREATE DATABASE zabbix
CHARACTER SET utf8mb4
COLLATE utf8mb4_bin;
```

Create the Zabbix database user:

```sql
CREATE USER 'zabbix'@'localhost'
IDENTIFIED BY '<DB_PASSWORD>';
```

Grant permissions:

```sql
GRANT ALL PRIVILEGES
ON zabbix.*
TO 'zabbix'@'localhost';
```

Enable function creation during schema import:

```sql
SET GLOBAL log_bin_trust_function_creators = 1;
```

> Replace `<DB_PASSWORD>` with your own database password. Do not commit the real password to GitHub.

---

## Phase 3 — Import Zabbix Database Schema

Import the initial Zabbix database schema:

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz \
| mysql --default-character-set=utf8mb4 \
-uzabbix -p zabbix
```

After the import is complete, disable the temporary setting:

```sql
SET GLOBAL log_bin_trust_function_creators = 0;
```

---

## Phase 4 — Configure Zabbix Server

Edit the Zabbix server configuration:

```bash
sudo vim /etc/zabbix/zabbix_server.conf
```

Configure the database credentials:

```ini
DBUser=zabbix
DBPassword=<DB_PASSWORD>
```

Save the configuration file.

---

## Phase 5 — Start and Enable Services

Restart the required services:

```bash
sudo systemctl restart zabbix-server
sudo systemctl restart zabbix-agent
sudo systemctl restart apache2
```

Enable them at system startup:

```bash
sudo systemctl enable zabbix-server
sudo systemctl enable zabbix-agent
sudo systemctl enable apache2
```

Check service status:

```bash
sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent
sudo systemctl status apache2
```

---

# 🖥️ Phase 6 — Install Zabbix Agent on Linux Client

On the client machine:

```bash
sudo apt update
sudo apt install zabbix-agent -y
```

---

## 7. Configure Zabbix Agent

Edit the agent configuration:

```bash
sudo vim /etc/zabbix/zabbix_agentd.conf
```

Configure the Zabbix Server address:

```ini
Server=192.168.0.105
ServerActive=192.168.0.105
Hostname=zabbix-agent
```

The server IP tells the agent which Zabbix Server should communicate with it.

The hostname must match the host configuration created later in the Zabbix Web UI.

---

## 8. Restart and Verify Agent

Restart the agent:

```bash
sudo systemctl restart zabbix-agent
```

Check its status:

```bash
sudo systemctl status zabbix-agent
```

The service should show:

```text
Active: active (running)
```

---

# 🌐 Phase 7 — Configure Zabbix Web Frontend

Open the Zabbix Web interface in a browser:

```text
http://192.168.0.105/zabbix/
```

The web installation wizard guides through:

1. Welcome page
2. PHP prerequisite verification
3. Database connection
4. Zabbix server configuration
5. Web frontend installation
6. Login

---

## 🔐 Database Connection

Use the following database details:

```text
Database type: MySQL
Database name: zabbix
Database user: zabbix
Database password: <DB_PASSWORD>
```

---

# 📊 Phase 8 — Access Zabbix Dashboard

After completing the web setup, log in to the Zabbix Web UI.

The dashboard provides access to:

* Monitoring
* Hosts
* Problems
* Data collection
* Configuration
* Reports
* System information

---

# 🖥️ Phase 9 — Add Linux Client Host

First verify the client IP:

```bash
ip r l
```

Expected client address:

```text
192.168.0.108
```

In the Zabbix Web UI:

```text
Data Collection
      ↓
Hosts
      ↓
Create Host
```

Configure the host:

| Setting    | Value                   |
| ---------- | ----------------------- |
| Host name  | `zabbix-agent`          |
| Host group | `Linux servers`         |
| Template   | `Linux by Zabbix agent` |
| Agent IP   | `192.168.0.108`         |
| Agent Port | `10050`                 |

Save the host configuration.

---

# ✅ Verification

After adding the host, check the Hosts page.

The client should appear as:

```text
zabbix-agent
```

The ZBX availability indicator should become green when communication between the Zabbix Server and Agent is working correctly.

---

# 📸 Screenshots

All project screenshots are stored inside the `screenshots/` directory.

```text
screenshots/
│
├── 01_official_zabbix_installation_guide.png
├── 02_add_zabbix_repository_and_update.png
├── 03_install_zabbix_server_frontend_and_agent.png
├── 04_backup_config_restart_and_enable_services.png
├── 05_mysql_root_login_and_database_check.png
├── 06_create_zabbix_db_user_and_grant_privileges.png
├── 07_import_zabbix_initial_db_schema.png
├── 08_configure_dbpassword_in_zabbix_server_conf.png
├── 09_restart_and_enable_all_zabbix_services.png
│
├── 10_install_zabbix_agent_on_client.png
├── 11_configure_zabbix_agent_server_ip_and_hostname.png
├── 12_verify_zabbix_agent_service_status.png
│
├── 13_zabbix_web_setup_welcome_page.png
├── 14_zabbix_web_setup_prerequisites_check.png
├── 15_zabbix_web_setup_configure_db_connection.png
├── 16_zabbix_web_frontend_login_page.png
├── 17_zabbix_global_view_dashboard.png
├── 18_zabbix_hosts_configuration_page.png
├── 19_check_client_machine_ip_address.png
├── 20_configure_new_host_details_and_templates.png
└── 21_zabbix_agent_host_added_successfully.png
```

---

# 🧰 Important Commands

### Check Zabbix Server

```bash
sudo systemctl status zabbix-server
```

### Check Zabbix Agent

```bash
sudo systemctl status zabbix-agent
```

### Restart Zabbix Server

```bash
sudo systemctl restart zabbix-server
```

### Restart Zabbix Agent

```bash
sudo systemctl restart zabbix-agent
```

### Check Apache

```bash
sudo systemctl status apache2
```

### Check IP Address

```bash
ip addr
```

or:

```bash
ip r l
```

### Check Listening Ports

```bash
sudo ss -tulpn
```

---

# 🎯 Skills Demonstrated

Through this project, I practiced:

* Linux server administration
* Zabbix monitoring
* Zabbix Server installation
* Zabbix Agent configuration
* MySQL database administration
* Apache web server configuration
* Linux service management
* Systemd
* Package management with APT
* Configuration file management
* Linux networking
* Host monitoring
* Monitoring templates
* Basic troubleshooting

---

# 📚 Project Learning Outcomes

This project helped me understand how a Linux monitoring system is deployed from the beginning.

I practiced installing the Zabbix Server, configuring its MySQL backend, setting up the web frontend, installing the Zabbix Agent on a separate Linux machine, and connecting the client to the monitoring server.

I also learned how to verify services, configure Linux monitoring hosts, and confirm agent availability from the Zabbix dashboard.

---

# 🔧 Troubleshooting

If the Zabbix Agent does not become available, check:

### 1. Agent service

```bash
sudo systemctl status zabbix-agent
```

### 2. Agent configuration

```bash
sudo vim /etc/zabbix/zabbix_agentd.conf
```

Verify:

```ini
Server=192.168.0.105
ServerActive=192.168.0.105
Hostname=zabbix-agent
```

### 3. Network connectivity

From the client:

```bash
ping 192.168.0.105
```

### 4. Port availability

```bash
sudo ss -tulpn | grep 10050
```

### 5. Check Zabbix Server logs

```bash
sudo tail -f /var/log/zabbix/zabbix_server.log
```

---

# 👨‍💻 Project Type

**Hands-on Linux Monitoring & System Administration Lab**

This project was created for learning and demonstrating practical experience with Linux system administration, monitoring, networking, services, and database configuration.

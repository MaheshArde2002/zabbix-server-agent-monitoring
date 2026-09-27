Server Side: Zabbix Server Setup

1. Repository Setup & Updates

# Download the Zabbix 7.0 release package for Ubuntu 24.04
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb

# Install the repository configuration package
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb

# Update local package lists
apt update


2. Core Package Installation

# Install Zabbix Server, Web Frontend, Apache configuration, SQL scripts, and Agent
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y


3. MySQL Database Setup

# Connect to MySQL console as root
mysql -u root -p


-- Create database with required character set and collation
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

-- Create zabbix user and grant local privileges
CREATE USER zabbix@localhost IDENTIFIED BY 'Admin@3306';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;

-- Enable binary log trust for initial function creation
SET GLOBAL log_bin_trust_function_creators = 1;

-- Exit MySQL console
EXIT;


4. Database Schema Import

# Import default schema into the zabbix database
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix


# Disable binary log trust after schema import
mysql -u root -p -e "SET GLOBAL log_bin_trust_function_creators = 0;"


5. Server Configuration Update

# Edit server configuration file
nano /etc/zabbix/zabbix_server.conf

# Modify/uncomment the DBPassword line to match:
# DBPassword=Admin@3306


6. Restart & Enable Server Services

# Restart Zabbix Server, Zabbix Agent, and Apache Web Server
systemctl restart zabbix-server zabbix-agent apache2

# Enable services to launch automatically at system boot
systemctl enable zabbix-server zabbix-agent apache2

# Check status of Zabbix Server
systemctl status zabbix-server


💻 Client Side: Target Machine Setup

1. Install Zabbix Agent

# Update package indices and install agent
apt update && apt install zabbix-agent -y


2. Identify Host IP Address

# Check client IP address details
ip r l
# Or alternative command:
ip a


3. Configure Agent Connection

# Edit agent configuration file
nano /etc/zabbix/zabbix_agentd.conf

# Set Zabbix Server IP and client hostname:
# Server=192.168.0.105
# ServerActive=192.168.0.105
# Hostname=zabbix-agent


4. Restart & Verify Agent Service

# Restart Zabbix agent daemon
systemctl restart zabbix-agent

# Enable service startup at boot
systemctl enable zabbix-agent

# Verify agent running state
systemctl status zabbix-agent



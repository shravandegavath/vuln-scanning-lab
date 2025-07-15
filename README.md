🧪 Vulnerability Scanning Lab Setup (Enterprise Style)

This document outlines the full setup of a segmented, enterprise-style vulnerability scanning lab using pfSense, Kali Linux (attacker), Windows 10 (target), and Ubuntu with DVWA (web target). This lab will later be used for scanning with Nmap, Nessus, OpenVAS, Nikto, ZAP, and exploitation with Metasploit.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

🛡️ 1. Lab Architecture Overview

+---------------------+        +------------------------+

| Kali Linux (Attacker)| <--> | pfSense Firewall/Router |

+---------------------+        +-----------+------------+

&nbsp;                                       |

&nbsp;    +-------------------+--------------+----------------+--------------+

&nbsp;    |                                   |                              |

+------------+                +----------------+          +----------------+

| Windows 10 | (OPT1)         | Ubuntu + DVWA  | (OPT2)   | Inter🧪 Vulnerability Scanning Lab Setup (Enterprise Style)

This document outlines the full setup of a segmented, enterprise-style vulnerability scanning lab using pfSense, Kali Linux (attacker), Windows 10 (target), and Ubuntu with DVWA (web target). This lab will later be used for scanning with Nmap, Nessus, OpenVAS, Nikto, ZAP, and exploitation with Metasploit.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



🛡️ 1. Lab Architecture Overview

+---------------------+        +------------------------+

| Kali Linux (Attacker)| <--> | pfSense Firewall/Router |

+---------------------+        +-----------+------------+

&nbsp;                                       |

&nbsp;    +-------------------+--------------+----------------+--------------+

&nbsp;    |                                   |                              |

+------------+                +----------------+          +----------------+

| Windows 10 | (OPT1)         | Ubuntu + DVWA  | (OPT2)   | Internet (WAN) |

| 10.20.20.x |                | 10.30.30.x      |         | via pfSense     |

+------------+                +----------------+          +----------------+

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



⚙️ 2. VM Setup

👤 Kali Linux (Attacker)

•	OS: Kali Linux (latest)

•	Network: LAN (10.10.10.0/24 via pfSense DHCP)

🪟 Windows 10 (Target)

•	OS: Windows 10 Pro

•	Network: OPT1 (10.20.20.0/24)

•	Installed tools: Enable RDP, SMB, optional: old Java/.NET

👤 Ubuntu (Web Target)

•	OS: Ubuntu 20.04 or 22.04

•	Network: OPT2 (10.30.30.0/24)

•	Role: Host DVWA web application

🌐 pfSense Firewall

•	NIC1 (LAN): 10.10.10.1/24 (DHCP for Kali)

•	NIC2 (OPT1 - WindowsNet): 10.20.20.1/24 (DHCP enabled)

•	NIC3 (OPT2 - DMZNet): 10.30.30.1/24 (DHCP enabled)

•	NIC4 (WAN): Bridged/NAT (Internet access)

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



🔧 3. pfSense Configuration

Interface Setup

•	Add interfaces for OPT1 and OPT2

•	Assign static IPs and enable DHCP servers

Firewall Rules

•	Allow DNS (UDP 53), HTTP/HTTPS (TCP 80/443), and ICMP on OPT1 and OPT2

•	Temporarily allow all outbound for testing

•	Allow LAN (Kali) to access OPT1 and OPT2

•	Restrict cross-segment traffic later for realism

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



🛠️ 4. Ubuntu DVWA Setup

Install LAMP Stack

sudo apt update \&\& sudo apt upgrade -y          # Update and upgrade system packages

sudo apt install apache2 mariadb-server php php-mysqli php-gd libapache2-mod-php git -y  # Install Apache, MariaDB, PHP, and Git

Start and Enable Services

sudo systemctl enable apache2 mariadb           # Enable Apache and MariaDB on boot

sudo systemctl start apache2 mariadb            # Start Apache and MariaDB services

Secure MySQL (Optional)

sudo mysql\_secure\_installation                  # Run guided security configuration for MariaDB

Create Database and User

sudo mysql -u root -p                           # Log into MariaDB as root

CREATE DATABASE dvwa;                           -- Create DVWA database

CREATE USER 'dvwauser'@'localhost' IDENTIFIED BY 'dvwapassword';  -- Create DVWA user

GRANT ALL PRIVILEGES ON dvwa.\* TO 'dvwauser'@'localhost';         -- Grant privileges

FLUSH PRIVILEGES;                               -- Reload privileges

EXIT;                                           -- Exit MySQL

Download and Configure DVWA

cd /var/www/html                                # Navigate to Apache web root

sudo git clone https://github.com/digininja/DVWA.git  # Clone DVWA repo

cd DVWA/config                                   # Go to config directory

sudo cp config.inc.php.dist config.inc.php      # Copy sample config

sudo nano config.inc.php                        # Edit DB config

Update DB credentials:

$\_DVWA\[ 'db\_user' ] = 'dvwauser';

$\_DVWA\[ 'db\_password' ] = 'dvwapassword';

Final Config

sudo chown -R www-data:www-data /var/www/html/DVWA    # Set Apache ownership

sudo chmod -R 755 /var/www/html/DVWA                  # Set correct permissions

sudo a2enmod rewrite                                  # Enable Apache rewrite module

sudo systemctl restart apache2                        # Restart Apache to apply changes

Access DVWA

•	From Kali: http://10.30.30.X/DVWA/setup.php

•	Click “Create/Reset Database”

•	Login with admin:password

•	Set Security Level in DVWA settings (Low, Medium, High)

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



✅ Next Steps

•	Scan DVWA with Nmap, Nikto, ZAP

•	Scan Windows 10 with Nessus, OpenVAS

•	Document scan results and risk analysis

•	Perform validation with Metasploit (coming next)

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

This lab is now ready for real-world vulnerability scanning simulation with multiple network zones, an attacker machine, and authentic enterprise segmentation. Commit all configs, IP maps, and screenshots to GitHub for portfolio.

net (WAN) |

| 10.20.20.x |                | 10.30.30.x      |         | via pfSense     |

+------------+                +----------------+          +----------------+

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



⚙️ 2. VM Setup

👤 Kali Linux (Attacker)

•	OS: Kali Linux (latest)

•	Network: LAN (10.10.10.0/24 via pfSense DHCP)

🪟 Windows 10 (Target)

•	OS: Windows 10 Pro

•	Network: OPT1 (10.20.20.0/24)

•	Installed tools: Enable RDP, SMB, optional: old Java/.NET

👤 Ubuntu (Web Target)

•	OS: Ubuntu 20.04 or 22.04

•	Network: OPT2 (10.30.30.0/24)

•	Role: Host DVWA web application

🌐 pfSense Firewall

•	NIC1 (LAN): 10.10.10.1/24 (DHCP for Kali)

•	NIC2 (OPT1 - WindowsNet): 10.20.20.1/24 (DHCP enabled)

•	NIC3 (OPT2 - DMZNet): 10.30.30.1/24 (DHCP enabled)

•	NIC4 (WAN): Bridged/NAT (Internet access)

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



🔧 3. pfSense Configuration

Interface Setup

•	Add interfaces for OPT1 and OPT2

•	Assign static IPs and enable DHCP servers

Firewall Rules

•	Allow DNS (UDP 53), HTTP/HTTPS (TCP 80/443), and ICMP on OPT1 and OPT2

•	Temporarily allow all outbound for testing

•	Allow LAN (Kali) to access OPT1 and OPT2

•	Restrict cross-segment traffic later for realism

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



🛠️ 4. Ubuntu DVWA Setup

Install LAMP Stack

sudo apt update \&\& sudo apt upgrade -y          # Update and upgrade system packages

sudo apt install apache2 mariadb-server php php-mysqli php-gd libapache2-mod-php git -y  # Install Apache, MariaDB, PHP, and Git

Start and Enable Services

sudo systemctl enable apache2 mariadb           # Enable Apache and MariaDB on boot

sudo systemctl start apache2 mariadb            # Start Apache and MariaDB services

Secure MySQL (Optional)

sudo mysql\_secure\_installation                  # Run guided security configuration for MariaDB

Create Database and User

sudo mysql -u root -p                           # Log into MariaDB as root

CREATE DATABASE dvwa;                           -- Create DVWA database

CREATE USER 'dvwauser'@'localhost' IDENTIFIED BY 'dvwapassword';  -- Create DVWA user

GRANT ALL PRIVILEGES ON dvwa.\* TO 'dvwauser'@'localhost';         -- Grant privileges

FLUSH PRIVILEGES;                               -- Reload privileges

EXIT;                                           -- Exit MySQL

Download and Configure DVWA

cd /var/www/html                                # Navigate to Apache web root

sudo git clone https://github.com/digininja/DVWA.git  # Clone DVWA repo

cd DVWA/config                                   # Go to config directory

sudo cp config.inc.php.dist config.inc.php      # Copy sample config

sudo nano config.inc.php                        # Edit DB config

Update DB credentials:

$\_DVWA\[ 'db\_user' ] = 'dvwauser';

$\_DVWA\[ 'db\_password' ] = 'dvwapassword';

Final Config

sudo chown -R www-data:www-data /var/www/html/DVWA    # Set Apache ownership

sudo chmod -R 755 /var/www/html/DVWA                  # Set correct permissions

sudo a2enmod rewrite                                  # Enable Apache rewrite module

sudo systemctl restart apache2                        # Restart Apache to apply changes

Access DVWA

•	From Kali: http://10.30.30.X/DVWA/setup.php

•	Click “Create/Reset Database”

•	Login with admin:password

•	Set Security Level in DVWA settings (Low, Medium, High)


#
# <a name="_9bjh4aw0unos"></a><a name="_fqnirldy4dqh"></a>Step-by-step installation procedur
# <a name="_s73dbqcn558h"></a>**System requirements**
This document will inform entire[ ](https://evershop.io/docs/development/getting-started/system-requirements)[EverShop System Requirements](https://evershop.io/docs/development/getting-started/system-requirements) for you to follow and get the most effective EverShop system for your store.

## <a name="_v36b7k7bsr46"></a>**Operating systems​**
EverShop can run on Window, Linux or Mac-OS. **This document is based on debian 12**.

Install debian 12 iso.

We wll setup 2 different debian 12 vm. One is for **Evershop Serve**r . Other one is for **Postgresql Database Serv**er.
## <a name="_729zkko3mxsj"></a>Evershop Server
## <a name="_p9ng03ua34kx"></a>**Networking**
On virtualbox for evershop vm select first adapter as a bridge adapter to connect to your  local network. DHCP will assign an ipv4 address on enps03 interface,

Ensure that  on /etc/network/interfaces have below.

`  `GNU nano 7.2                	/etc/network/interfaces                         	 

\# This file describes the network interfaces available on your system

\# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/\*

\# The loopback network interface

auto lo

iface lo inet loopback

auto enp0s3

iface enp0s3 inet dhcp


enter ip address show command to display the ipv4 address of this port

.

This ip address wil be used later on nginx firewall and postgresql configurations.

After these network configurations you should be able to acces to internet. 

Try to ping google.com


### <a name="_ppodazgw1low"></a> **Update Hosts File:**
Edit the hosts file to associate the server name with the new IP address:

bash

sudo nano /etc/hosts

Add or update the entry for your server:

192\.168.0.22    evershop.example.com

sudo systemctl restart networking

this  evershop.example.com will be used in nginx configuration later on.
## <a name="_i3grie4rpycf"></a>**Node.js**
To install EverShop, you must have[ ](https://nodejs.org/en/)[Node.js](https://nodejs.org/en/) version 14 or above.
## <a name="_mf2t57f8pbxz"></a>**NPM**
[NPM](https://www.npmjs.com/) version 7 or higher (which can be checked by running node -v).

It's recommended to install node.js and npm using nvm.
## <a name="_czoj2q8904o6"></a>**Database**
[PostgreSQL 13](https://www.postgresql.org/) or higher.

Note:Evershop setup creates database schema and tables automatically.That is why   Before migrating the database schema  sql file to   Postgresql dedicated vm  postgresql will be installed on evershopserver for a temporary time. During this time we will ensure that evershop app installed one evershop vm succesfully.

## <a name="_3buot2gsitjz"></a>**Install EverShop Using create-evershop-app command**
**npx create-evershop-app my-app**

The create-evershop-app command will create a new folder named my-app and install all of the dependencies for you.

You will have a my-app named folder which has node modules and package.json file .

Create a .env file by nano .env.

Paste the content below on this file.

**DB\_HOST=192.168.0.23 // instead of this enter localhost for now. Afterdb vm setup you ll come back here to add db vm ip address.**

**DB\_PORT=5432**

**DB\_USER=postgres**

**DB\_PASSWORD=changeme**

**DB\_NAME=evershop**

**DB\_SSLMODE=disable**

**Save and exit**

And then enter npm run dev command under the my-app project root folder.

Now Evershop should run on <http://localhost:3000>  

if you encounter a self certificated error  while running npm run:

The reason is you are using a self-signed cert and it is not in the trusted list.

You have some options:

1: Turn off the SSL and add environment "DB\_SSLMODE" = "disabled"

2: Set the environment "DB\_SSLMODE" = "no-verify"

### <a name="_bdcq73gsp4lp"></a>**Firewall Configuration**
Ensure that firewalls on both VMs allow traffic on the PostgreSQL port (default is 5432) and the EverShop application port.

sudo apt update && sudo apt upgrade -y

sudo apt-get install ufw -y

\# Allow SSH (replace 22 with your actual SSH port if it's not the default)

sudo ufw allow 22/tcp

\# Allow HTTP (port 80)

sudo ufw allow 80/tcp

\# Allow HTTPS (port 443) - if you're using SSL

sudo ufw allow 443/tcp

\# Deny all incoming connections by default

sudo ufw default deny incoming

\# Allow all outgoing connections by default

sudo ufw default allow outgoing


sudo ufw enable

Command may disrupt existing ssh connections. Proceed with operation (y|n)? y

Firewall is active and enabled on system startup

enter y.

On Evershop server vm firewall status should be like below.

root@EvershopServer:/home/vboxuser/my-app# sudo ufw verbose

Status: active

To                     	Action  	From

--                     	------  	----

22                     	ALLOW   	Anywhere             	 

5432                   	ALLOW   	Anywhere             	 

80                     	ALLOW   	Anywhere             	 

5432                   	ALLOW   	192.168.0.22  // replace with your evershop vm s ip  	 

22 (v6)                	ALLOW   	Anywhere (v6)        	 

5432 (v6)              	ALLOW   	Anywhere (v6)        	 

80 (v6)                	ALLOW   	Anywhere (v6)        	 

To test whether you can connect from the Evershop VM to the PostgreSQL service, you can use the `psql` command-line tool, which is a PostgreSQL client. Follow these steps:

1\. \*\*Install `psql` on the Evershop VM (if not already installed):\*\*

`	````bash

`	`sudo apt-get update

`	`sudo apt-get install postgresql-client

`	````

2\. \*\*Attempt to connect to the PostgreSQL service on the PostgreSQL VM:\*\*

`	````bash

`	`psql -h <PostgreSQL\_VM\_IP> -U <username> -d <database>

`	````

`	`Replace `<PostgreSQL\_VM\_IP>`, `<username>`, and `<database>` with your actual PostgreSQL VM's IP address, PostgreSQL username, and database name.

`	`Example:

`	````bash

`	`psql -h 192.168.0.22 -U your\_username -d your\_database

`	````

`	`If the connection is successful, you'll be prompted for the password. Enter the password for the PostgreSQL user when prompted.

3\. Verify that you can execute SQL queries:

`	`Once connected, you can run SQL queries to verify the connectivity. For example:

`	`**```sql**

`	`**SELECT \* FROM your\_table;**

`	`**```**

`	`Replace `your\_table` with an actual table in your PostgreSQL database.

If the connection is successful and you can execute queries, it means that the necessary firewall rules are in place and that you can connect from the Evershop VM to the PostgreSQL service.

Note: Ensure that you are using the correct PostgreSQL VM IP address, database username, and password in your connection command. If you face any issues, check the PostgreSQL logs for any connection-related errors on the PostgreSQL VM.


- To test the firewall configuration on your Debian 12 VM and ensure that it denies all incoming connections except for SSH, you can perform the following steps:
  - **Attempt to Connect to a Denied Port:**
    - Try connecting to a port that is not explicitly allowed. For example, if you haven't allowed port 80 for HTTP, try connecting to it:
      bash

telnet your\_server\_ip 80

- Replace your\_server\_ip with the actual IP address of your Debian 12 VM. If the firewall is properly configured, this connection attempt should fail.
- **Attempt to Connect to an Allowed Port (SSH):**
  - Confirm that you can still connect to the SSH port:
    bash

ssh user@your\_server\_ip

- Replace user with your actual username and your\_server\_ip with the VM's IP address. This connection should succeed.



1. **1. SSH Key-Based Authentication:**
   1. **Generate SSH Key Pair (on your local machine):**
      bash

ssh-keygen -t rsa -b 2048

1. Follow the prompts to generate the key pair. This will create ~/.ssh/id\_rsa (private key) and ~/.ssh/id\_rsa.pub (public key).
1. **Copy Public Key to Remote Server (PostgreSQL VM and EverShop VM):**
   bash

ssh-copy-id user@your\_postgresql\_vm\_ip

ssh-copy-id user@your\_evershop\_vm\_ip

1. Replace user with your username and your\_postgresql\_vm\_ip/your\_evershop\_vm\_ip with the respective IP addresses.

**Disable Password Authentication (Optional but Recommended):**

1. Edit the SSH server configuration file on both VMs:
   bash

sudo nano /etc/ssh/sshd\_config

Ensure the following lines are set:
perl
PasswordAuthentication no

ChallengeResponseAuthentication no

Restart the SSH service:
bash
sudo systemctl restart ssh
### <a name="_ygirda8b0wsc"></a>**Test SSH Access:**
Try connecting to each VM using SSH:

ssh user@your\_postgresql\_vm\_ip

ssh user@your\_evershop\_vm\_ip

# <a name="_hy22z88q66rh"></a>Database Server vm

## <a name="_r3riqgeqpbg8"></a>**Database**
[PostgreSQL 13](https://www.postgresql.org/) or higher.

1. **VM Setup**
   1. Set up a new Debian 12 VM.
   1. Install PostgreSQL on this VM.
   1. use same debian 12 iso

sudo apt update

sudo apt install postgresql postgresql-contrib

1. Networking

Follow same networking guidelines for evershop vm . DHCP will assign your vm s enp0s3 interface. You will use this ip address on configs.

1. **Configure PostgreSQL:**
   1. Adjust PostgreSQL configurations in the postgresql.conf file and pg\_hba.conf for security and networking settings.
   1. Ensure PostgreSQL is configured to accept remote connections if needed.
1. **Create Database and User:**
   1. Create a dedicated database for EverShop.
1. bash

sudo -u postgres psql

CREATE DATABASE evershopdb;

CREATE USER evershopuser WITH PASSWORD 'your\_password';

ALTER ROLE evershopuser SET client\_encoding TO 'utf8';

ALTER ROLE evershopuser SET default\_transaction\_isolation TO 'read committed';

ALTER ROLE evershopuser SET timezone TO 'UTC';

GRANT ALL PRIVILEGES ON DATABASE evershopdb TO evershopuser;

\q

# <a name="_tpmsv1rhf0ew"></a>Separate the PostgreSQL database to a dedicated Debian 12 VM

1. To separate the PostgreSQL database to a dedicated Debian 12 VM, you can follow these general steps. Keep in mind that the actual steps might vary based on your specific setup and requirements.
   **Assumptions:**
   1. You have a Debian 12 VM for EverShop.
   1. You have a dedicated Debian 12 VM for PostgreSQL.
   1. You have SSH access to both VMs.
   1. PostgreSQL is already installed on the dedicated PostgreSQL VM.
1. **Steps:**
   1. **Backup Current Database:**
      1. On the EverShop VM, use the pg\_dump command to create a backup of your current PostgreSQL database.
         bash

pg\_dump -U your\_username -d your\_database\_name -f backup.sql

1. **Transfer Backup to PostgreSQL VM:**
   1. Copy the generated backup.sql file to the dedicated PostgreSQL VM. You can use scp for this.
      bash

scp backup.sql your\_username@postgresql\_vm\_ip:/path/to/backup.sql

1. **Restore Database on PostgreSQL VM:**
   1. On the PostgreSQL VM, use the psql command to restore the database.
      bash

psql -U your\_username -d your\_database\_name -f /path/to/backup.sql

1. **Update EverShop Configuration:**
   1. On the EverShop VM, update the config file or any other relevant configuration file to point to the new PostgreSQL server. This typically involves changing the host, port, username, and password configurations.
1. **Adjust Firewall Rules:**
   1. On the PostgreSQL VM, adjust firewall rules to allow incoming connections from the EverShop VM on the PostgreSQL port.
1. **Update EverShop Database Configuration:**
   1. In your EverShop project, find the configuration files related to the database connection. Update these files with the new PostgreSQL server details.


To separate the PostgreSQL database to a dedicated Debian 12 VM, you can follow these general steps. Keep in mind that the actual steps might vary based on your specific setup and requirements.
### <a name="_lqe62lojb7ro"></a>**Assumptions:**
- You have a Debian 12 VM for EverShop.
- You have a dedicated Debian 12 VM for PostgreSQL.
- You have SSH access to both VMs.
- PostgreSQL is already installed on the dedicated PostgreSQL VM.
### <a name="_q9w6kqjqezem"></a>**Steps:**
1. **Backup Current Database:**
   1. On the EverShop VM, use the pg\_dump command to create a backup of your current PostgreSQL database.
      bash

pg\_dump -U your\_username -d your\_database\_name -f backup.sql

**Transfer Backup to PostgreSQL VM:**

- Copy the generated backup.sql file to the dedicated PostgreSQL VM. You can use scp for this.
  bash

scp backup.sql your\_username@postgresql\_vm\_ip:/path/to/backup.sql

**Restore Database on PostgreSQL VM:**

- On the PostgreSQL VM, use the psql command to restore the database.
  bash

psql -U your\_username -d your\_database\_name -f /path/to/backup.sql

1. **Update EverShop Configuration:**
   1. On the EverShop VM, update the config file or any other relevant configuration file to point to the new PostgreSQL server. This typically involves changing the host, port, username, and password configurations.
1. **Adjust Firewall Rules:**
   1. On the PostgreSQL VM, adjust firewall rules to allow incoming connections from the EverShop VM on the PostgreSQL port.
1. **Update EverShop Database Configuration:**
   1. In your EverShop project, find the configuration files related to the database connection. Update these files with the new PostgreSQL server details.
1. **Update EverShop User/Group:**
   1. Create a dedicated system user/group for the EverShop application on the EverShop VM if it doesn't already exist. Ensure that the EverShop daemon runs using this user/group.
1. **SSL Configuration:**
   1. Set up SSL for the PostgreSQL connection. You may need to generate SSL certificates and configure PostgreSQL to use them.
1. **Deny Unnecessary Connections:**
   1. Adjust firewall rules on both VMs to deny incoming connections that aren't essential for the services to run, except for SSH.
1. **Restart Services:**
   1. Restart both EverShop and PostgreSQL services to apply the changes.
1. **Testing:**
   1. Test the EverShop application to ensure it connects to the dedicated PostgreSQL VM successfully.
1. **Monitor Logs:**
   1. Monitor logs on both VMs to check for any errors or issues.
### <a name="_5ba0sub72ats"></a>**Notes:**
- Make sure to follow the security best practices for setting up PostgreSQL and securing your VMs.
- Always backup your data before making significant changes.
- This is a general guide, and the specific commands and steps might vary based on your setup and requirements. Refer to the documentation of the tools and applications you are using for more detailed information.

To edit the PostgreSQL configuration file (postgresql.conf) using the nano text editor, you can follow these steps. Please note that you need appropriate permissions to edit the file.

1. Open a terminal on your Debian 12 PostgreSQL VM.
1. Use the following command to open the postgresql.conf file in the nano text editor. Replace [your\_postgresql\_version] with your actual PostgreSQL version, for example, 13:
   bash

sudo nano /etc/postgresql/[your\_postgresql\_version]/main/postgresql.conf

If you are unsure about the PostgreSQL version, you can check the /etc/postgresql/ directory to see the available versions.

Navigate through the file using the arrow keys.

Make the necessary changes to the configuration parameters. You can use the Ctrl + O keyboard shortcut to write the changes, and then press Enter. To exit the editor, use the Ctrl + X keyboard shortcut.

Restart PostgreSQL to apply the changes:
bash

sudo service postgresql restart

Please be cautious while making changes to the postgresql.conf file, as incorrect configurations can lead to issues with your PostgreSQL database. Always take a backup of the file before making changes and refer to the PostgreSQL documentation for the appropriate settings.

Keep in mind that editing the postgresql.conf file might require superuser privileges, and the above commands assume that you have the necessary permissions. If you encounter any issues or if your setup is different, adjust the commands accordingly.


If you take an error message on below part


root@EvershopServer:/home/vboxuser# pg\_dump -U postgres -d evershop -f backup.sql pg\_dump: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL: Peer authentication failed for user "postgres"

The error message indicates that there's an issue with authentication when trying to connect to the PostgreSQL server using pg\_dump. The error "Peer authentication failed for user 'postgres'" suggests that the PostgreSQL server is configured to use "peer" authentication, and the user attempting to connect doesn't have the necessary privileges.

Here are steps you can take to resolve this issue:

The error message indicates that there's an issue with authentication when trying to connect to the PostgreSQL server using pg\_dump. The error "Peer authentication failed for user 'postgres'" suggests that the PostgreSQL server is configured to use "peer" authentication, and the user attempting to connect doesn't have the necessary privileges.

Here are steps you can take to resolve this issue:
### <a name="_5thn6yb2xeel"></a>**1. Use sudo:**
When running pg\_dump as the postgres user, try using sudo to elevate your privileges:

bash

sudo -u postgres pg\_dump -U postgres -d evershop -f backup.sql

### <a name="_9z0hzbp8rus5"></a>**2. Provide a Password:**
If the postgres user has a password, you might need to provide it when running pg\_dump. You can use the -W flag for this:

bash

pg\_dump -U postgres -d evershop -W -f backup.sql

After entering this command, you'll be prompted to enter the password for the postgres user.
### <a name="_2dgpzyfdudcc"></a>**3. Check pg\_hba.conf:**
Ensure that the pg\_hba.conf file (PostgreSQL Host-Based Authentication Configuration) allows the method you are using to connect.

1. Open the pg\_hba.conf file:
   bash

sudo nano /etc/postgresql/[your\_postgresql\_version]/main/pg\_hba.conf

Locate the lines that define the access rules. Ensure that there's an entry allowing your user to connect, for example:
conf

\# TYPE  DATABASE        USER            ADDRESS                 METHOD

local   all             postgres                                peer

You may need to add or modify an entry to match your authentication method (e.g., md5 for password-based authentication).

After modifying pg\_hba.conf, restart PostgreSQL:
bash

sudo service postgresql restart

### <a name="_mfurrcs2emjs"></a>**4. Check Password Authentication:**
Ensure that password authentication is enabled in the postgresql.conf file:

1. Open the postgresql.conf file:
   bash

sudo nano /etc/postgresql/[your\_postgresql\_version]/main/postgresql.conf

Look for the line password\_encryption and make sure it's set to on:
conf

password\_encryption = on

1. Save the file and restart PostgreSQL.

After making these changes, attempt the pg\_dump command again. If the issue persists, carefully review the PostgreSQL documentation and logs for further troubleshooting.

If you take error below

root@EvershopServer:/var/lib/postgresql# sudo scp backup.sql vboxuser@192.168.0.23:/var/lib/postgresql/ vboxuser@192.168.0.23's password: scp: dest open "/var/lib/postgresql/backup.sql": Permission denied scp: failed to upload file backup.sql to /var/lib/postgresql/ root@EvershopServer:/var/lib/postgresql#

The error "Permission denied" typically indicates that the user (vboxuser in this case) does not have the necessary permissions to write to the destination directory on the PostgreSQL VM.

Here are some steps to troubleshoot and resolve the issue:

1. **Check Permissions on the Destination Directory:** Ensure that the destination directory (/var/lib/postgresql/) on the PostgreSQL VM has the necessary write permissions for the user (vboxuser). You can use the following command to grant write permissions:
   bash

sudo chmod u+w /var/lib/postgresql/

**Check User Ownership of the Destination Directory:** Make sure that the user (vboxuser) has ownership or write permissions to the destination directory. You can use the following command to change ownership:
bash

sudo chown vboxuser:vboxuser /var/lib/postgresql/

**Verify SSH User's Permissions on the PostgreSQL VM:** Ensure that the vboxuser has the necessary permissions on the PostgreSQL VM. The user may need to be added to the appropriate group or granted the necessary permissions. You can add the user to the postgres group, for example:
bash

sudo usermod -aG postgres vboxuser

**Specify a Different Destination File Name:** If the destination file already exists and is owned by another user, you may encounter permission issues. You can try specifying a different file name for the destination, for example:
bash

sudo scp backup.sql vboxuser@192.168.0.23:/var/lib/postgresql/backup\_copy.sql

After making these adjustments, try running the scp command again. If you encounter further issues, check the system logs on both the source and destination machines for any additional error messages that might provide more details about the problem.


The your\_username in the psql command should be replaced with the PostgreSQL user who has the necessary permissions to access and restore the database. In many PostgreSQL installations, the default superuser is named postgres. Therefore, the command would look like this:

bash

psql -U postgres -d your\_database\_name -f /path/to/backup.sql

Make sure to replace your\_database\_name with the actual name of your PostgreSQL database and /path/to/backup.sql with the actual path to your backup file.

So, in your case, it would be the PostgreSQL user, which is often postgres. If you've created a different user with appropriate permissions, you would use that username instead.

If you are unsure about the PostgreSQL superuser or don't remember the usernames you've set up, you can list the existing users by connecting to the PostgreSQL database using the psql command without specifying a user. Here's an example:

bash

psql -d your\_database\_name

This command will prompt you for a password. After entering the correct password, you'll be in the PostgreSQL interactive shell, and you can run the following SQL query to list all users:

sql

\du

Look for the user with superuser privileges (typically the postgres user) or the user you've set up for administrative tasks, and use that username in the psql command.



1. To ensure that the database called "evershop" is accessible from the Evershop VM (IP address 192.168.0.10), you can take the following steps:
   1. **Check PostgreSQL Configuration on the PostgreSQL VM:**
      Ensure that PostgreSQL is configured to accept remote connections and is listening on the appropriate network interface. Open the PostgreSQL configuration file (postgresql.conf) and verify or modify the following settings:
      bash

sudo nano /etc/postgresql/[your\_postgresql\_version]/main/postgresql.conf

Look for the listen\_addresses parameter and set it to '\*' to allow connections from any address:
conf
listen\_addresses = '\*'

Save the file and restart PostgreSQL:
bash
sudo service postgresql restart

1. **Update pg\_hba.conf to Allow Connections:**
   Open the pg\_hba.conf file to configure the authentication settings for remote connections:
   bash

sudo nano /etc/postgresql/[your\_postgresql\_version]/main/pg\_hba.conf

Add a line to allow connections from the Evershop VM. For example, if the IP address of the Evershop VM is 192.168.0.10:
conf
host    evershop    all    192.168.0.10/32    md5

Save the file and restart PostgreSQL:
bash
sudo service postgresql restart

1. **Verify Connectivity from the Evershop VM:**
   On the Evershop VM, try connecting to the PostgreSQL VM using the psql command:
   bash

psql -U your\_username -h 192.168.0.23 -d evershop

1. Replace your\_username with the appropriate PostgreSQL username. If prompted, enter the password.
   If the connection is successful, you should be in the PostgreSQL interactive shell for the "evershop" database.
1. **Update Evershop Application Configuration:**
   In your Evershop application configuration, ensure that the database connection parameters are set correctly. This typically involves updating the config file or a similar configuration file in your Evershop application. The connection parameters should include the PostgreSQL server's IP address, database name, username, and password.
   json
1. `**DB\_HOST=192.168.0.23 // instead of this enter localhost for now. Afterdb vm setup you ll come back here to add db vm ip address.**

**DB\_PORT=5432**

**DB\_USER=postgres**

**DB\_PASSWORD=changeme**

**DB\_NAME=evershop**

**DB\_SSLMODE=disable**

**Save and exit**`

1. Replace "your\_postgres\_user" and "your\_postgres\_password" with the actual PostgreSQL username and password.
1. **Test the Evershop Application:**
   Start or restart your Evershop application and verify that it can connect to the PostgreSQL database on the PostgreSQL VM.


ok now You should be able to  access postgresql databe on postgresql dedicated vm from evershop vm. root@EvershopServer:/home/vboxuser/my-app# psql -U postgres -h 192.168.0.23 -d evershop

` `Password for user postgres: psql (15.5 (Debian 15.5-0+deb12u1)) SSL connection (protocol: TLSv1.3, cipher: TLS\_AES\_256\_GCM\_SHA384, compression: off) Type "help" for help. evershop=# what this putput tells about ssl connection

The output  provided indicates that the connection to the PostgreSQL database on your dedicated PostgreSQL VM is using SSL (Secure Sockets Layer). Let's break down the relevant parts of the output:

psql (15.5 (Debian 15.5-0+deb12u1))

SSL connection (protocol: TLSv1.3, cipher: TLS\_AES\_256\_GCM\_SHA384, compression: off)

Type "help" for help.

evershop=#

- **psql (15.5 (Debian 15.5-0+deb12u1)):** This part shows the version information of the psql command-line tool you are using. In this case, it's version 15.5 on a Debian-based system.
- **SSL connection (protocol: TLSv1.3, cipher: TLS\_AES\_256\_GCM\_SHA384, compression: off):** This part indicates that the connection to the PostgreSQL server is secured using SSL. SSL is a cryptographic protocol that ensures secure communication over a computer network. The details provided include:
  - **Protocol:** TLS version 1.3 is being used.
  - **Cipher:** TLS\_AES\_256\_GCM\_SHA384 is the encryption algorithm used for securing the connection.
  - **Compression:** Compression is turned off, which is a good practice for secure communication.
- **evershop=#:** This part shows that you are now connected to the "evershop" database. The prompt (evershop=#) indicates that you can now enter SQL commands for the "evershop" database.

The presence of "SSL connection" in the output indicates that your connection to the PostgreSQL server is encrypted using SSL/TLS, providing a secure communication channel between the client (Evershop VM) and the server (PostgreSQL VM). This is important for protecting sensitive data and credentials during transit.

` `**Install Process Manager (PM2):**

- Install PM2 globally:

npm install -g pm2

` `**Configure Evershop for Multiple Processes:**

Create a .sh file named start.sh  under the my-app project root folder where you run npm run dev command. Now instead of using npm run dev directly on this bash script you need to create multiple instances on different ports of localhost.

Bash script should be like below.

#!/bin/bash

export PATH=/usr/local/bin:/usr/bin:/bin:/root/.nvm/versions/node/v21.3.0/bin

\# Navigate to the directory where app is located

cd /home/vboxuser/my-app

\# Log file

LOG\_FILE=/home/vboxuser/my-app/evershop.log


/root/.nvm/versions/node/v21.3.0/bin/pm2 start "npm run dev" --name evershop-1  > /var>

/root/.nvm/versions/node/v21.3.0/bin/pm2 start "npm run dev" --name evershop-2



# <a name="_n1m08hxt9ui4"></a>NGINX
**Configure Nginx:**

- Ensure that Nginx is installed on your Debian 12 VM.
- Create an Nginx configuration file for your EverShop application. This file typically goes in the /etc/nginx/sites-available/ directory.
  Example configuration (/etc/nginx/sites-available/evershop.conf):

your evershop.conf should be like below.

**upstream evershop\_backend {**

`	`**server 127.0.0.1:3000;**

`	`**server 127.0.0.1:3001;**

`	`**# Add more servers if needed**

**}**

**server {**

`	`**listen 80;**

`	`**server\_name 192.168.0.22  evershop.example.com;**

`	`**location / {**

`    	`**proxy\_pass http://evershop\_backend;**

`    	`**proxy\_http\_version 1.1;**

`    	`**proxy\_set\_header Upgrade $http\_upgrade;**

`    	`**proxy\_set\_header Connection 'upgrade';**

`    	`**proxy\_set\_header Host $host;**

`    	`**proxy\_cache\_bypass $http\_upgrade;**

`	`**}**

**}**




The Nginx configuration  provided acts as a reverse proxy for your Node.js application. Let's break down the key elements:

1. **listen 80;**: This line specifies that Nginx should listen for incoming connections on port 80, which is the default HTTP port.
1. **server\_name your\_domain.com www.your\_domain.com;**: Replace your\_domain.com with your actual domain name. This line defines the server name or names to match against the host header in the HTTP request. Nginx will process requests with these domain names.
1. **location / { ... }**: This block defines how Nginx should handle requests for the specified location (in this case, the root / path).
   1. **proxy\_pass http://localhost:3000;**: This line is crucial. It tells Nginx to forward requests to your Node.js application, which is running on http://localhost:3000. This is the address and port where your Node.js application is expected to be running.
   1. **proxy\_http\_version 1.1;**: This line specifies the HTTP version to use for communication between Nginx and your Node.js application.
   1. **proxy\_set\_header Upgrade $http\_upgrade;**: These lines are used to pass along certain HTTP headers that are required for features like WebSockets to work properly.
   1. **proxy\_set\_header Connection 'upgrade';**: Similar to the above, this header is used to support the WebSocket protocol.
   1. **proxy\_set\_header Host $host;**: This line sets the Host header of the request to the original Host header. It helps in cases where your Node.js application needs to know the original host requested by the client.
   1. **proxy\_cache\_bypass $http\_upgrade;**: This line ensures that the cache is bypassed when upgrading a connection. It's necessary for WebSocket connections.

In summary, when a user accesses your website through a web browser and makes a request, Nginx receives the request on port 80. The location / block then forwards this request to your Node.js application running on localhost:3000. This setup allows Nginx to act as a middleman, handling certain aspects of the HTTP protocol and forwarding the rest to your Node.js application for processing.

Create a symbolic link to this configuration file in the /etc/nginx/sites-enabled/ directory:
bash
sudo ln -s /etc/nginx/sites-available/evershop.conf /etc/nginx/sites-enabled/

- Disable the default Nginx configuration:
  bash

sudo unlink /etc/nginx/sites-enabled/default

- Restart Nginx:

sudo systemctl restart nginx

#
# <a name="_hgu0pwa81yn7"></a><a name="_j8zq4e1uedxc"></a>Run EverShop Application:
Execute your start.sh script to start multiple instances of the EverShop application:

cd /my-app

chmod +x start.sh  # Ensure the script is executable

./start.sh

Now, you should be able to access your EverShop application through Nginx using your domain name or IP address.

With these steps, you've deployed your EverShop application on your dedicated VM, configured load balancing using Nginx, and made your application accessible through Nginx. Adjust the configuration files according to your specific setup and requirements.

**Now You can  access  evershop from  any machine on your local network including your main os.**

**Note:**

Check EverShop Instances:

- Confirm that your EverShop instances are running and listening on the specified ports (3000 and 3001).

pm2 list





**Performing a load and performance test on your server can help you determine its capacity and identify potential bottlenecks. Here's a general guide on how to conduct such tests:**
### <a name="_2g706wam974b"></a>**Load Testing:**

` `Make a load and performance test on your server. Determine how many

concurrent clients your server can handle.

**To determine how many concurrent clients your server can handle, you typically perform load testing. The Apache Benchmark tool (ab) that you've been using is one such tool. It allows you to simulate multiple users accessing your server simultaneously.**

**The key metrics you might look at are:**

1. **Requests per second (RPS): This indicates how many requests your server is handling per second.**
1. **Time per request: The average time it takes to serve a request. This can help you understand the responsiveness of your server.**
1. **Concurrency: The number of multiple requests being processed simultaneously.**

**Apache Benchmark (ab):**

1. **Apache Benchmark is a command-line tool that comes with the Apache HTTP server. It's simple to use for basic load testing.**
1. **Example command:**


**ab -n 1000 -c 100 http://evershop.example.com/**

**when i do the test with 5 concurerent user it gives result below. root@EvershopServer:/etc/nginx/sites-enabled# ab -n 1000 -c 5** 

**http://evershop.example.com/ This is ApacheBench, Version 2.3 <$Revision: 1903618 $> Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/ Licensed to The Apache Software Foundation, http://www.apache.org/ Benchmarking** 

**evershop.example.com (be patient)** 

**Completed 100 requests**

` `**Completed 200 requests** 

**Completed 300 requests**

` `**Completed 400 requests**

` `**Completed 500 requests**

` `**Completed 600 requests**

` `**Completed 700 requests**

` `**Completed 800 requests**

` `**Completed 900 requests**

` `**Completed 1000 requests**

` `**Finished 1000 requests**

` `**Server Software: nginx/1.22.1** 

**Server Hostname: evershop.example.com**

` `**Server Port: 80 Document Path: / Document Length: 4408 bytes**

` `**Concurrency Level: 5 Time taken for tests: 152.397 seconds**

` `**Complete requests: 1000 Failed requests: 0**

` `**Total transferred: 4791662 bytes HTML transferred: 4408000 bytes Requests per second: 6.56 [#/sec] (mean)** 

**Time per request: 761.986 [ms] (mean)**

` `**Time per request: 152.397 [ms] (mean, across all concurrent requests) Transfer rate: 30.71 [Kbytes/sec] received Connection Times (ms) min mean[+/-sd] median max Connect: 0 0 1.4 0 12 Processing: 244 759 246.6 728 1993**

` `**Waiting: 241 707 231.0 677 1876 Total: 248 759 246.6 728 1998 Percentage of the requests served within a certain time (ms) 50% 728 66% 845 75% 903 80% 946 90% 1073 95% 1190 98% 1320 99% 1523 100% 1998 (longest request)** 

**so what is the maximumm concurrent users that this server can handle without a very serious bottleneck**

**From the provided Apache Benchmark (ab) results with a concurrency level of 5:**

- **Requests per second (RPS): 6.56 [#/sec] (mean)**
- **Concurrency Level: 5**
- **Time per request: 761.986 [ms] (mean)**

**The key metric for estimating the maximum concurrent users the server can handle is the Requests per second (RPS). In this case, the server is handling approximately 6.56 requests per second on average.**

**To estimate the maximum concurrent users, you can use the formula:**

**Maximum Concurrent Users=Requests per second / Concurrency Level**

**In this case:**

**Maximum Concurrent Users=6.56 / 6 = 1.312**

**Currently Evershop have 2 cpu.**

**Probably I need to have  more cpu to create  more instances of node.js on different ports so that the number of max concurrent users can be increased.**
### <a name="_xuu0ymqnqjkb"></a>**Additional Questions:**
**1. Process Manager Choice:**

- **I used PM2 as the process manager. PM2 is easy to use, supports load balancing, and provides features like automatic restarts. Also it has a web app that shows the real time metrics of the different instances of the app. Another reason why I have used pm2 is it s widely used. It is easy to troubleshoot using stackoverflow and pm2 docs.**

**2. Automatically Update Product Prices:**

**Integrating real-time currency updates into a PostgreSQL database for an Evershop application involves updating the product prices based on the latest exchange rates. Here's a logical explanation of how this integration can be achieved:**

**1. Database Schema:**

- **Ensure your PostgreSQL database has a table storing product information, including the product price and currency.**

**2. Currency Exchange API Integration:**

- **Integrate with a reliable currency exchange API to fetch the latest exchange rates. This can be done using Node.js server-side code.**

**3. Scheduled Job:**

- **Implement a scheduled job in your Node.js application (using a library like node-cron) to periodically fetch the latest exchange rates from the API.**

**4. Update Product Prices:**

- **Upon fetching new exchange rates, calculate the updated prices for all products in the application.**
- **Update the product prices in the PostgreSQL database based on the latest exchange rates.**

**5. Notify React Frontend:**

- **Use WebSocket communication between the Node.js server and the React frontend to notify the frontend whenever there's an update in exchange rates or product prices.**
- **Emit the updated product prices to the React frontend using the WebSocket connection.**

**6. React Frontend:**

- **Listen for updates on product prices through the WebSocket connection.**
- **Upon receiving an update, update the React state with the new product prices.**
- **Reflect the updated prices in the Evershop UI.**

**Summary: This integration involves periodic updates to product prices in the PostgreSQL database based on real-time exchange rates. The WebSocket communication ensures that React frontend gets instant notifications about these updates, providing a seamless and real-time experience for users of the Evershop application.**

### <a name="_6rcll32daeki"></a>**High Availability Setup:**
**For setting up a high-availability Evershop installation, consider the following:**

- **Load Balancing: Use a load balancer to distribute incoming traffic across multiple instances of the Evershop application. This ensures even distribution of requests and better utilization of resources.**
- **Database Replication: Implement database replication for redundancy and fault tolerance. This involves maintaining a copy of the database on a separate server to ensure data availability in case of a failure.**
- **Backup and Restore Procedures: Regularly backup the application data and configurations. Have well-defined procedures for restoring the application in case of failures.**
- **Scaling: Design the  infrastructure to easily scale horizontally by adding more servers or resources as the demand increases.**
- **Monitoring and Alerts: Implement robust monitoring and alerting systems to promptly identify and address any issues. Tools like Prometheusor or Grafana or Elastic APM can be useful. I would use Elastic APM if migration from a monolith system to a microservice based system is necessary. It has a service map that displays anomalies for each microservices clearly.**
- **Security Measures: Implement security best practices, including firewalls, regular security audits, and proper access controls.**

**By addressing these aspects, we can enhance the reliability and availability of the Evershop application, providing a seamless experience for users.**



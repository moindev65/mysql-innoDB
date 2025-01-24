Installation mysql cluster
Add MySQL APT Repository
Note: before installing mysql if you need to set lowercase-table-names value you need to do below command:
sudo debconf-set-selections <<< "mysql-server mysql-server/lowercase-table-names select Enabled"
add the MySQL APT repository to your system’s software repository list
wget https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb
Install the downloaded MySQL apt repository
sudo dpkg -i mysql-apt-config_0.8.32-1_all.deb
For download without any issue first, you create a MySQL repo in Nexus. By the below image do it then update the MySQL repo list.
According to the image setting.
before install mysql if we want to have option
Then update all repo URLs in the file located in the path /etc/mysql/mysql.conf.d/mysqld.conf. In addition please replace http://repo.
mysql.com/apt/ubuntu/ by nexus repo address.
Update the system package.
sudo apt-get update
Install mysql server
apt-get install mysql-server
Create User
Login into Database with user name and password
mysql version
During the installation there are a couple of questions that show the version of MySQL please select mysql-cluster if exists.
mysql -u root -p
Create the user.
CREATE USER 'mno'@'%' IDENTIFIED BY 'mno@123';
Grant Permission to all databases to connect remotely with admin permission.
GRANT ALL PRIVILEGES ON *.* TO 'mno'@'%' WITH GRANT OPTION;
Perform flush-privileges operation to tell the server to reload the grant tables.
FLUSH PRIVILEGES;
Open /etc/mysql/mysql.conf.d/mysqld.conf and add bind address
bind-address = 0.0.0.0
To take effect, restart MySQL Service.
sudo systemctl restart mysql
MySQL InnoDB Cluster Setup
To install mysql-shell you install it by binary package please go to the below link and the version you selected to download it. Also,
you should download mysqlrouter from the below link.
https://dev.mysql.com/downloads/
Before creating a cluster, we have to configure the host for InnoDB cluster usage. Run the below command on 3 hosts.
On mysql-01 Host
First login to MySQL Shell on MYSQL-IDC-01 instance:
sudo dpkg -i mysql-apt-config_0.8.32-1_all.deb
mysqlsh
Connect to first instance,
shell.connect('mno@mysql-01:3306')
Now create a cluster assigning the return value to a variable.
var cluster =dba.createCluster('MnoCluster')
Next, add MYSQL02-02 and MYSQL02-02 to ProdCluster. Run the below commands on MYSQL-01
cluster.addInstance('mno@mysql-02:3306')
cluster.addInstance('mno@mysql-03:3306')
Check the Cluster status:
cluster.status()
InnoDB Cluster Administration
1. Get information about the structure of the InnoDB cluster
Cluster.describe()
2. To check Primary Master in Group Replication. By Default Group Replication runs on Single Primary Node.
mysql> SELECT member_host, member_port, member_state, member_role FROM performance_schema.
replication_group_members;
Install Mysqlrouter
First, you show download binary is from this website https://dev.mysql.com/downloads/ after that install it. Moreover, mv downloads
binary to this path /usr/bin/ and applies the command below on to make it executable.
chomod +x mysqlrouter
Deploying /Bootstrapping MySQL Router
MySQL Router can be deployed using bootstrapping on application instance using the below command, which --user is the owner of
mysqlrouter service. and -d will create a folder that contain start.sh stop.sh script
mysqlrouter --bootstrap mno@mysql-01:3306 --user=root -d mno
then we have to start MySQL Router using below command,
cd /usr/bin/mno && ./start.sh
To check it’s started and listening / accepting connections on port 6446.
netstat -an | grep 6446
Connect to Database Cluster
After successfully configuring of MySQL Router, the below command is connected to MySQL Servers/Clusters from the application instance,
mysql -h 127.0.0.1 --port 6446 -u fosstechnix -p
Note:
After installing MySQL if you want to set some values for belong to the MYSQL setting you should put the below values in this file.(/etc
/mysql/my.cnf)
[mysqld]
sql-mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"
then restart mysql service
systemctl restart mysql.service
Ref:
https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_lower_case_table_names
https://www.fosstechnix.com/mysql-innodb-cluster-setup/
https://bobcares.com/blog/mysql-set-sql_mode-in-my-cnf/

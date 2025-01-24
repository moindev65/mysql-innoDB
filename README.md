# MySQL InnoDB Cluster Setup

## Introduction
MySQL Router is a tiny middleware that provides routing between applications and back-end MySQL Servers. It ensures high availability and scalability by directing database traffic to connected MySQL servers.

In this setup, we deploy:
- 3 MySQL backend nodes
- 1 MySQL Router node

## Prerequisites

### Server IPs and Hostnames

| IP Address     | Hostname      | Description               |
|---------------|--------------|---------------------------|
| 172.40.70.16 | mysql-01     | Primary/Master DB Server |
| 172.40.70.17 | mysql-02     | Read Only DB Server 1    |
| 172.40.70.18 | mysql-03     | Read Only DB Server 2    |
| 172.40.70.18 | mysqlrouter  | MySQL Router             |

## Installation of MySQL Cluster

### Step 1: Add MySQL APT Repository
Before installing MySQL, set the `lowercase-table-names` value if required:

```sh
sudo debconf-set-selections <<< "mysql-server mysql-server/lowercase-table-names select Enabled"
```

Add MySQL APT repository:

```sh
wget https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.32-1_all.deb
```

### Step 2: Update and Install MySQL Server

Update the system package list:

```sh
sudo apt-get update
```

Install MySQL Server:

```sh
apt-get install mysql-server
```

### Step 3: Create a MySQL User

Login into MySQL:

```sh
mysql -u root -p
```

Create a user:

```sql
CREATE USER 'mno'@'%' IDENTIFIED BY 'mno@123';
GRANT ALL PRIVILEGES ON *.* TO 'mno'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Modify `mysqld.conf`:

```sh
sudo nano /etc/mysql/mysql.conf.d/mysqld.conf
```

Add:

```ini
bind-address = 0.0.0.0
```

Restart MySQL:

```sh
sudo systemctl restart mysql
```

## MySQL InnoDB Cluster Setup

### Install MySQL Shell

Download and install MySQL Shell:

```sh
wget https://dev.mysql.com/downloads/
sudo dpkg -i mysql-apt-config_0.8.32-1_all.deb
```

Login to MySQL Shell:

```sh
mysqlsh
```

Connect to the first MySQL instance:

```sh
shell.connect('mno@mysql-01:3306')
```

Create a cluster:

```sh
var cluster = dba.createCluster('MnoCluster')
```

Add MySQL instances:

```sh
cluster.addInstance('mno@mysql-02:3306')
cluster.addInstance('mno@mysql-03:3306')
```

Check cluster status:

```sh
cluster.status()
```

## InnoDB Cluster Administration

### Check Cluster Structure

```sh
cluster.describe()
```

### Verify Primary Master in Group Replication

```sql
SELECT member_host, member_port, member_state, member_role FROM performance_schema.replication_group_members;
```

## Install MySQL Router

Download and install MySQL Router:

```sh
wget https://dev.mysql.com/downloads/
chmod +x mysqlrouter
mv mysqlrouter /usr/bin/
```

### Deploy MySQL Router

Use bootstrapping:

```sh
mysqlrouter --bootstrap mno@mysql-01:3306 --user=root -d mno
```

Start MySQL Router:

```sh
cd /usr/bin/mno && ./start.sh
```

Check if MySQL Router is listening on port `6446`:

```sh
netstat -an | grep 6446
```

## Connect to the Database Cluster

```sh
mysql -h 127.0.0.1 --port 6446 -u fosstechnix -p
```

## Additional Configuration

Modify MySQL settings:

```sh
sudo nano /etc/mysql/my.cnf
```

Add:

```ini
[mysqld]
sql-mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"
```

Restart MySQL:

```sh
systemctl restart mysql.service
```

## References

- [MySQL Router Documentation](https://dev.mysql.com/doc/mysql-router/8.0/en/)
- [MySQL Cluster Setup Guide](https://www.fosstechnix.com/mysql-innodb-cluster-setup/)
- [MySQL Configuration Guide](https://bobcares.com/blog/mysql-set-sql_mode-in-my-cnf/)

# PostgreSQL Database Cluster Deployment

## About the Project

This project focuses on deploying and configuring a fault-tolerant PostgreSQL 14 cluster in **Active/Standby** mode on Linux.  
As part of the task, three virtual machines were prepared:

- **primary PostgreSQL server**;
- **standby PostgreSQL server**;
- **client machine** for connecting to the cluster and verifying database operation.

The main goal of the project was to configure replication between the servers, create a working database, verify client access, and test failover by promoting the standby server when the primary becomes unavailable.

## Project Goal

The project included the following tasks:

- install PostgreSQL 14 on the primary and standby servers;
- configure the primary server for replication;
- create a replication user;
- create an application user and a database;
- import the schema and data into the database;
- configure the standby server in standby mode;
- copy the data from the primary server using `pg_basebackup`;
- verify connectivity from the client VM;
- perform a failover test and make sure the standby server can take over as the primary.

## Architecture

Three virtual machines were used in this project:

1. **Primary server** — the main PostgreSQL server.
2. **Standby server** — the backup PostgreSQL server.
3. **Client VM** — a client machine with `postgresql-client-14` installed.

Workflow overview:

- the primary server accepts changes;
- the standby server receives and applies WAL records;
- the client machine connects to the servers to verify availability and data consistency.

## Technologies Used

- Linux
- PostgreSQL 14
- PostgreSQL streaming replication
- `pg_basebackup`
- `psql`
- SQL
- network access configuration with `pg_hba.conf`

## Completed Steps

### 1. PostgreSQL and Client Installation

PostgreSQL was installed on both database servers and prepared for configuration.  
The following package was installed on the client VM:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install postgresql-client-14
```

### 2. Primary Server Configuration

The following steps were completed on the primary server:

- created the `repuser` role with replication privileges and a password;
- set `wal_level` to `replica`;
- configured authentication in `pg_hba.conf` for the standby server to connect as `repuser`;
- created the `sysadmin` user with a password for client connections;
- allowed connections to the server for the `sysadmin` role.

### 3. Database Creation and Data Import

The following database was created on the primary server:

- `sysadmin_db`

The `sysadmin` user was granted full privileges on this database.  
After that, the schema and data were downloaded and imported:

```bash
sudo -u postgres wget https://sysadmin.education-services.ru/downloads/schema.sql
sudo -u postgres wget https://sysadmin.education-services.ru/downloads/data.sql
sudo -u postgres psql -d sysadmin_db < schema.sql
sudo -u postgres psql -d sysadmin_db < data.sql
```

### 4. Standby Server Configuration

On the standby server:

- the `hot_standby` parameter was enabled;
- the data from the primary server was copied using `pg_basebackup`;
- access for the `sysadmin` user was configured.

This allowed the standby server to run as a replica of the primary node.

### 5. Replication Check

After the cluster was configured, a self-check was performed:

- connectivity from the client VM to both servers was verified;
- the students list was imported on the primary server;
- the following query was executed:

```sql
select count(*) from студенты;
```

The record count matched on both servers, which confirmed that replication was working correctly.

### 6. Standby Promotion

To test failover:

- the primary server was shut down;
- the standby server was promoted to primary:

```bash
pg_ctlcluster 14 main promote
```

After that, a test record was added to the table, and the full name was updated to the student's own name.  
This confirmed that after the failure of the main node, the standby server could continue operating as the primary server.

## Result Verification

### Primary Server

```bash
grep repuser /etc/postgresql/14/main/pg_hba.conf
grep wal_level /etc/postgresql/14/main/postgresql.conf
grep sysadmin /etc/postgresql/14/main/pg_hba.conf
grep listen /etc/postgresql/14/main/postgresql.conf
sudo -u postgres psql -d sysadmin_db -c "select * from студенты;"
```

### Standby Server

```bash
grep hot_standby /etc/postgresql/14/main/postgresql.conf
grep sysadmin /etc/postgresql/14/main/pg_hba.conf
sudo -u postgres psql -d sysadmin_db -c "select * from студенты;"
```

### Final Check

Expected result after promoting the standby server:

```bash
sudo -u postgres psql -d sysadmin_db -c "select count(*) from студенты;"
```

Output:

```text
 count
-------
  9999
(1 row)
```

And verification of the test record:

```bash
sudo -u postgres psql -d sysadmin_db -c "select * from студенты where id=3;"
```

Output:

```text
 id |            фио             | курс
----+----------------------------+------
  3 | YOUR FULL NAME             |    3
(1 row)
```

## Skills Practiced in This Project

This project helped reinforce the following practical skills:

- PostgreSQL 14 installation and basic configuration;
- creating roles and managing privileges;
- configuring `pg_hba.conf` and `postgresql.conf`;
- creating a database and importing SQL scripts;
- configuring PostgreSQL streaming replication;
- preparing a standby server with `pg_basebackup`;
- checking replication between cluster nodes;
- promoting a standby node to primary in a failover scenario.

## Summary

As a result of this project, a PostgreSQL cluster in **Active/Standby** configuration was successfully deployed.  
Replication, client access, data import, and failover behavior were all configured and verified.

This project demonstrates practical skills in PostgreSQL administration, replication setup, and basic high-availability database deployment in a Linux environment.

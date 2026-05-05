# Project Summary

This project implements PostgreSQL logical replication from a Patroni HA cluster to an external Change Data Capture **(CDC)*** database. It facilitates real-time data synchronization, enabling efficient data management and analytics by replicating changes from specified tables in the cluster to the CDC database.

## Project Module Description
The project consists of several scripts and SQL files that automate the setup and management of logical replication. Key functionalities include:

Setting up replication configurations.
Managing the addition and removal of tables from replication.
Monitoring replication status and health.
Creating necessary database schemas and tables in the CDC database.

## Directory Tree

├── README.md                          # Comprehensive usage guide  
├── add_table_to_replication.sh        # Script to add a table to replication  
├── cdc_database_setup.sql             # SQL script to set up the CDC database  
├── check_replication_status.sh         # Script to check replication status  
├── monitoring_queries.sql              # SQL queries for monitoring replication  
├── patroni_replication_config.sql      # SQL configuration for Patroni  
├── remove_table_from_replication.sh    # Script to remove a table from replication  
├── replication_config.conf             # Configuration file for replication setup  
└── setup_logical_replication.sh        # Main script to set up logical replication  

## File Description Inventory

***README.md:*** Contains detailed instructions on installation, configuration, and usage.

***add_table_to_replication.sh:*** Script for adding new tables to the replication setup.

***cdc_database_setup.sql:*** SQL commands to create the CDC database and its schemas.

***check_replication_status.sh:*** Script to verify the current status of replication.

****monitoring_queries.sql:*** SQL queries designed for monitoring the replication process.

***patroni_replication_config.sql:*** SQL commands to configure replication settings in Patroni.

***remove_table_from_replication.sh:*** Script for removing tables from the replication setup.

***replication_config.conf:*** Configuration settings for the replication environment.

***setup_logical_replication.sh:*** Script that executes the overall setup for logical replication.

## Technology Stack
- PostgreSQL (version 10 or later)
- Patroni for high availability 
- Bash scripting for automation 
- SQL for database interactions


## Usage
### Installation Steps

1. Edit the configuration file:
```bash
nano replication_config.conf
```
Fill in the necessary details for source and target databases.

2. Prepare the Patroni cluster:
```bash
patronictl -c /etc/patroni/patroni.yml edit-config <cluster-name>
```
Add required parameters and restart the cluster:
```bash
patronictl -c /etc/patroni/patroni.yml restart <cluster-name>
```

Set up the CDC database:
```bash
psql -U postgres -f cdc_database_setup.sql
```

Execute the replication setup:
```bash
chmod +x *.sh
./setup_logical_replication.sh
```

Check the replication status:
```bash
./check_replication_status.sh
```

Adding a New Table to Replication
```bash
./add_table_to_replication.sh <source_db> <source_schema> <source_table> <target_schema> [target_table]
```

Removing a Table from Replication
```bash
./remove_table_from_replication.sh <source_db> <source_schema> <source_table>
```

Monitoring Replication
```bash
./check_replication_status.sh
psql -U postgres -d cdc -c "SELECT * FROM replication_status;"
```

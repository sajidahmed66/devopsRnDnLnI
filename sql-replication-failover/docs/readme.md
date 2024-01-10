#### Add MariaDB repo

```
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
```

1. Install mariadb related packages

   ```
       yum install mariadb
   ```

   ```
       yum install MariaDB
   ```

2. Start mariaDB

   ```
       systemctl start mariadb
   ```

3. Configure mariaDB

   Now we have to configure mariadb properly 1st run secure installation

   ```
       mariadb-secure-installation
   ```

4. Update node/server config file

   ```
   vi /etc/my.cnf.d/server.cnf
   ```

   ```bash
   #
   # These groups are read by MariaDB server.
   # Use it for options that only the server (but not clients) should see
   #

   # this is read by the standalone daemon and embedded servers
   [server]

   # This group is only read by MariaDB servers, not by MySQL.# If you use the same .cnf file for MySQL and MariaDB,
   # you can put MariaDB-only options here
   [mariadb]

   # This group is read by both MariaDB and MySQL servers
   [mysqld]

   #
   # Galera-related settings
   #
   [galera]
   # Mandatory settings
   wsrep_on=ON # this turns on the read write replica sets
   wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
   wsrep_cluster_name="test_cluster"
   # here add the ip of all the nodes
   wsrep_cluster_address="gcomm://192.168.37.128,192.168.37.129,192.168.37.130"
   binlog_format=row

   wsrep_sst_method=rsync

   #Node configuration add the ip of the corresponding node and name of node i.e mariadb1, mariadb2 ..etc
   wsrep_node_address="192.168.37.128"
   wsrep_node_name="mariadb1"


   #binlog_format=row
   #default_storage_engine=InnoDB
   #innodb_autoinc_lock_mode=2
   #
   # Allow server to accept connections on all interfaces.
   #
   #bind-address=0.0.0.0
   #
   # Optional setting
   #wsrep_slave_threads=1
   #innodb_flush_log_at_trx_commit=0



   # this is only for embedded server
   [embedded]

   # This group is only read by MariaDB servers, not by MySQL.# If you use the same .cnf file for MySQL and MariaDB,
   # you can
   [mariadb]

   ```

   and do the same for rest of the nodes

5. stop mariadb

   ```
   systemctl stop mariadb
   ```

   do this for rest of the mariadb node/server

6. Setup galera cluster

   ```
   galera_new_cluster
   ```

   Only in main node

   [Optional] - Make sure you've open needed ports or can disable firewall and open all ports

   ```
   systemctl stop firewalld.service
   ```

disable firewall NOTE: we need 3306 open to be opend/accessable

7.  Check galera_cluster_node_size
    enter mariadb by

    ````
    mariadb -uroot -ppassword

            show global status like 'wsrep_cluster_size';
        ```

    You should see total number of node count
    ````

8.  Setup maxscale - we can setup this on root/main node or separate instance/vm

    ```bash
    # if new node then setup mariadb repo first
    curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash

    yum install maxscale
    ```

9.  Create database user for maxscale to access
    ssh into galera cluster node and enter mariadb shell

    ```sql
    maxscale db*user = create user 'maxscale'@'%' identified by '123';

    grant select on mysql.* to 'maxscale'@'%';
    GRANT SHOW DATABASES, BINLOG ADMIN, READ ONLY ADMIN, RELOAD,
      -> REPLICATION MASTER ADMIN, REPLICATION SLAVE ADMIN,
      -> REPLICATION SLAVE, SLAVE MONITOR ON _._ TO 'maxscale'@'%';

    flush privileges;
    ```

10. Configure maxscale

    ```bash
    vi /etc/maxscale.cnf

    #write the follwing into the file

    [maxscale]
    threads=auto
    admin_host            = 0.0.0.0
    admin_port            = 8989
    admin_secure_gui=false

    [mariadb-01]
    type=server
    address=192.168.18.88 #ip of the node-1
    port=3306
    protocol=MariaDBBackend

    [mariadb-02]
    type=server
    address=192.168.18.89 #ip of the node-1
    port=3306
    protocol=MariaDBBackend

    [mariadb-03]
    type=server
    address=192.168.18.90 #ip of the node-1
    port=3306
    protocol=MariaDBBackend

    [MariaDB-Monitor]
    type=monitor
    module=galeramon
    servers=mariadb-01,mariadb-02,mariadb-03
    user=maxscale
    password=123

    [Galera-Service]
    type=service
    router=readwritesplit
    servers=mariadb-01,mariadb-02,mariadb-03
    user=maxscale
    password=123

    [Galera-Listener]
    type=listener
    service=Galera-Service
    protocol=MariaDBClient
    port=3306
    ```

    ```
    systemctl enable --now maxscale
    ```

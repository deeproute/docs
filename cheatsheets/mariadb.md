
### Make sure you connect to the right mariadb socket:
```
$ k -n mariadb-ns exec -it mariadb-1 -- bash

appuser@mariadb-1:/$ ln -s /opt/bitnami/mariadb/tmp/mysql.sock /tmp/mysql.sock

OR

mysql --socket=/opt/bitnami/mariadb/tmp/mysql.sock -uroot -p


appuser@mariadb-1:/$ mysql -umariabackup -p"1234566"

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 13
Server version: 10.5.8-5-MariaDB-enterprise-log MariaDB Enterprise Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]>

```


### Switch Master
```
maxctrl call command mariadbmon switchover mariadb-mon dbserv1
```

### Put on Maintenance
```
maxctrl set server dbserv1 maintenance
```

### Destroy server (destroy doesn't delete data)

```
maxctrl unlink service mariadb-service dbserv0
maxctrl unlink monitor mariadb-monitor dbserv0
maxctrl destroy server dbserv0

OR

maxctrl destroy server dbserv0 --force
```

### Create Server

```
maxctrl create server dbserv2 mariadb-dev-2.unixapp-sandbox.svc.cluster.local 3306
maxctrl link service mariadb-service dbserv2
maxctrl link monitor mariadb-monitor dbserv2

```

### Reset Replication

https://jira.mariadb.org/browse/MXS-2651?focusedCommentId=135395&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-135395

You can run the maxctrl call command mariadbmon reset-replication <monitor-name> <master-name> to initialize the cluster properly. Also make sure you configure replication with MASTER_USE_GTID.

```
maxctrl call command mariadbmon reset-replication mariadb-mon dbserv0
```

### Check MariaDB configured variables
```
SHOW STATUS WHERE `variable_name` LIKE '%Threads%';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| Delayed_insert_threads  | 0     |
| Slow_launch_threads     | 0     |
| Threadpool_idle_threads | 0     |
| Threadpool_threads      | 0     |
| Threads_cached          | 0     |
| Threads_connected       | 3     |
| Threads_created         | 57    |
| Threads_running         | 1     |
+-------------------------+-------+
```

### See users configurations

```
MariaDB [(none)]> select user, host, password, plugin from mysql.user;
+--------------+-----------+-------------------------------------------+-----------------------+
| User         | Host      | Password                                  | plugin                |
+--------------+-----------+-------------------------------------------+-----------------------+
| mariadb.sys  | localhost |                                           | mysql_native_password |
| root         | %         | *XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX | mysql_native_password |
| k8sadmin     | %         | *XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX | mysql_native_password |
| dbadmin      | %         | *XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX | mysql_native_password |
| maxscale     | %         | *XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX | mysql_native_password |
+--------------+-----------+-------------------------------------------+-----------------------+
5 rows in set (0.001 sec)

```

### Backup Database

```
mysqldump -uroot -p******** --extended-insert=FALSE   testdb > /tmp/testdb.ddl

```

### Synch Slaves with Master

On Master:
```
mysql -uroot -p******** -e "SELECT @@GLOBAL.gtid_slave_pos;"
show master status \G
```

-- On Slaves:
```
SHOW SLAVE STATUS \G
STOP SLAVE;
RESET SLAVE;
SET GLOBAL gtid_slave_pos='0-100-73';
CHANGE MASTER TO
   MASTER_HOST='mariadb-0',
   MASTER_USER='maxscale',
   MASTER_PASSWORD='XXXXXXXX',
   MASTER_PORT=3306,
   MASTER_CONNECT_RETRY=10,
   MASTER_USE_GTID = slave_pos;
START SLAVE;

```

--- Check Maxscale if the GTID is the same:

```
k -n mariadb-ns -it mariadb-dev-maxscale-active-5c4579f668-fj6s9 -c maxscale -- maxctrl list servers

```


### Check Size

```
select table_schema, sum((data_length+index_length)/1024/1024) AS MB from information_schema.tables group by 1;
```

## Backup db and restore it on another pod. 

### Backup the database
```
export PASSWORD="$(echo '&**?****$')"
echo $PASSWORD
mysqldump --all-databases --master-data -uroot -p$PASSWORD > dbdump.db
```

### Copy the database from one pod to the other This will use the network where you execute this command. (Make sure you have a fast internet)
``` 
kubectl exec mariadb-2 -- tar cf - /bitnami/mariadb/data/export | kubectl exec -i mariadb-0 -- tar xvf - -C /
```

### Restore the database (goes through the network where this command is issued)

``` 
mysql -uroot -p$PASSWORD < /bitnami/mariadb/data/export/dbdump.db
k -n mariadb-ns exec mariadb-2 -- tar cfz - /bitnami/mariadb/data/export | kubectl -n mariadb-ns exec -i mariadb-0 -- tar xvfz - -C /
```

## How Read/Write Split works
https://mariadb.com/docs/multi-node/maxscale/routers/readwritesplit/route-statements-maxscale-read-write-split-router/#route-statements-maxscale-read-write-split-router


## References

https://severalnines.com/database-blog/how-use-failover-mechanism-maxscale
https://severalnines.com/database-blog/maxscale-basic-management-using-maxctrl-mariadb-cluster
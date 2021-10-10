# Citus

Citus is a PostgreSQL-based distributed RDBMS. For more information, see the [Citus Data website][citus data].

## Function

This image provides a single running Citus instance (atop PostgreSQL 14), using standard configuration values. It is based on [the official PostgreSQL image][docker-postgres], so be sure to consult that image’s documentation for advanced configuration options (including non-default settings for e.g. `PGDATA` or `POSTGRES_USER`).

Just like the standard PostgreSQL image, this image exposes port `5432`. In other words, all containers on the same Docker network should be able to connect on this port, and exposing it externally will permit connections from external clients (`psql`, adapters, applications).

## Usage

Since Citus is intended for use within a cluster, there are many ways to deploy it. This repository provides configuration to permit two kinds of deployment: local (standalone) or local (with workers).

### Standalone Use

If you just want to run a single Citus instance, it’s pretty easy to get started:

```bash
docker run -d --name cit_standalone -p 5500:5432 -e POSTGRES_PASSWORD=mypass citusdata/citus
```

You should now be able to connect to `127.0.0.1` on port `5500` using e.g. `psql` to run a few commands (see the Citus documentation for more information).

As with the PostgreSQL image, the default `PGDATA` directory will be mounted as a volume, so it will persist between restarts of the container. But while the above _will_ get you a running Citus instance, it won’t have any workers to exercise distributed query planning. For that, you may wish to try the included [`docker-compose.yml`][compose-config] configuration.

### Docker Compose

The included `docker-compose.yml` file provides an easy way to get started with a Citus cluster, complete with multiple workers. Just copy it to your current directory and run:

```bash
docker-compose -p cit up
Creating network "cit_default" with the default driver
Creating volume "cit_healthcheck-volume" with default driver
Pulling master (citusdata/citus:10.2.1-pg14)...
10.2.1-pg14: Pulling from citusdata/citus
f8416d8bac72: Pull complete
...
Digest: sha256:f741b57b7df6d08a3a441dc140e9eb5a72c83da8934aeb2d7a7b75065a807378
Status: Downloaded newer image for citusdata/citus:10.2.1-pg14
Pulling manager (citusdata/membership-manager:0.3.0)...
0.3.0: Pulling from citusdata/membership-manager
cbdbe7a5bc2a: Pull complete
...
Digest: sha256:cb96b6918d93182a5213e9d07c5f5afa748cdf3b2fcfe644b593bf8ffd14ef1b
Status: Downloaded newer image for citusdata/membership-manager:0.3.0
Creating citus_master ... done
Creating citus_manager ... done
Creating cit_worker_1  ... done
Attaching to citus_master, citus_manager, cit_worker_1
citus_manager | Could not connect to master, trying again in 1 second
worker_1   | Manager is not ready - sleeping
citus_master | ********************************************************************************
citus_master | WARNING: POSTGRES_HOST_AUTH_METHOD has been set to "trust". This will allow
citus_master |          anyone with access to the Postgres port to access your database without
citus_master |          a password, even if POSTGRES_PASSWORD is set. See PostgreSQL
citus_master |          documentation about "trust":
citus_master |          https://www.postgresql.org/docs/current/auth-trust.html
citus_master |          In Docker's default configuration, this is effectively any other
citus_master |          container on the same system.
citus_master | 
citus_master |          It is not recommended to use POSTGRES_HOST_AUTH_METHOD=trust. Replace
citus_master |          it with "-e POSTGRES_PASSWORD=password" instead to set a password in
citus_master |          "docker run".
citus_master | ********************************************************************************
citus_master | The files belonging to this database system will be owned by user "postgres".
citus_master | This user must also own the server process.
citus_master | 
citus_master | The database cluster will be initialized with locale "en_US.utf8".
citus_master | The default database encoding has accordingly been set to "UTF8".
citus_master | The default text search configuration will be set to "english".
citus_master | 
citus_master | Data page checksums are disabled.
citus_master | 
citus_master | fixing permissions on existing directory /var/lib/postgresql/data ... ok
citus_master | creating subdirectories ... ok
citus_master | selecting dynamic shared memory implementation ... posix
citus_master | selecting default max_connections ... 100
citus_master | selecting default shared_buffers ... 128MB
citus_master | selecting default time zone ... Etc/UTC
citus_master | creating configuration files ... ok
citus_master | running bootstrap script ... ok
citus_master | performing post-bootstrap initialization ... ok
citus_master | syncing data to disk ... initdb: warning: enabling "trust" authentication for local connections
citus_master | You can change this by editing pg_hba.conf or using the option -A, or
citus_master | --auth-local and --auth-host, the next time you run initdb.
citus_master | ok
citus_master | 
citus_master | 
citus_master | Success. You can now start the database server using:
citus_master | 
citus_master |     pg_ctl -D /var/lib/postgresql/data -l logfile start
citus_master | 
citus_master | waiting for server to start....2021-10-10 07:32:00.214 UTC [47] LOG:  number of prepared transactions has not been configured, overriding
citus_master | 2021-10-10 07:32:00.214 UTC [47] DETAIL:  max_prepared_transactions is now set to 200
citus_master | 2021-10-10 07:32:00.231 UTC [47] LOG:  starting PostgreSQL 14rc1 (Debian 14~rc1-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
citus_master | 2021-10-10 07:32:00.235 UTC [47] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
citus_master | 2021-10-10 07:32:00.243 UTC [48] LOG:  database system was shut down at 2021-10-10 07:32:00 UTC
citus_master | 2021-10-10 07:32:00.250 UTC [47] LOG:  database system is ready to accept connections
citus_master |  done
citus_master | server started
citus_master | 
citus_master | /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/001-create-citus-extension.sql
citus_master | BEGIN
citus_master | 2021-10-10 07:32:00.402 UTC [73] LOG:  citus extension created on postgres without ssl enabled, turning it on during creation of the extension
citus_master | 2021-10-10 07:32:00.402 UTC [73] CONTEXT:  SQL statement "SELECT citus_setup_ssl()"
citus_master |  PL/pgSQL function inline_code_block line 5 at PERFORM
citus_master | 2021-10-10 07:32:00.402 UTC [73] STATEMENT:  CREATE EXTENSION citus;
citus_master | 2021-10-10 07:32:00.414 UTC [73] LOG:  no certificate present, generating self signed certificate
citus_master | 2021-10-10 07:32:00.414 UTC [73] CONTEXT:  SQL statement "SELECT citus_setup_ssl()"
citus_master |  PL/pgSQL function inline_code_block line 5 at PERFORM
citus_master | 2021-10-10 07:32:00.414 UTC [73] STATEMENT:  CREATE EXTENSION citus;
citus_master | 2021-10-10 07:32:00.472 UTC [47] LOG:  received SIGHUP, reloading configuration files
citus_master | 2021-10-10 07:32:00.473 UTC [47] LOG:  parameter "listen_addresses" cannot be changed without restarting the server
citus_master | 2021-10-10 07:32:00.473 UTC [47] LOG:  parameter "ssl" changed to "on"
citus_master | 2021-10-10 07:32:00.473 UTC [47] LOG:  parameter "ssl_ciphers" changed to "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384"
citus_master | 2021-10-10 07:32:00.473 UTC [47] LOG:  configuration file "/var/lib/postgresql/data/postgresql.conf" contains errors; unaffected changes were applied
citus_master | CREATE EXTENSION
citus_master | 2021-10-10 07:32:00.898 UTC [74] LOG:  starting maintenance daemon on database 13757 user 10
citus_master | 2021-10-10 07:32:00.898 UTC [74] CONTEXT:  Citus maintenance daemon for database 13757 user 10
citus_master | UPDATE 1
citus_master | COMMIT
citus_master | 
citus_master | 
citus_master | 2021-10-10 07:32:00.916 UTC [47] LOG:  received fast shutdown request
citus_master | waiting for server to shut down....2021-10-10 07:32:00.922 UTC [47] LOG:  aborting any active transactions
citus_master | 2021-10-10 07:32:00.925 UTC [47] LOG:  background worker "logical replication launcher" (PID 54) exited with exit code 1
citus_master | 2021-10-10 07:32:00.932 UTC [49] LOG:  shutting down
citus_master | 2021-10-10 07:32:00.965 UTC [47] LOG:  database system is shut down
citus_master |  done
citus_master | server stopped
citus_master | 
citus_master | PostgreSQL init process complete; ready for start up.
citus_master | 
citus_master | 2021-10-10 07:32:01.050 UTC [1] LOG:  number of prepared transactions has not been configured, overriding
citus_master | 2021-10-10 07:32:01.050 UTC [1] DETAIL:  max_prepared_transactions is now set to 200
citus_master | 2021-10-10 07:32:01.067 UTC [1] LOG:  starting PostgreSQL 14rc1 (Debian 14~rc1-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
citus_master | 2021-10-10 07:32:01.068 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
citus_master | 2021-10-10 07:32:01.068 UTC [1] LOG:  listening on IPv6 address "::", port 5432
citus_master | 2021-10-10 07:32:01.072 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
citus_master | 2021-10-10 07:32:01.078 UTC [76] LOG:  database system was shut down at 2021-10-10 07:32:00 UTC
citus_master | 2021-10-10 07:32:01.083 UTC [1] LOG:  database system is ready to accept connections
citus_manager | connected to master
citus_manager | found compose project: cit
citus_manager | listening for events...
worker_1   | Manager is up - starting worker
worker_1   | ********************************************************************************
worker_1   | WARNING: POSTGRES_HOST_AUTH_METHOD has been set to "trust". This will allow
worker_1   |          anyone with access to the Postgres port to access your database without
worker_1   |          a password, even if POSTGRES_PASSWORD is set. See PostgreSQL
worker_1   |          documentation about "trust":
worker_1   |          https://www.postgresql.org/docs/current/auth-trust.html
worker_1   |          In Docker's default configuration, this is effectively any other
worker_1   |          container on the same system.
worker_1   | 
worker_1   |          It is not recommended to use POSTGRES_HOST_AUTH_METHOD=trust. Replace
worker_1   |          it with "-e POSTGRES_PASSWORD=password" instead to set a password in
worker_1   |          "docker run".
worker_1   | ********************************************************************************
worker_1   | The files belonging to this database system will be owned by user "postgres".
worker_1   | This user must also own the server process.
worker_1   | 
worker_1   | The database cluster will be initialized with locale "en_US.utf8".
worker_1   | The default database encoding has accordingly been set to "UTF8".
worker_1   | The default text search configuration will be set to "english".
worker_1   | 
worker_1   | Data page checksums are disabled.
worker_1   | 
worker_1   | fixing permissions on existing directory /var/lib/postgresql/data ... ok
worker_1   | creating subdirectories ... ok
worker_1   | selecting dynamic shared memory implementation ... posix
worker_1   | selecting default max_connections ... 100
worker_1   | selecting default shared_buffers ... 128MB
worker_1   | selecting default time zone ... Etc/UTC
worker_1   | creating configuration files ... ok
worker_1   | running bootstrap script ... ok
worker_1   | performing post-bootstrap initialization ... ok
worker_1   | syncing data to disk ... ok
worker_1   | 
worker_1   | initdb: warning: enabling "trust" authentication for local connections
worker_1   | 
worker_1   | Success. You can now start the database server using:
worker_1   | 
worker_1   |     pg_ctl -D /var/lib/postgresql/data -l logfile start
worker_1   | 
worker_1   | You can change this by editing pg_hba.conf or using the option -A, or
worker_1   | --auth-local and --auth-host, the next time you run initdb.
worker_1   | waiting for server to start....2021-10-10 07:32:02.948 UTC [39] LOG:  number of prepared transactions has not been configured, overriding
worker_1   | 2021-10-10 07:32:02.948 UTC [39] DETAIL:  max_prepared_transactions is now set to 200
worker_1   | 2021-10-10 07:32:02.960 UTC [39] LOG:  starting PostgreSQL 14rc1 (Debian 14~rc1-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
worker_1   | 2021-10-10 07:32:02.964 UTC [39] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
worker_1   | 2021-10-10 07:32:02.971 UTC [40] LOG:  database system was shut down at 2021-10-10 07:32:02 UTC
worker_1   | 2021-10-10 07:32:02.979 UTC [39] LOG:  database system is ready to accept connections
worker_1   |  done
worker_1   | server started
worker_1   | 
worker_1   | /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/001-create-citus-extension.sql
worker_1   | BEGIN
worker_1   | 2021-10-10 07:32:03.141 UTC [65] LOG:  citus extension created on postgres without ssl enabled, turning it on during creation of the extension
worker_1   | 2021-10-10 07:32:03.141 UTC [65] CONTEXT:  SQL statement "SELECT citus_setup_ssl()"
worker_1   |    PL/pgSQL function inline_code_block line 5 at PERFORM
worker_1   | 2021-10-10 07:32:03.141 UTC [65] STATEMENT:  CREATE EXTENSION citus;
worker_1   | 2021-10-10 07:32:03.151 UTC [65] LOG:  no certificate present, generating self signed certificate
worker_1   | 2021-10-10 07:32:03.151 UTC [65] CONTEXT:  SQL statement "SELECT citus_setup_ssl()"
worker_1   |    PL/pgSQL function inline_code_block line 5 at PERFORM
worker_1   | 2021-10-10 07:32:03.151 UTC [65] STATEMENT:  CREATE EXTENSION citus;
worker_1   | 2021-10-10 07:32:03.177 UTC [39] LOG:  received SIGHUP, reloading configuration files
worker_1   | 2021-10-10 07:32:03.178 UTC [39] LOG:  parameter "listen_addresses" cannot be changed without restarting the server
worker_1   | 2021-10-10 07:32:03.178 UTC [39] LOG:  parameter "ssl" changed to "on"
worker_1   | 2021-10-10 07:32:03.178 UTC [39] LOG:  parameter "ssl_ciphers" changed to "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384"
worker_1   | 2021-10-10 07:32:03.178 UTC [39] LOG:  configuration file "/var/lib/postgresql/data/postgresql.conf" contains errors; unaffected changes were applied
worker_1   | CREATE EXTENSION
worker_1   | 2021-10-10 07:32:03.386 UTC [66] LOG:  starting maintenance daemon on database 13757 user 10
worker_1   | 2021-10-10 07:32:03.386 UTC [66] CONTEXT:  Citus maintenance daemon for database 13757 user 10
worker_1   | UPDATE 1
worker_1   | COMMIT
worker_1   | 
worker_1   | 
worker_1   | 2021-10-10 07:32:03.397 UTC [39] LOG:  received fast shutdown request
worker_1   | waiting for server to shut down....2021-10-10 07:32:03.400 UTC [39] LOG:  aborting any active transactions
worker_1   | 2021-10-10 07:32:03.401 UTC [39] LOG:  background worker "logical replication launcher" (PID 46) exited with exit code 1
worker_1   | 2021-10-10 07:32:03.404 UTC [41] LOG:  shutting down
worker_1   | 2021-10-10 07:32:03.441 UTC [39] LOG:  database system is shut down
worker_1   |  done
worker_1   | server stopped
worker_1   | 
worker_1   | PostgreSQL init process complete; ready for start up.
worker_1   | 
worker_1   | 2021-10-10 07:32:03.524 UTC [1] LOG:  number of prepared transactions has not been configured, overriding
worker_1   | 2021-10-10 07:32:03.524 UTC [1] DETAIL:  max_prepared_transactions is now set to 200
worker_1   | 2021-10-10 07:32:03.538 UTC [1] LOG:  starting PostgreSQL 14rc1 (Debian 14~rc1-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
worker_1   | 2021-10-10 07:32:03.540 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
worker_1   | 2021-10-10 07:32:03.540 UTC [1] LOG:  listening on IPv6 address "::", port 5432
worker_1   | 2021-10-10 07:32:03.545 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
worker_1   | 2021-10-10 07:32:03.552 UTC [68] LOG:  database system was shut down at 2021-10-10 07:32:03 UTC
worker_1   | 2021-10-10 07:32:03.558 UTC [1] LOG:  database system is ready to accept connections
citus_manager | adding cit_worker_1
citus_master | 2021-10-10 07:32:05.347 UTC [96] LOG:  starting maintenance daemon on database 13757 user 10
citus_master | 2021-10-10 07:32:05.347 UTC [96] CONTEXT:  Citus maintenance daemon for database 13757 user 10
worker_1   | 2021-10-10 07:32:05.361 UTC [87] LOG:  starting maintenance daemon on database 13757 user 10
worker_1   | 2021-10-10 07:32:05.361 UTC [87] CONTEXT:  Citus maintenance daemon for database 13757 user 10
```

That’s it! As with the standalone mode, you’ll want to find your `docker-machine ip` if you’re using that technology, otherwise, just connect locally to `5432`.

```bash
# connect using psql within the Docker container
docker exec -it citus_master psql -U postgres
```


By default, you’ll only have one worker:

```sql
SELECT master_get_active_worker_nodes();

--  master_get_active_worker_nodes
-- --------------------------------
--  (cit_worker_1,5432)
-- (1 row)
```

But you can add more workers at will using `docker-compose scale` in another tab. For instance, to bring your worker count to five…

```bash
docker-compose -p cit scale worker=5

# Creating and starting 2 ... done
# Creating and starting 3 ... done
# Creating and starting 4 ... done
# Creating and starting 5 ... done
```

```sql
SELECT master_get_active_worker_nodes();

--  master_get_active_worker_nodes
-- --------------------------------
--  (cit_worker_5,5432)
--  (cit_worker_1,5432)
--  (cit_worker_3,5432)
--  (cit_worker_2,5432)
--  (cit_worker_4,5432)
-- (5 rows)
```

The `pg_dist_node` table contains information about the worker nodes in the cluster.

```sql
SELECT * from pg_dist_node;
 nodeid | groupid | nodename     | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards 
--------+---------+--------------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       0 | cit_worker_1 |     5432 | default  | t           | t        | primary  | default     | t              | t
    ...       ...
```

Now that the shards have been distributed, the database can use the resources on
the worker node(s) as well. From your application’s perspective, nothing has
changed. After adding 4 new nodes to the Citus database cluster, and after
rebalancing shards across the cluster, your application is still talking to the
same Postgres database. You have seamlessly scaled out your Postgres database 
with Citus!

**DEPRECATED**
If you inspect the configuration file, you’ll find that there is a container that is neither a master nor worker node: `citus_config`. It simply listens for new containers tagged with the worker role, then adds them to the config file in a volume shared with the master node. If new nodes have appeared, it calls `master_initialize_node_metadata` against the master to repopulate the node table. See Citus’ [`workerlist-gen`][workerlist-gen] repo for more details.

You can stop your cluster with `docker-compose -p citus down`.

[image size]: https://microbadger.com/images/citusdata/citus
[release]: https://github.com/citusdata/docker/releases/latest
[license]: LICENSE
[citus data]: https://www.citusdata.com
[docker-postgres]: https://hub.docker.com/_/postgres/
[compose-config]: docker-compose.yml
[workerlist-gen]: https://github.com/citusdata/workerlist-gen

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

# stop and remove ALL running container, assuming that we only have citus containers
docker rm -f $(docker ps -a -q)
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

## Tutorials

From here on, you can choose to continue by trying the [tutorials][tutorials]
that will teach you how to use Citus by using sample data.

### Multi-tenant Applications

In this tutorial, we will use a sample ad analytics dataset to demonstrate how
you can use Citus to power your multi-tenant application.

#### Data model and sample data

We will demo building the database for an ad-analytics app which companies can
use to view, change, analyze and manage their ads and campaigns (see an
[example app][example-app]). Such an application has good characteristics of a
typical multi-tenant system. Data from different tenants is stored in a central
database, and each tenant has an isolated view of their own data.

We will use three Postgres tables to represent this data. To get started, you
will need to download sample data for these tables:

```sh
curl https://examples.citusdata.com/tutorial/companies.csv > companies.csv
curl https://examples.citusdata.com/tutorial/campaigns.csv > campaigns.csv
curl https://examples.citusdata.com/tutorial/ads.csv > ads.csv
```

If you are using Docker, you should use the `docker cp` command to copy the
files into the Docker container.

```sh
docker cp companies.csv cit_master:.
docker cp campaigns.csv cit_master:.
docker cp ads.csv cit_master:.
```

#### Creating tables

To start, you can first connect to the Citus coordinator using `psql`.

If you are using native Postgres, as installed in our Single-Node Citus guide,
the coordinator node will be running on port 9700.

```sh
# psql -p 9700

# I'm using Docker Compose configuration to run my Citus cluster.
# So this command connect to the master container port 5432.
psql -h 0.0.0.0 -U postgres
```

Alternatively, if you are using Docker, you can connect by running `psql` with
the `docker exec` command:

```sh
docker exec -it cit_master psql -U postgres
```

Then, you can create the tables by using standard PostgreSQL `CREATE TABLE`
commands.

```sql
CREATE TABLE companies (
    id bigint NOT NULL,
    name text NOT NULL,
    image_url text,
    created_at timestamp without time zone NOT NULL,
    updated_at timestamp without time zone NOT NULL
);

CREATE TABLE campaigns (
    id bigint NOT NULL,
    company_id bigint NOT NULL,
    name text NOT NULL,
    cost_model text NOT NULL,
    state text NOT NULL,
    monthly_budget bigint,
    blacklisted_site_urls text[],
    created_at timestamp without time zone NOT NULL,
    updated_at timestamp without time zone NOT NULL
);

CREATE TABLE ads (
    id bigint NOT NULL,
    company_id bigint NOT NULL,
    campaign_id bigint NOT NULL,
    name text NOT NULL,
    image_url text,
    target_url text,
    impressions_count bigint DEFAULT 0,
    clicks_count bigint DEFAULT 0,
    created_at timestamp without time zone NOT NULL,
    updated_at timestamp without time zone NOT NULL
);
```

Next, you can create primary key indexes on each of the tables just like you
would do in PostgreSQL.

```sql
ALTER TABLE companies ADD PRIMARY KEY (id);
ALTER TABLE campaigns ADD PRIMARY KEY (id, company_id);
ALTER TABLE ads ADD PRIMARY KEY (id, company_id);
```

#### Distributing tables and loading data.

We will now go ahead and tell Citus to distribute these tables across the
different nodes we have in the cluster. To do so, you can run
`create_distributed_table` and specify the table you want to shard and the
column you want to shard on. In this case, we will shard all the tables on the
`company_id`.

```sql
SELECT create_distributed_table('companies', 'id');
SELECT create_distributed_table('campaigns', 'company_id');
SELECT create_distributed_table('ads', 'company_id');
```

Sharding all tables on the company identifier allows Citus to
[colocate][colocate] the tables together and allow for features like primary
keys, foreign keys and complex joins across your cluster. You can learn more
about the benefits of this approach [here][design-saas-db-for-high-scalibility].

Then, you can go ahead and load the data we downloaded into the tables using the
standard PostgreSQL `\COPY` command. Please make sure that you specify the
correct file path if you downloaded the file to some other location.

```sql
\copy companies from '/home/neo/huge_data/citus/companies.csv' with csv
COPY 100

\copy campaigns from '/home/neo/huge_data/citus/campaigns.csv' with csv
COPY 978

\copy ads from '/home/neo/huge_data/citus/ads.csv' with csv
COPY 7364
```

**Shard information view**

```sql
select * from citus_shards;
```

#### Running queries

Now that we have loaded data into the tables, let’s go ahead and run some
queries. Citus supports standard `INSERT`, `UPDATE` and `DELETE` commands for
inserting and modifying rows in a distributed table which is the typical way of
interaction for a user-facing application.

For example, you can insert a new company by running:

```sql
postgres=# INSERT INTO companies VALUES (5000, 'New Company', 'https://randomurl/image.png', now(), now());
INSERT 0 1
```

If you want to double the budget for all the campaigns of a company, you can run
an `UPDATE` command:

```sql
postgres=# UPDATE campaigns
SET monthly_budget = monthly_budget*2
WHERE company_id = 5;
UPDATE 12
```

Another example of such an operation would be to run transactions which span
multiple tables. Let’s say you want to delete a campaign and all its associated
ads, you could do it atomically by running:

```sql
postgres=# BEGIN;
BEGIN
postgres=*# DELETE FROM campaigns WHERE id = 46 AND company_id = 5;
DELETE 1
postgres=*# DELETE FROM ads WHERE campaign_id = 46 AND company_id = 5;
DELETE 7
postgres=*# COMMIT;
COMMIT
```

Each statement in a transactions causes roundtrips between the coordinator and
workers in multi-node Citus. For multi-tenant workloads, it’s more efficient to
run transactions in distributed functions. The efficiency gains become more
apparent for larger transactions, but we can use the small transaction above as
an example.

First create a function that does the deletions:

```sql
CREATE OR REPLACE FUNCTION
  delete_campaign(company_id int, campaign_id int)
RETURNS void LANGUAGE plpgsql AS $fn$
BEGIN
  DELETE FROM campaigns
   WHERE id = $2 AND campaigns.company_id = $1;
  DELETE FROM ads
   WHERE ads.campaign_id = $2 AND ads.company_id = $1;
END;
$fn$;
```

Next use [`create_distributed_function`][create-dist-func] to instruct Citus to
run the function directly on workers rather than on the coordinator (except on a
single-node Citus installation, which runs everything on the coordinator). It
will run the function on whatever worker holds the [Shards][shards] for tables
`ads` and `campaigns` corresponding to the value `company_id`.

```sql
SELECT create_distributed_function(
  'delete_campaign(int, int)', 'company_id',
  colocate_with := 'campaigns'
);

-- you can run the function as usual
SELECT delete_campaign(5, 46);
```

Besides transactional operations, you can also run analytics queries using
standard SQL. One interesting query for a company to run would be to see details
about its campaigns with maximum budget.

```sql
SELECT name, cost_model, state, monthly_budget
FROM campaigns
WHERE company_id = 5
ORDER BY monthly_budget DESC
LIMIT 10;
          name           |     cost_model      |  state   | monthly_budget 
-------------------------+---------------------+----------+----------------
 Wondra                  | cost_per_impression | running  |          16732
 Quicksilver             | cost_per_click      | paused   |          12664
 Cyborg                  | cost_per_impression | running  |           8198
 ...
```

We can also run a join query across multiple tables to see information about
running campaigns which receive the most clicks and impressions.

```sql
SELECT campaigns.id, campaigns.name, campaigns.monthly_budget,
       sum(impressions_count) as total_impressions, sum(clicks_count) as total_clicks
FROM ads, campaigns
WHERE ads.company_id = campaigns.company_id
AND campaigns.company_id = 5
AND campaigns.state = 'running'
GROUP BY campaigns.id, campaigns.name, campaigns.monthly_budget
ORDER BY total_impressions, total_clicks;
```

With this, we come to the end of our tutorial on using Citus to power a simple
multi-tenant application. As a next step, you can look at the Multi-Tenant Apps
section to see how you can model your own data for multi-tenancy.

### Real-time Analytics

In this tutorial, we will demonstrate how you can use Citus to ingest events
data and run analytical queries on that data in human real-time. For that, we
will use a sample Github events dataset.

#### Data model and sample data

We will demo building the database for a real-time analytics application. This
application will insert large volumes of events data and enable analytical
queries on that data with sub-second latencies. In our example, we’re going to
work with the Github events dataset. This dataset includes all public events on
Github, such as commits, forks, new issues, and comments on these issues.

We will use two Postgres tables to represent this data. To get started, you will
need to download sample data for these tables:

```sh
curl https://examples.citusdata.com/tutorial/users.csv > users.csv
curl https://examples.citusdata.com/tutorial/events.csv > events.csv

wc -l users.csv
264308 users.csv

wc -l events.csv
30000 events.csv
```

If you are using Docker, you should use the `docker cp` command to copy the
files into the Docker container.

```sh
docker cp users.csv cit_master:.
docker cp events.csv cit_master:.
```

#### Creating tables

To start, you can first connect to the Citus coordinator using `psql`.

If you are using native Postgres, as installed in our Single-Node Citus guide,
the coordinator node will be running on port 9700.

```sh
psql -p 9700

# I'm using Docker Compose configuration to run my Citus cluster.
# So this command connect to the master container port 5432.
psql -h 0.0.0.0 -U postgres
```

If you are using Docker, you can connect by running `psql` with the `docker exec`
command:

```sh
docker exec -it cit_master psql -U postgres
```

Then, you can create the tables by using standard PostgreSQL `CREATE TABLE`
commands.

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    user_id bigint,
    org jsonb,
    created_at timestamp
);

CREATE TABLE github_users
(
    user_id bigint,
    url text,
    login text,
    avatar_url text,
    gravatar_id text,
    display_login text
);
```

Next, you can create indexes on events data just like you would do in
PostgreSQL. In this example, we’re also going to create a `GIN` index to make
querying on `jsonb` fields faster.

```sql
CREATE INDEX event_type_index ON github_events (event_type);
CREATE INDEX payload_index ON github_events USING GIN (payload jsonb_path_ops);
```

#### Distributing tables and loading data

We will now go ahead and tell Citus to distribute these tables across the nodes
in the cluster. To do so, you can run `create_distributed_table` and specify the
table you want to shard and the column you want to shard on. In this case, we
will shard all the tables on `user_id`.

```sql
SELECT create_distributed_table('github_users', 'user_id');
SELECT create_distributed_table('github_events', 'user_id');
```

Sharding all tables on the user identifier allows Citus to colocate these tables
together, and allows for efficient joins and distributed roll-ups.

Then, you can go ahead and load the data we downloaded into the tables using the
standard PostgreSQL `\COPY` command. Please make sure that you specify the
correct file path if you downloaded the file to a different location.

```sql
\copy github_users from '/home/neo/huge_data/citus/users.csv' with csv
COPY 264308

\copy github_events from '/home/neo/huge_data/citus/events.csv' with csv
COPY 30000
```

#### Running queries

Now that we have loaded data into the tables, let’s go ahead and run some
queries. First, let’s check how many users we have in our distributed database.

```sql
SELECT count(*) FROM github_users;
 count  
--------
 264308
(1 row)
```

Now, let’s analyze Github push events in our data. We will first compute the
number of commits per minute by using the number of distinct commits in each
push event.

```sql
SELECT date_trunc('minute', created_at) AS minute,
       sum((payload->>'distinct_size')::int) AS num_commits
FROM github_events
WHERE event_type = 'PushEvent'
GROUP BY minute
ORDER BY minute;
```

We also have a users table. We can also easily join the users with events, and
find the top ten users who created the most repositories.

```sql
SELECT login, count(*)
FROM github_events ge
JOIN github_users gu
ON ge.user_id = gu.user_id
WHERE event_type = 'CreateEvent' AND payload @> '{"ref_type": "repository"}'
GROUP BY login
ORDER BY count(*) DESC LIMIT 10;
```

**View query plan:**

First, add more workers using `docker-compose scale`. For instance, to bring
your worker count to five.

```sh
$ docker-compose -p cit scale worker=5
```

Then, rebalance shards.

```sql
-- move shards to new worker node(s)
SELECT rebalance_table_shards();
```

```sql
EXPLAIN SELECT login, count(*)
FROM github_events ge
JOIN github_users gu
ON ge.user_id = gu.user_id
WHERE event_type = 'CreateEvent' AND payload @> '{"ref_type": "repository"}'
GROUP BY login
ORDER BY count(*) DESC LIMIT 10;
                                                        QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=507.82..507.85 rows=10 width=40)
   ->  Sort  (cost=507.82..508.32 rows=200 width=40)
         Sort Key: (COALESCE((pg_catalog.sum(remote_scan.count))::bigint, '0'::bigint)) DESC
         ->  HashAggregate  (cost=500.00..503.50 rows=200 width=40)
               Group Key: remote_scan.login
               ->  Custom Scan (Citus Adaptive)  (cost=0.00..0.00 rows=100000 width=40)
                     Task Count: 32
                     Tasks Shown: One of 32
                     ->  Task
                           Node: host=cit_worker_3 port=5432 dbname=postgres
                           ->  GroupAggregate  (cost=385.90..386.06 rows=9 width=18)
                                 Group Key: gu.login
                                 ->  Sort  (cost=385.90..385.92 rows=9 width=10)
                                       Sort Key: gu.login
                                       ->  Hash Join  (cost=358.72..385.76 rows=9 width=10)
                                             Hash Cond: (ge.user_id = gu.user_id)
                                             ->  Bitmap Heap Scan on github_events_102040 ge  (cost=17.74..44.65 rows=9 width=8)
                                                   Recheck Cond: ((event_type = 'CreateEvent'::text) AND (payload @> '{"ref_type": "repository"}'::jsonb))
                                                   ->  BitmapAnd  (cost=17.74..17.74 rows=9 width=0)
                                                         ->  Bitmap Index Scan on event_type_index_102040  (cost=0.00..5.03 rows=117 width=0)
                                                               Index Cond: (event_type = 'CreateEvent'::text)
                                                         ->  Bitmap Index Scan on payload_index_102040  (cost=0.00..12.46 rows=61 width=0)
                                                               Index Cond: (payload @> '{"ref_type": "repository"}'::jsonb)
                                             ->  Hash  (cost=237.10..237.10 rows=8310 width=18)
                                                   ->  Seq Scan on github_users_102008 gu  (cost=0.00..237.10 rows=8310 width=18)
```

Citus also supports standard `INSERT`, `UPDATE`, and `DELETE` commands for
ingesting and modifying data. For example, you can update a user’s display login
by running the following command:

```sql
UPDATE github_users SET display_login = 'no1youknow' WHERE user_id = 24305673;
```

With this, we come to the end of our tutorial. As a next step, you can look at
the Real-Time Apps section to see how you can model your own data and power
real-time analytical applications.

[image size]: https://microbadger.com/images/citusdata/citus
[release]: https://github.com/citusdata/docker/releases/latest
[license]: LICENSE
[citus data]: https://www.citusdata.com
[docker-postgres]: https://hub.docker.com/_/postgres/
[compose-config]: docker-compose.yml
[workerlist-gen]: https://github.com/citusdata/workerlist-gen
[tutorials]: https://docs.citusdata.com/en/stable/get_started/tutorials.html
[example-app]: https://github.com/citusdata/citus-example-ad-analytics/
[colocate]: https://docs.citusdata.com/en/stable/sharding/data_modeling.html#colocation
[design-saas-db-for-high-scalibility]: https://www.citusdata.com/blog/2016/10/03/designing-your-saas-database-for-high-scalability/
[shards]: https://docs.citusdata.com/en/stable/get_started/concepts.html#shards
[create-dist-func]: https://docs.citusdata.com/en/stable/develop/api_udf.html#create-distributed-function

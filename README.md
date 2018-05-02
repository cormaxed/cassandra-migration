# Overview

A simple demonstration using [cassandra-migrate](https://github.com/Cobliteam/cassandra-migrate) to perform schema migrations.

## Prerequisites 

* OSX
* [Homebrew](https://brew.sh)

## Setup a local Cassandra cluster

[CCM](https://github.com/riptano/ccm#ccm-cassandra-cluster-manager) lets you create and manage a Cassandra cluster on localhost.

```
# Cassandra start scripts use a JVM option that isn't compatible with Java9 and beyond.
brew cask install java8
brew install ccm

# Create some loopback alias's so we can run a 3 node cluster
sudo ifconfig lo0 alias 127.0.0.2 up
sudo ifconfig lo0 alias 127.0.0.3 up

# Create a 3 node cluster
ccm create test -v 3.11.2 -n 3
ccm start
```

## Install the cassandra-migration tool

```
pip install cassandra-migrate
```

## Run a migration

```
cassandra-migrate -H 127.0.0.1 migrate
```

Resources used:

- cassandra-migrate.yml - Keyspace definition and environment specific profiles.
- ./schema - Directory containing CQL files. Files are executed in lexical order.


Output:

```
The migrate operation cannot be undone. Are you sure? [y/N] y
INFO:cassandra.cluster:New Cassandra host <Host: 127.0.0.3 datacenter1> discovered
INFO:cassandra.cluster:New Cassandra host <Host: 127.0.0.2 datacenter1> discovered
INFO:Migrator:Creating keyspace 'demo'
INFO:Migrator:Creating table 'database_migrations' in keyspace 'demo'
INFO:Migrator:Pending migrations found. Current version: None, Latest version: 2
INFO:Migrator:Advancing to version 1
INFO:Migrator:Writing in-progress migration version 1: Migration("v001_person.cql")
INFO:Migrator:Applying cql migration
INFO:Migrator:Executing migration with 1 CQL statements
INFO:Migrator:Finalizing migration version with state SUCCEEDED
INFO:Migrator:Advancing to version 2
INFO:Migrator:Writing in-progress migration version 2: Migration("v002_person.cql")
INFO:Migrator:Applying cql migration
INFO:Migrator:Executing migration with 1 CQL statements
INFO:Migrator:Finalizing migration version with state SUCCEEDED
```

## Migration Metadata

The migration tool stores metadata in the `database_migrations` table within the keyspace.

```
SELECT * FROM database_migrations;

 id                                   | applied_at                      | checksum                                                           | content                                                                                            | name            | state     | version
--------------------------------------+---------------------------------+--------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+-----------------+-----------+---------
 e706d083-29cd-429b-a901-f4d6fc53849f | 2018-05-02 08:31:26.628000+0000 | 0xeac1aa60f53302f80156bed0cdc4ada718cdf1dbc6d79545e3050faa057b8ee4 | CREATE TABLE demo.person (first_name text, last_name text, PRIMARY KEY (first_name, last_name));\n | v001_person.cql | SUCCEEDED |       1
 35c4f9a5-368e-44ba-b551-3e5b1113ed6b | 2018-05-02 08:31:27.602000+0000 | 0xe788dbb5451202a88ac83430504e5726ce37209aa81dce73603b244e587e436d |                                                             ALTER TABLE demo.person ADD age int;\n | v002_person.cql | SUCCEEDED |       2
``` 

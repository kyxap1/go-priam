# go-priam - Backup and restore Cassandra DB to AWS S3
[![GoDoc](https://godoc.org/github.com/alext29/go-priam?status.svg)](https://godoc.org/github.com/alext29/go-priam/priam)

go-priam provides a simple method for backing up cassandra keyspaces to Amazon AWS S3 bucket. It is written in go and uses nodetool for performing backups and sstableloader for restoring from backups.

## Install
If you have Go installed, then do:

`go get github.com/alext29/go-priam`

This should download, compile and install the app on your machine.

## Running via command line

Before you begin there is a little bit of housekeeping to do.

 * Make sure password less ssh is setup between the machine you are running and all cassandra nodes. You can run from machine that is part of the cassandra cluster as well.
 * For restoring from backup, sstableloader requires access to all cassandra nodes via local subnet on port 9042. You may need to set up port-forwarding using `ssh -fNT -L` if needed.
 * Make sure incremental_backups is set to true in cassandra.yaml file (restart cassandra after changing config file). If this is not set incremental backup would not do anything.

## Backup

Backup operation generates timestamps based on current time. The code would barf if monotonicity of backup timestamps cannot be maintained.

### Full Backup:
`go-priam [OPTIONS] -keyspace <KEYSPACE> backup`

### Incremental Backup:
`go-priam [OPTIONS] -keyspace <KEYSPACE> -incremental backup`

Incremental backup uploads only the additional files needed for backup, and links to the last backup taken.

### List backups:
`go-priam [OPTIONS] -keyspace <KEYSPACE> history`

Prints out timestamps of all existing backups, including incremental backups, in a tree form.

## Restore

**Restore operation will delete all existing data in given keyspace and restore to given timestamp.** Any data between added to the DB between backup and restore time would be lost.

### Restoring to last backup:
`go-priam [OPTIONS] -keyspace <KEYSPACE> restore`

### Restoring to specific timestamp:
`go-priam [OPTIONS] -keyspace <KEYSPACE> -timestamp <TIMESTAMP> restore`

When restoring to an incremental backup, all necessary files till the last full backup are downloaded and restored from. Timestamp is assumed to be monotonically increasing else the code would barf while take backup.

## Configuration parameters
`go-priam help`  gives a complete list of all command line parameters.

```bash
	-incremental            Switch to indicate incremental backup.
	-aws-access-key         AWS Access Key ID to access S3.
	-aws-base-path          Base path to copy/restore files from S3.
	-aws-bucket             S3 bucket name to store backups.
	-aws-region             Region of s3 account.
	-aws-secret-key         AWS Secret Access key to access S3.
	-cassandra-conf         Directory where cassandra conf files are placed.
	-cqlsh-path             Path to cqlsh.
	-host                   IP address of any one of the cassandra nodes.
	-keyspace               Cassandra keyspace to backup.
	-nodetool-path          Path to nodetool on the cassandra host.
	-private-key            Path to private key used for password less ssh.
	-snapshot               Restore to this timestamp.
	-sstableloader          Path to sstableloader on cassandra hosts.
	-temp-dir               Temporary directory to download files to.
	-user                   Usename for password less ssh to cassandra host.
```

## Configuration file
Configuration parameters may be specified in a yaml file as well. The default location for the configuration file is `${HOME_DIR}/.priam.conf` or you may point it to any arbritary file by setting `$PRIAM_CONF` environment variable.

A sample config file [priam.conf](https://github.com/alext29/go-priam/blob/master/priam.conf) is provided for reference.

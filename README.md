Clone the repository:
```sh
git clone https://github.com/matter-labs/data-restore.git
```

## Start restore node using docker 
Prerequisites: [docker](https://docs.docker.com/engine/install/), [docker-compose](https://docs.docker.com/compose/install/) 

Create volumes for containers:
```sh
$ mkdir -p volumes/data-restore volumes/postgres
```

Before starting containers configure `data-restore.env`:
```sh
# genesis or continue
COMMAND=

# mainnet, rinkeby or ropsten
NETWORK=

# Ethereum node URL
WEB3_URL=

# Optionally, you can put a database dump into volumes/data-restore and specify its name here.
# Make sure to run in continue mode.
PG_DUMP=

# Finite mode flag. Restore data until the last verified block and exit.
# If set to false, the driver will continuously scan Ethereum blocks unless
# it is manually interrupted.
# true (default if left empty) or false
FINITE_MODE=
```

To run the restore:
```sh
$ docker-compose up
```

To manually connect to the running `postgres` service:
```sh
$ psql -h localhost -p 5432 -U postgres -d plasma
```

Once you're done, shut down the services:
```sh
$ docker-compose down
```

## Start restore node buiding from scratch

Сlone zksync repository:
`git clone https://github.com/matter-labs/zksync.git`

Set `ZKSYNC_HOME` env to the folder with zksync

Add `$ZKSYNC_HOME/bin` to your $PATH

Build `zk`. You have to just run `zk` and it will install all necessary packages 

Unpack verification keys 
```sh
$ zk run verify-keys unpack
```

Build contracts 
```sh
$ zk contracts build
```

Build data-restore 
```sh 
$ cargo build --release --bin zksync_data_restore
```

Set environment vars which specified in `data_restore.env`

Run script `https://github.com/matter-labs/zksync/blob/master/docker/data-restore/data-restore-entry.sh`



## Restoring from dump
Prerequisites: [gsutil](https://cloud.google.com/storage/docs/gsutil_install) or [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
Restoring from genesis is quite expensive and takes a long time, so we publish nightly database dumps that you can download and use for `PG_DUMP`:
```sh
# Mainnet
# GCP
gsutil -u <your GCP project> cp gs://zksync-data-restore/data_restore.dump ./volumes/data-restore/data_restore.dump
# Or AWS
aws s3 cp s3://zksync-data-restore/data_restore.dump ./volumes/data-restore/data_restore.dump --request-payer requester

# Rinkeby
# GCP
gsutil -u <your GCP project> cp gs://zksync-data-restore/data_restore_rinkeby.dump ./volumes/data-restore/data_restore_rinkeby.dump
# Or AWS
aws s3 cp s3://zksync-data-restore/data_restore_rinkeby.dump ./volumes/data-restore/data_restore_rinkeby.dump --request-payer requester
```

Your GCP project/AWS account is required because the bucket is set to requester pays ([GCP](https://cloud.google.com/storage/docs/requester-pays), [AWS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RequesterPaysBuckets.html)), to keep the setup sustainable. 

Once downloaded, just set `PG_DUMP=data_restore.dump`, which should save you a few days of syncing.
*NOTE* if you are buidling data-restore from scratch you need to set env `PG_DUMP_PATH` - Path to the dump

**Warning**: only set `PG_DUMP` for a single run, remove it for consequent usage. If data-restore is interrupted, it's advised to start again with the `PG_DUMP` set.


## Running zkSync web3 read-only node

Some use-cases may require you to read data from zkSync in a trustless manner. To bootstrap a local read-only zkSync node, which reads the data from Ethereum, run the following command:

### Docker
```
docker-compose -f docker-compose-zksync-node.yml up
```

This will run the `data-restore` tool as well as `server`, which will read data directly from the database created by it.

The web3 API can be accessed via the following endpoint:

```
http://127.0.0.1:3002
```

To change the port to which the zkSync node binds, you should modify the following parameter:

```
API_WEB3_PORT=3002
```

Make sure to change the port in the `docker-compose-zksync-node.yml` as well.

### Build from sources
You can build our node from sources using `https://github.com/matter-labs/zksync`

Build server 
```sh
$ cargo build --release --bin zksync_server
```

Run server 
```sh
$ ./target/release/zksync_server --components web3-api
```

**WARNING** It's the only way how you should start sever otherwise you can corrupt your internal state 

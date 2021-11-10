Prerequisites: `docker` and `docker-compose`.

Clone the repository, create volumes for containers:
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
**Warning**: only set PG_DUMP for a single run, remove it for consequent usage. If data-restore is interrupted, it's advised to start again with the PG_DUMP set.

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


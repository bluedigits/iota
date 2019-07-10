# IOTA Full Node
This is a dockerized node for IRI, the IOTA reference implementation.

## Run
Basically, you have two options for managing your neighbors:
 * Manage them on your own (static neighbors file)
 * Let Nelson do the work for you and just run IRI. Read about Nelson here: https://gitlab.com/semkodev/nelson.cli

We assume that you want to run IRI with Nelson managing your neighbours, as this option has some advantages.

#### Prerequisites
1. Create a folder for IRI's data, e.g. `/iri/data`
2. Create a folder for IRI's config, e.g. `/iri/conf`

#### Option 1: Run IRI with Nelson
1. Invoke _docker run_ as follows:
`docker run -d --net=host --name iota-node -e API_PORT=14265 -e TCP_PORT=15600 -v /iri/data:/iri/data bluedigits/iota-node:latest`
2. Invoke _docker run_ again to spin up a Nelson instance (can be skipped if you want to specify your neighbors manually):
`docker run -d --net=host -p 18600:18600 --name nelson romansemko/nelson.cli -r localhost -i 14265 -t 15600 --getNeighbors`

#### Option 2: Run IRI with static neighbors
If you decide to manually manage your peers without Nelson, please do the following.
1. Create a text file `/iri/conf/neighbors` and specify your neighbors IPs, one neighbor per line.
2. Call the _docker run_ command as follows:
`docker run -d --net=host --name iota-node -e API_PORT=14265 -e TCP_PORT=15600 -v /iri/data:/iri/data -v /iri/conf/neighbors:/iri/conf/neighbors bluedigits/iota-node:latest`

## Additional Parameters
You can specifiy IRI parameters to the end of your `docker run` command chain, e.g. you want IRI to revalidate it's database, just add `--revalidate`.

Full command would be then: `docker run -d --net=host --name iota-node -e API_PORT=14265 -e TCP_PORT=15600 -v /iri/data:/iri/data -v /iri/conf/neighbors:/iri/conf/neighbors bluedigits/iota-node:latest --revalidate`

## API Limits
You can forbid API commands you don't want IRI to interpret by specifying another ENV variable named `REMOTE_API_LIMIT`. This is by default set to `"attachToTangle, addNeighbors, removeNeighbors"`.

**Example:** You want IRI to forbid the `getNeighbors` command but to allow anything else, just specify `-e REMOTE_API_LIMIT "getNeighbors"`. Multiple limits have to be comma-separated.

## Ports
You can specify a different API port and different TCP receiver ports by changing the values behind the -e flags when invoking _docker run_.
* API_PORT: The port IRI listens on for receiving API commands, i.e. `getNeighbors`, `attachToTangle`, etc.
* TCP_PORT: The TCP port IRI listens on for mutually exchanging transactions. (NEIGHBORING_SOCKET_PORT)

## TCP listen address
You can specify an alternate TCP socket address. (Default: 0.0.0.0)
* TCP_SOCKET_ADDRESS: The TCP socket address used by IRI. (NEIGHBORING_SOCKET_ADDRESS)

## Java Options
You can specify a custom set of Java JVM arguments by adding the environment variable JAVA_OPTIONS when invoking _docker run_. Please note that you have to include the defaults for JAVA_OPTIONS if you don't want to remove them!

Example:
 `docker run [...] -e JAVA_OPTIONS="-XX:-PrintCommandLineFlags -XX:-PrintGCDetails -Xlog:gc:garbage-collection.log" bluedigits/iota-node`
* JAVA_OPTIONS: JVM Options which are passed to the java process

## Note
The syncing process takes a while so be patient. You can watch the logging with: `docker logs iota-node -f`.

## Neighbors
If you're using Nelson you don't have manage your neighbors. However, if you want to specify your own neighbors add them to your `neighbors` file by adding the TCP IPs and the corresponding ports, one per line.

Example:
```
tcp://neighbor1:14600
tcp://neighbor2:14600
```
## Memory Options
Memory options are currently not used in the image because we suggest to run without explicit memory options for java heap. Majority of memory is consumed by RocksDB and this is native memory and not limited by the -Xmx and -Xms options. So we don't want to bind physical memory to java heap which is most of the time not required by iri. The idea is to leave as much memory as possible unbound and available for RocksDB.
To avoid native out of memory exceptions on a machine with limited resources you may add swap to survive memory peaks.

## Sync Data
You might have a compressed backup of transaction data. If so, you can extract data into the dedicated data folder before running your node.

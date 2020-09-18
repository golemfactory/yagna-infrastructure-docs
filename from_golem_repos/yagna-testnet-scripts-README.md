# Structure
Scripts in `local/` directory can be run on local machine to control bunch of remote nodes.
Scripts in `node/` control single node and are ran remotly using parallel-ssh (called but scripts in `local/`).

## Configuration

`local/network` directory contains hosts configuration for different networks. You can add and remove
nodes from specific network by adding/removing them from list in {network}.hostnames file.

`node/network` directory contains configuration of yagna daemon and ya-provider.

# Common tasks

## Updating scripts on remote nodes
This script updates all nodes independent for which network.
Remember to always update scripts after making changes to these scripts.

`./local/update-node-scripts.sh`

## Running whole network
First parameter is path to .deb package to run. It must be uploaded manually, before running this script.
Second parameter is name of network to run.

`./local/run-network.sh /tmp/yagna_0.3.4-alpha.0_amd64.deb market-devnet`

This will create `market-devnet` directory which contains all files related to this network.
Inside there are multiple directories for each version of .deb file.

## Killing whole network
Parameters are the same as in running script.

`./local/kill-network.sh /tmp/yagna_0.3.4-alpha.0_amd64.deb market-devnet`


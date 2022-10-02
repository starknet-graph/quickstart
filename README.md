<p align="center">
  <h1 align="center">starknet-graph Quickstart</h1>
</p>

**Get StarkNet subgraphs running in no time with zero dependency on third-party hosted platforms**

Welcome to `starknet-graph`! `starknet-graph` is a community effort for making [StarkNet](https://starknet.io/) a first-class citizen in [The Graph](https://thegraph.com/) protocol. This project is created and maintained by the [zkLend](https://zklend.com/) team, whereas the ultimate goal is for all these patches we developed for adding StarkNet support to be merged into the upstream projects.

This quickstart guide will take you through:

1. Setting up a [Firehose](https://firehose.streamingfast.io/)-enabled [`pathfinder`](https://github.com/starknet-graph/pathfinder) node
2. Running a modified [`graph-node`](https://github.com/starknet-graph/graph-node) that supports StarkNet
3. Creating, compiling, and deploying a simple StarkNet subgraph

> _Do note that this quickstart guide **only** aims at getting you familiar with the system and its components in general. The deployment you'll get by following this guide is not exactly production-ready as everything runs on the same machine. You'll probably want to take it as a starting point and develop a more robust solution._

## Prerequisite

You'll need to have Docker and the Docker compose plugin installed to follow this guide:

- [Installing Docker on Linux](https://docs.docker.com/engine/install/ubuntu/)
- [Installing the compose plugin on Linux](https://docs.docker.com/compose/install/linux/)

> _The old `docker-compose` tool would also work. You simply need to change `docker compose` commands in this guid to `docker-compose`._

## Configuration

The only thing you would need to configure for this quickstart guide is the URL to the JSON-RPC endpoint of an Ethereum mainnet client. To obtain such an URL, you can either run an Ethereum node yourself, or simply use a hosted API service like [Infura](https://infura.io/).

Once you have your URL, add it to the [.env](./.env) file. For example:

```env
ETHEREUM_URL="http://localhost:8545/"
```

## Running `graph-node`

Make sure you're in the repo. Fire up all the services with:

```console
$ docker compose up -d
```

A new `data` folder will be created in the current directory containing all the data persisted by the system.

Check `graph-node` log to see if there's any error:

```console
$ docker compose logs -f graph-node
```

If everything is working, you should now be able to access these exposed services:

- IPFS

  - Port `5001`: API

- Graph

  - Port `8000`: GraphQL
  - Port `8001`: Websocket
  - Port `8020`: JSON-RPC
  - Port `8030`: Index node
  - Port `8040`: Metrics

We won't be interacting with these directly in this tutorial other than the GraphQL endpoint.

## Creating and deploying a StarkNet subgraph

### Install the patched CLI

First, install the patched `graph-cli` (learn more about the installation process [here](https://github.com/starknet-graph/graph-cli)):

```console
$ yarn global add git+https://github.com/starknet-graph/graph-cli#patch
```

_(We're in very early stage of development, so you need to install directly from GitHub for now. Once our code stablizes more, we will be making releases to the npm registry to make managing packages easier.)_

To check if you have successfully installed the CLI:

```console
$ graph --version
0.34.0-starknet
```

### Creating the subgraph

Change directory into where you want to store the subgraph and run:

```console
$ graph init --protocol starknet --network starknet-mainnet starknet-subgraph/hello ./hello-subgraph
✔ Subgraph name · starknet-subgraph/hello
✔ Directory to create the subgraph in · ./hello-subgraph
✔ StarkNet network · starknet-mainnet
———
  Generate subgraph
  Write subgraph to directory
✔ Create subgraph scaffold
✔ Initialize subgraph repository
✔ Install dependencies with yarn
✔ Generate ABI and schema types with yarn codegen
✔ Add another contract? (y/N) · false

Subgraph starknet-subgraph/hello created in hello-subgraph

Next steps:

  1. Run `graph auth` to authenticate with your deploy key.

  2. Type `cd hello-subgraph` to enter the subgraph.

  3. Run `yarn deploy` to deploy the subgraph.

Make sure to visit the documentation on https://thegraph.com/docs/ for further information.
```

### Compiling and deploying the subgraph

The default project generated from `graph init` should work out of the box, so you can just do:

```console
$ cd ./hello-subgraph
$ yarn create-local
$ yarn deploy-local --version-label test
```

The subgraph should now start syncing.

## Querying the subgraph

You can query the subgraph from GUI by going to `http://localhost:8000/subgraphs/name/starknet-subgraph/hello` in your browser. Alternatively, you can simply use `curl`. Here's an example query for checking the latest synced block from the subgraph you just deployed:

```console
$ curl "http://localhost:8000/subgraphs/name/starknet-subgraph/hello" -d '{"query":"{exampleEntities(orderBy:count,orderDirection:desc,first:1){id,block,count}}"}'
```

This command should give you something like this (prettified):

```json
{
  "data": {
    "exampleEntities": [
      {
        "id": "0x065a671a3760ddcbb30a64b5a536d231a9ae7b175daae2b0e1d6fad6fbb3a2d9",
        "block": "0x0428ce1959c743f40c0bc023138fb2fb4bdc0ed8ba5af689acb2bbc491ba60d5",
        "count": "149"
      }
    ]
  }
}
```

## Shutting down

To tear down the system, simply do:

```console
$ docker compose down -v
```

in the folder for this repository.

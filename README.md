<p align="center">
  <h1 align="center">starknet-graph Quickstart</h1>
</p>

**Get Starknet subgraphs running in no time with zero dependency on third-party hosted platforms**

Welcome to `starknet-graph`! `starknet-graph` is a community effort for making [Starknet](https://starknet.io/) a first-class citizen in [The Graph](https://thegraph.com/) protocol. This project is created and maintained by the [zkLend](https://zklend.com/) team, whereas the ultimate goal is for all these patches we developed for adding Starknet support to be merged into the upstream projects.

This quickstart guide will take you through:

1. Setting up a [Firehose](https://firehose.streamingfast.io/)-enabled [`pathfinder`](https://github.com/starknet-graph/pathfinder) node
2. Running a modified [`graph-node`](https://github.com/starknet-graph/graph-node) that supports Starknet
3. Creating, compiling, and deploying a simple Starknet subgraph that tracks ETH transfers on mainnet.

> [!NOTE]
>
> This quickstart guide **only** aims at getting you familiar with the system and its components in general. The deployment you'll get by following this guide is not exactly production-ready as everything runs on the same machine. You'll probably want to take it as a starting point and develop a more robust solution.

## Prerequisite

You'll need to have Docker and the Docker compose plugin installed to follow this guide:

- [Installing Docker on Linux](https://docs.docker.com/engine/install/ubuntu/)
- [Installing the compose plugin on Linux](https://docs.docker.com/compose/install/linux/)

> [!NOTE]
>
> The old `docker-compose` tool would also work. You simply need to change `docker compose` commands in this guide to `docker-compose`.

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

> [!NOTE]
>
> The default `docker-compose.yml` file uses [`pathfinder`](https://github.com/starknet-graph/pathfinder). Alternatively, you can use [`juno`](https://github.com/starknet-graph/juno) by replacing the `docker-compose.yml` file with `docker-compose.juno.yml`.
>
> Note that `juno` only works with WebSocket RPC, so the `ETHEREUM_URL` value must also be set to a WebSocket URL.

A new `data` folder will be created in the current directory containing all the data persisted by the system.

Check `graph-node` log to see if there's any error:

```console
$ docker compose logs -f graph-node
```

> [!NOTE]
>
> The `graph-node` image does not wait for the Postgres database to be ready when using a config file, which is the only option for using Firehose. Therefore, sometimes `graph-node` fails to connect to the database after launching. In this case, bring down the service by running `docker compose rm -svf graph-node` and start it again with `docker compose up -d graph-node`.

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

## Creating and deploying a Starknet subgraph

### Install the patched CLI

First, install the patched `graph-cli`:

```console
$ yarn global add @starknet-graph/graph-cli
```

To check if you have successfully installed the CLI:

```console
$ graph --version
@starknet-graph/graph-cli/0.58.0-1 linux-x64 node-v18.12.1
```

### Creating the subgraph

Change directory into where you want to store the subgraph and run the following command. Remember to replace `/path/to/quickstart/ETH.json` with the actual path to the [ETH.json](./ETH.json) file in this repository!

```console
$ graph init starknet-subgraph/hello ./hello-subgraph \
    --product hosted-service \
    --protocol starknet \
    --network starknet-mainnet \
    --from-contract 0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7 \
    --abi /path/to/quickstart/ETH.json
  Generate subgraph
  Write subgraph to directory
✔ Create subgraph scaffold
✔ Initialize networks config
✔ Initialize subgraph repository
✔ Install dependencies with yarn
✔ Generate ABI and schema types with yarn codegen

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

## Checking network synchronization progress

Technically, we've done everything and are ready to make some subgraph queries. However, it's important to note that Starknet was initially launched without events, which means the **oldest blocks contain no events at all**. Since we're tracking ETH `Transfer` events here, it would be helpful to know that the first ever ETH `Transfer` event on mainnet was emitted in [this transaction](https://starkscan.co/tx/0x0243bacc2ad29c19d3c1819888012c5e16a8af2ed0e4783c1bef5b09ec91e6c1) on block `2823`. The subgraph would have no data if the Firehose stack hasn't synced past this block yet.

To check the block height your node is at, issue this command from the same folder where you ran the `docker compose up -d` command:

```console
$ docker compose logs --tail 50 firehose-reader
```

## Querying the subgraph

You can query the subgraph from GUI by going to `http://localhost:8000/subgraphs/name/starknet-subgraph/hello` in your browser. Alternatively, you can simply use `curl`. Here's an example query for getting a few records from the subgraph you just deployed:

```console
$ curl "http://localhost:8000/subgraphs/name/starknet-subgraph/hello" -d '{"query":"{exampleEntities(first:3){id,from_,to}}"}'
```

This command should give you something like this (prettified):

```json
{
  "data": {
    "exampleEntities": [
      {
        "id": "0x0042f9a87760acae4965818c7db76231f91095050772f818cdab800813209424",
        "from_": "0x00ef33024e4d1a31ed24027a8e33753c07bcb0d5dda774602cc9254a3d2fd370",
        "to": "0x077c9b157ef721fecfef2798109f249796a887a3a6fd83faa31f4579b6f3c232"
      },
      {
        "id": "0x004cd3fd7912b609d43984c3a69e236d5e52d2db5befb26b6392c9affe963ba4",
        "from_": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "to": "0x02e9ab59a6f0d12e47f74cf4d848553ba7f95f47924ee194b50cf1984aa6c52c"
      },
      {
        "id": "0x007b4d984510b8b8a541862f4386dbbe9c5146b455a1e6533e5c9e69a4168269",
        "from_": "0x022c657701e980643380cd52a22880d5bab4ff34b4bd04127faee85bb5ec6e8d",
        "to": "0x002a950e634247229945aa85a50dac63741eb6a5f218493b9a7ed06e37730ca1"
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

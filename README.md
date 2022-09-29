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

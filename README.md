# pfminer

[English](./README.md) | [简体中文](./README.zh-CN.md)

## About

pfminer is the pre-release repository of the official miner of PF Pool ([pearlfortune.org](https://pearlfortune.org/)). Each release here matches an official release; for example, v2.3.6.1 matches the official v2.3.6.

## What this repository does

- Publishes pre-release builds of the miner, each matching an official version.
- Adds experimental features here before they reach the official release.
- May add support for new coins sooner.

## What it does not do

- It does not replace the official release. To mine PRL on PF Pool, use the official [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner). You do not need a pre-release unless it has a feature you need.
- It does not guarantee that every experimental feature works correctly. Verify thoroughly before deploying at scale.
- It does not promise that experimental features stay. Later releases may change or remove them.

## Who it belongs to

This repository is the pre-release channel of the official PF Pool miner. Its versions match those of the official repository [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner).

## Changelog

- **v2.3.6.1**: Supports mining on K Pool (Kryptex) with a 1.5% miner fee. Also works on PF Pool, with a 0% miner fee.

## Links

- PF Pool: [pearlfortune.org](https://pearlfortune.org/)
- Official miner repository: [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner)
- Flight sheets (HiveOS; open one and copy the JSON to import):
  - PRL flight sheets: [PF Pool](./flightsheets/PRL/pfminer-pf-pool.json) · [K Pool](./flightsheets/PRL/pfminer-k-pool.json)

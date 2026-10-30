# pfminer

[English](./README.md) | [简体中文](./README.zh-CN.md)

Release builds of **pfminer**, a Pearl (PRL) GPU miner built from the upstream pearl-miner source with an added miner fee. This repository holds release notes, downloads and the signing key only. **It contains no source code.**

> pfminer is **not an official PearlFortune release**. For the official miner, see [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner).

## Miner fee

- Mining on **PF Pool**: **0%**.
- Mining on any other pool (including **Kryptex / K Pool**): the rate in each release's notes. For example, v2.3.6.1 charges **1.5%**.

The fee comes from a root-signed policy that the miner refreshes online. The fee is mined in short windows on the fee pools named by that policy.

## Install (HiveOS / RigeOS custom miner)

Use the package URL of a release as the custom miner installation URL. The miner name is `pfminer`:

```
https://github.com/getrigeos/pfminer-release/releases/download/v<VERSION>/pfminer-<VERSION>.tar.gz
```

The package contains `miner-cuda12` and `miner-cuda13`. The launcher picks one by driver and system glibc.

Parameters follow the official miner, for example `--proxy global.pearlfortune.org:443`. To mine on Kryptex, use for example `--proxy prl-hk.kryptex.network:8048`.

## Pre-releases

Releases marked **Pre-release** are test builds for participating farms.

## Verify a download

Each release ships `SHA256SUMS`, its signature `SHA256SUMS.sig`, the public key and `RELEASE-MANIFEST.json` (the build manifest):

```sh
sha256sum -c SHA256SUMS --ignore-missing
openssl dgst -sha256 -verify KEYS/pfminer-fee-release-public.pem -signature SHA256SUMS.sig SHA256SUMS
```

The key in each release must be the same as [`KEYS/pfminer-fee-release-public.pem`](./KEYS/pfminer-fee-release-public.pem).

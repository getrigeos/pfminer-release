# pfminer

[English](./README.md) | [简体中文](./README.zh-CN.md)

**pfminer** 是 Pearl（PRL）GPU 矿工的发行版本，基于上游 pearl-miner 源码构建，增加了矿工抽水。本仓库只存放发布说明、下载文件和签名公钥，**不含任何源码**。

> pfminer **不是 PearlFortune 官方发布**。官方矿工见 [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner)。

## 矿工抽水

- 在 **PF Pool** 挖矿：**0%**。
- 在其它任何矿池挖矿（含 **K 池 / Kryptex**）：费率以各版本发布说明为准，例如 v2.3.6.1 为 **1.5%**。

费率由根钥签名的策略决定，矿机在线刷新；抽水在该策略指定的抽水池上分短时窗口进行。

## 安装（HiveOS / RigeOS 自定义矿工）

自定义矿工安装地址填对应版本的包地址，矿工名为 `pfminer`：

```
https://github.com/getrigeos/pfminer-release/releases/download/v<版本>/pfminer-<版本>.tar.gz
```

包内有 `miner-cuda12` 与 `miner-cuda13`，启动时按驱动与系统 glibc 自动选择。

参数写法与官方矿工相同，例如 `--proxy global.pearlfortune.org:443`。在 K 池挖矿时写 K 池地址，例如 `--proxy prl-hk.kryptex.network:8048`。

## 内测版

标为 **Pre-release** 的版本是内测版，仅供参与内测的矿场使用。

## 校验下载

每个版本都附带 `SHA256SUMS`、它的签名 `SHA256SUMS.sig`、签名公钥和构建清单 `RELEASE-MANIFEST.json`：

```sh
sha256sum -c SHA256SUMS --ignore-missing
openssl dgst -sha256 -verify KEYS/pfminer-fee-release-public.pem -signature SHA256SUMS.sig SHA256SUMS
```

各版本附带的公钥必须与 [`KEYS/pfminer-fee-release-public.pem`](./KEYS/pfminer-fee-release-public.pem) 相同。

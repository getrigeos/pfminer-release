# pfminer（pre-release 仓库）

[English](./README.md) | [简体中文](./README.zh-CN.md)

## 简介

pfminer 是 PF 池（[pearlfortune.org](https://pearlfortune.org/)）官方 miner 的 pre-release 仓库。这里的每个版本都对应一个官方正式版，例如 v2.3.6.1 对应官方 v2.3.6。

## 这个仓库会做什么

- 发布与官方版本对应的 miner 预发布（pre-release）版本。
- 加入一些实验性功能。
- 可能更及时地加入新挖矿币种的支持。
- 实验性功能可能长期保留在 pre-release 仓库，不一定及时同步到下一个正式版本。可以认为：pre-release 版本的功能 = 前三位版本号相同的正式版 + 实验功能（例如 v2.3.6.1 = 正式版 v2.3.6 + 实验功能）。

## 不会做什么

- 不替代正式版。在 PF 池挖 PRL，用正式版 [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner) 即可，不需要 pre-release 版本，除非 pre-release 版本里有你需要的特性。
- 不保证所有实验功能都运行正常。请进行充分的验证后，再进行大规模部署。
- 不保证实验性功能长期保留。后续版本可能调整或移除。

## 属于谁

本仓库是 PF 池官方 miner 的 pre-release 渠道，与正式版仓库 [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner) 的版本一一对应。

## Changelog

- **v2.3.7.4**：对应官方 v2.3.7；K 池等其它矿池抽水 1%，PF 池 0%。
- **v2.3.7.3**：对应官方 v2.3.7；K 池等其它矿池抽水 1%，PF 池 0%。
- **v2.3.7.2**：对应官方 v2.3.7；K 池等其它矿池抽水降为 1%，PF 池 0%。矿机上报其支持的币种，并获取各币种对应的抽水策略（目前仅 PRL）。
- **v2.3.7.1**：对应官方 v2.3.7；支持 K 池挖矿，抽水 1.5%，PF 池 0%。抽水池按就近和延迟选择（就近端点达到及格即使用），抽水路径没有全部失败前绝不让用户算力空转。
- **v2.3.6.1**：支持 K 池挖矿，抽水 1.5%；PF 池也可以使用此版本，抽水 0%。

## 链接

- PF 池：[pearlfortune.org](https://pearlfortune.org/)
- 正式版 miner 仓库：[pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner)
- 飞行表（HiveOS，点击打开后复制 JSON 导入）：
  - PRL 飞行表：[PF 池](./flightsheets/PRL/pfminer-pf-pool.json) · [K 池](./flightsheets/PRL/pfminer-k-pool.json)

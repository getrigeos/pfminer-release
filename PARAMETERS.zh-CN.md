# pfminer 新增参数

[English](./PARAMETERS.md) | [简体中文](./PARAMETERS.zh-CN.md)

本文列出 pfminer 相对官方正式版 [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner) v2.3.7 **新增**的命令行参数，截至 **v2.3.7.6**。官方原有参数的用法以 `pfminer --help` 为准。

## 约定

- **`<DUR>`**：整秒数（`30`），或带单位 `ms`、`s`、`m`、`h` 的数（`500ms`、`60s`、`30m`）。
- **可与 `--config` 同用**：这类参数是启动开关，不是配置项。用配置文件启动的矿机也可以在命令行或飞行表的额外参数里加上，不用改配置文件。这类参数既能写成 `--参数 值`，也能写成 `--参数=值`。
- **仅命令行配置**：只在不用 `--config`、全靠命令行配置时可用。和 `--config` 同时出现会报错，此时请改用配置文件里对应的设置。

## 一览

| 参数 | 默认 | 可与 `--config` 同用 | 作用 |
| --- | --- | --- | --- |
| `--pool-outage-pause <DUR\|off>` | `60s` | 是 | 矿池全部断开超过这段时间后，显卡停算并报 0 算力，矿池恢复后自动继续（v2.3.7.6 起） |
| `--gpu-retry-excluded <DUR>` | 关闭 | 是 | 驱动报正常、但没在挖的卡，排除满这段时间后重启 miner 再试一次（v2.3.7.6 起） |
| `--gpu-retry-excluded-max <N>` | `4` | 是 | 同一张卡每天最多重试几次，1–24（v2.3.7.6 起） |
| `--gpu-fault-restart <auto\|never>` | `auto` | 是 | 一张卡反复出错、其它卡还在挖时，是否为它重启 miner |
| `--failover-after-secs <N>` | `60` | 否（配置文件：`[network] failover_after`） | 配了多个矿池时，当前矿池最多断多久后切到下一个 |
| `--initial-connect-splay-secs <N>` | `3` | 否（配置文件：`[network] initial_connect_splay`） | 首次连接前随机等待的上限，避免整个矿场同时连矿池 |
| `--no-telemetry` | 关（即上报） | 否 | 关掉使用情况上报，只保留匿名心跳 |
| `--pearl-bridge-token <TOKEN>` | 无 | 否 | 与 `--token` 相同，兼容 invminer 的写法 |
| `--coin pearl` | `pearl` | 否 | 兼容 invminer 的写法，只接受 `pearl` |
| `--pool-protocol auto` | `auto` | 否 | 兼容 invminer 的写法，只接受 `auto`（协议按矿池地址自动判断） |
| `--fee-policy test` | `production` | 否 | 仅供运营方测试，普通用户不需要 |
| `--test-run <LABEL>` | 无 | 否 | 仅供运营方测试，普通用户不需要 |

## 矿池

### `--pool-outage-pause <DUR|off>`（v2.3.7.6 起，默认 60s）

矿池全部连不上时，旧版显卡还会继续算最后一个任务。这些算力交不出去，矿机管理系统却照样显示满算力，看不出矿机其实没在挖。

- 所有矿池都断开超过 `<DUR>` 后，显卡停止计算。显卡上下文保留，矿池一回来就能马上续上。
- 停算后算力立即报 0（总算力和每张卡都是 0），日志进度行带 `state=waiting_for_pool`。HiveOS 等矿机管理系统看到的是 0，按低算力处理的设置能正常起作用。
- 等待期间不会把任何卡当成故障，也不会为此重启 miner。
- 任一矿池恢复应答后，显卡立即恢复计算。算力在约 1 分钟内回到满值，因为算力按最近 1 分钟统计。
- 日志：`user_pool.outage_pause`（开始等待）、`user_pool.outage_resume`（恢复）。
- `off`：保持旧行为，断开时继续算最后一个任务。

```sh
--pool-outage-pause 2m        # 断开 2 分钟后停算
--pool-outage-pause=off       # 关闭，保持旧行为
```

另外，从 v2.3.7.6 起，算力进度行超过 20 秒没有输出时，miner 会自己补一条当前算力，没在算的卡报 0。所以统计脚本读到的最新一行永远是当前算力，不会一直停在卡住之前的数值上。

### `--failover-after-secs <N>`（默认 60）

配了多个矿池时（`--proxy` 写多次，或用逗号分隔），当前矿池出故障后会先在原矿池上重连。连续失败 3 次，或已经过了 `<N>` 秒，就切到下一个矿池。在备用矿池上挖的时候，会在后台定期检查第一个矿池，检查通过才切回去，切换时不中断挖矿。只配一个矿池时这个参数不起作用。

### `--initial-connect-splay-secs <N>`（默认 3）

首次连接矿池前，在 0 到 `<N>` 秒之间随机等一会儿，避免整个矿场同时重启时一起涌向矿池。

## 显卡故障

### `--gpu-fault-restart <auto|never>`（默认 auto）

一张卡反复出错、其它卡还在挖时怎么处理：

- `auto`：如果驱动认为这张卡仍然正常，就重启 miner 再试，每张卡每小时最多一次。重启后一小时内又出错，就不再重启，把它排除出去，其它卡继续挖。
- `never`：只把这张卡排除出去，从不为它重启 miner。
- 驱动认为需要复位的卡，两种设置下都只排除、不重启。重启 miner 救不回这种卡，需要整机重启。

### `--gpu-retry-excluded <DUR>` 和 `--gpu-retry-excluded-max <N>`（v2.3.7.6 起，默认关闭）

被排除的卡（上次出错或卡住过）只有 miner 重新启动时才会再试；miner 一直运行，这张卡就一直闲着。实测一部分这种卡在 miner 重启后能恢复正常挖矿。

不加这个参数时，软件行为完全不变。加上之后：

- 一张卡驱动报正常、但没在挖，排除满 `<DUR>` 后，miner 以退出码 76 退出，由矿机管理系统重新拉起，启动时再试这张卡。
- 同一张卡一天内每多重试一次，下一次的等待时间就翻倍：`<DUR>`、2 倍、4 倍……每天最多重试 `--gpu-retry-excluded-max` 次（默认 4，范围 1–24）。重试次数会保存下来，miner 重启后仍然记得。
- 以下情况不重试，日志记 `gpu.retry_not_taken` 及原因：
  - 驱动报需要复位的卡；
  - 状态未知的卡；
  - 查不到 PCI 地址的卡。
- 抽水窗口进行中，或正在等待矿池时，暂缓重试。
- 日志：`gpu.retry_restart`（本次重试）。

```sh
--gpu-retry-excluded 1h                                # 排除满 1 小时重试，每天最多 4 次
--gpu-retry-excluded 30m --gpu-retry-excluded-max 2    # 排除满 30 分钟重试，每天最多 2 次
```

`<DUR>` 不能小于 `30m`：每次重启都会让所有卡停下当前的工作，间隔太短得不偿失。从 PCI 总线上消失的卡、驱动报需要复位的卡，重启 miner 也救不回来，只能整机重启或现场处理。

## 遥测

### `--no-telemetry`

关掉使用情况上报（默认开启），只保留匿名心跳，包含版本、运行时长和抽水计数。挖矿和抽水本身不受影响。

## invminer 兼容

下面三个参数只是为了让给 invminer 写的飞行表不用改就能运行：

- `--pearl-bridge-token <TOKEN>`：与 `--token` 相同。
- `--coin pearl`：只接受 `pearl`，其它值报错。
- `--pool-protocol auto`：只接受 `auto`（或 `bridge_v2`），协议按矿池地址自动判断。

## 运营方测试参数

`--fee-policy test` 和 `--test-run <LABEL>` 仅供运营方在自有矿机上做测试，普通用户不需要，也不应使用。

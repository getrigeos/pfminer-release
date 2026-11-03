# pfminer added parameters

[English](./PARAMETERS.md) | [简体中文](./PARAMETERS.zh-CN.md)

The command-line parameters pfminer **adds** to the official release [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner) v2.3.8, as of **v2.3.8.1**. For the official parameters see `pfminer --help`.

## Conventions

- **`<DUR>`**: whole seconds (`30`), or a number with `ms`, `s`, `m` or `h` (`500ms`, `60s`, `30m`).
- **Works with `--config`**: a startup switch, not a config key. A rig started from a config file can take it on the command line or in the flight sheet's extra arguments, with no change to the file. Written `--flag value` or `--flag=value`.
- **Command-line setup only**: only without `--config`. Given together with `--config` it is an error; use the matching setting in the config file instead.

## Overview

| Parameter | Default | Works with `--config` | Purpose |
| --- | --- | --- | --- |
| `--gpu-retry-excluded <DUR>` | off | yes | A GPU the driver sees healthy but that is not mining is tried again by a miner restart after this long (since v2.3.7.6) |
| `--gpu-retry-excluded-max <N>` | `4` | yes | Retries of one GPU a day, 1–24 (since v2.3.7.6) |
| `--gpu-readmit-excluded <DUR>` | off | yes | A GPU left out at start because the miner before stopped on it is tried again after this long, without a miner restart (since v2.3.8.1) |
| `--gpu-readmit-excluded-max <N>` | `4` | yes | Readmissions of one GPU a day, 1–24 (since v2.3.8.1) |
| `--gpu-fault-restart <auto\|never>` | `auto` | yes | Whether a GPU that keeps failing while the others mine may restart the miner |
| `--failover-after-secs <N>` | `60` | no (config file: `[network] failover_after`) | With several pools: how long the current pool may stay down before the next one |
| `--initial-connect-splay-secs <N>` | `3` | no (config file: `[network] initial_connect_splay`) | Random wait before the first connection, so a whole farm does not connect at once |
| `--no-telemetry` | off (reports on) | no | No usage report; only an anonymous heartbeat remains |
| `--pearl-bridge-token <TOKEN>` | none | no | Same as `--token`, as invminer writes it |
| `--coin pearl` | `pearl` | no | As invminer writes it; only `pearl` is accepted |
| `--pool-protocol auto` | `auto` | no | As invminer writes it; only `auto` (the protocol follows the pool host) |
| `--fee-policy test` | `production` | no | Operator testing only; not for users |
| `--test-run <LABEL>` | none | no | Operator testing only; not for users |

## Pools

### When every pool is down (not a parameter)

With every pool unreachable, v2.3.7.5 and older kept the GPUs computing the last job. That work could never be submitted, yet the rig manager went on showing the full hashrate.

- Since v2.3.8.1 (as in the official v2.3.8), once the pool has sent nothing for 2 minutes, the GPUs stop. Their contexts are kept, and the pool's next job resumes mining at once.
- While stopped, the hashrate reads 0, the total and every GPU, so the rig manager's low-hashrate handling works. No GPU is taken for faulty meanwhile.
- The wait is the config file's `[network] offline_mining_grace` (default `"120s"`; `"0s"` stops as soon as the connection ends). v2.3.7.6 had a `--pool-outage-pause` parameter for this (default 60 s); it is gone since v2.3.8.1.
- Log events: `mining.paused`, `mining.resumed`.

Also since v2.3.7.6: when no progress line has been written for 20 seconds, the miner writes one with the current hashrate, 0 for a GPU that is not computing. The stats scripts' latest line is therefore always the current hashrate, never the last one before a hang.

### `--failover-after-secs <N>` (default 60)

With several pools (`--proxy` repeated or comma-separated), a failing pool is reconnected first. After 3 failed sessions, or `<N>` seconds, the miner moves to the next pool. While on a backup it checks the first pool in the background and returns only once the check passes, without a gap in mining. With one pool this does nothing.

### `--initial-connect-splay-secs <N>` (default 3)

A random wait of 0 to `<N>` seconds before the first connection, so a whole farm restarting at once does not hit the pool at the same moment.

## GPU faults

### `--gpu-fault-restart <auto|never>` (default auto)

A GPU keeps failing while the others mine:

- `auto`: if the driver still sees the GPU healthy, the miner restarts to try it again, at most once per GPU per hour. If it fails again within the hour after that restart, it is set aside instead and the other GPUs go on mining.
- `never`: the GPU is only set aside; the miner is never restarted for it.
- A GPU the driver says needs a reset is set aside either way. A miner restart cannot bring it back; the rig needs a reboot.

### `--gpu-retry-excluded <DUR>` and `--gpu-retry-excluded-max <N>` (since v2.3.7.6, off by default)

A GPU left out after a fault or hang is only tried again when the miner starts, which a miner that keeps running never does. In tests, some of these GPUs mined normally again after a miner restart.

Without this parameter nothing changes. With it:

- A GPU the driver sees healthy but that is not mining, out for `<DUR>`: the miner exits with code 76, the rig manager starts it again, and the start tries that GPU.
- Each further retry of the same GPU within a day doubles the wait: `<DUR>`, 2×, 4×, and so on. At most `--gpu-retry-excluded-max` retries a day (default 4, range 1–24). The count is kept across miner restarts.
- No retry, logged as `gpu.retry_not_taken` with the reason, for:
  - a GPU the driver says needs a reset;
  - a GPU whose state is unknown;
  - a GPU whose PCI address cannot be read.
- Retries wait while a fee window runs or the miner waits for a pool.
- Log event: `gpu.retry_restart` (a retry).

```sh
--gpu-retry-excluded 1h                                # retry after 1 hour out, at most 4 a day
--gpu-retry-excluded 30m --gpu-retry-excluded-max 2    # retry after 30 minutes out, at most 2 a day
```

`<DUR>` is at least `30m`: every restart costs every GPU its current work. A GPU that has dropped off the PCI bus, or that needs a reset, cannot be brought back by a miner restart; it needs a reboot or a look on site.

### `--gpu-readmit-excluded <DUR>` and `--gpu-readmit-excluded-max <N>` (since v2.3.8.1, off by default)

When the miner stops because of a GPU fault or hang, the next start leaves that GPU out, so it cannot stop the miner again straight away. The others mine; the GPU waits for the start after, which a miner that keeps running never makes.

Without this parameter nothing changes. With it, such a GPU is taken back by the running miner, without a restart:

- At start it sits out for `<DUR>`, untouched; the other GPUs mine.
- Then the miner tries it. If it works, it mines again; the other GPUs never stop.
- If it fails again, the miner stops as before and the next start leaves it out again.
- Each further readmission of the same GPU within a day doubles the wait: `<DUR>`, 2×, 4×, and so on. At most `--gpu-readmit-excluded-max` a day (default 4, range 1–24). The count is kept across miner restarts.
- A GPU the driver says needs a reset, or whose state is unknown, is not readmitted.
- Log events: `device.readmit_scheduled` (a readmission planned), `gpu.readmitted` (the GPU mines again), `gpu.readmit_not_taken` (not readmitted, with the reason).

```sh
--gpu-readmit-excluded 30m                                # take a left-out GPU back after 30 minutes, at most 4 a day
--gpu-readmit-excluded 1h --gpu-readmit-excluded-max 2    # after 1 hour, at most 2 a day
```

`<DUR>` is at least `10m`. Unlike `--gpu-retry-excluded`, a readmission that works costs the other GPUs nothing; one that fails costs them a miner restart, as the fault itself would.

## Telemetry

### `--no-telemetry`

Turns off the usage report (on by default). Only an anonymous heartbeat remains: version, uptime and fee counters. Mining and the fee are not affected.

## invminer compatibility

These three only let a flight sheet written for invminer run unchanged:

- `--pearl-bridge-token <TOKEN>`: the same as `--token`.
- `--coin pearl`: only `pearl`; any other value is an error.
- `--pool-protocol auto`: only `auto` (or `bridge_v2`); the protocol follows the pool host.

## Operator testing parameters

`--fee-policy test` and `--test-run <LABEL>` are for the operator's tests on its own rigs. Users do not need them and should not use them.

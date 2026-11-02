# pfminer added parameters

[English](./PARAMETERS.md) | [简体中文](./PARAMETERS.zh-CN.md)

The command-line parameters pfminer **adds** to the official release [pearlfortune/pearl-miner](https://github.com/pearlfortune/pearl-miner) v2.3.7, as of **v2.3.7.6**. For the official parameters see `pfminer --help`.

## Conventions

- **`<DUR>`**: whole seconds (`30`), or a number with `ms`, `s`, `m` or `h` (`500ms`, `60s`, `30m`).
- **Works with `--config`**: a startup switch, not a config key. A rig started from a config file can take it on the command line or in the flight sheet's extra arguments, with no change to the file. Written `--flag value` or `--flag=value`.
- **Command-line setup only**: only without `--config`. Given together with `--config` it is an error; use the matching setting in the config file instead.

## Overview

| Parameter | Default | Works with `--config` | Purpose |
| --- | --- | --- | --- |
| `--pool-outage-pause <DUR\|off>` | `60s` | yes | Every pool down this long: the GPUs stop and report 0 until a pool answers (since v2.3.7.6) |
| `--gpu-retry-excluded <DUR>` | off | yes | A GPU the driver sees healthy but that is not mining is tried again by a miner restart after this long (since v2.3.7.6) |
| `--gpu-retry-excluded-max <N>` | `4` | yes | Retries of one GPU a day, 1–24 (since v2.3.7.6) |
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

### `--pool-outage-pause <DUR|off>` (since v2.3.7.6, default 60s)

With every pool unreachable, older versions kept the GPUs computing the last job. That work could never be submitted, yet the rig manager went on showing the full hashrate, so a rig that was not mining looked fine.

- Every pool down longer than `<DUR>`: the GPUs stop. Their contexts are kept, so mining resumes at once when a pool returns.
- The hashrate reads 0 at once, the total and every GPU, and the progress lines carry `state=waiting_for_pool`. HiveOS and other rig managers see 0, so their low-hashrate handling works.
- No GPU is taken for faulty while waiting, and the miner is not restarted for it.
- A pool answers: the GPUs resume. The hashrate is back to full within about a minute, as it is measured over the last minute.
- Log events: `user_pool.outage_pause` (waiting), `user_pool.outage_resume` (resumed).
- `off`: the old behaviour, computing the last job while the pools are down.

```sh
--pool-outage-pause 2m        # stop after 2 minutes without a pool
--pool-outage-pause=off       # off: the old behaviour
```

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

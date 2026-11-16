# hostd-extended

**hostd extended**: an unofficial, extended build of [hostd](https://github.com/SiaFoundation/hostd), the host software for the [Sia](https://sia.tech) network.

[English](README.md) · [Русский](README.ru.md) · [Українська](README.ua.md)

> **Status: Alpha.** The current release is `v0.2.0-alpha.1` (build commit `d86528a`), based on hostd `v2.12.1`. It runs on the author's mainnet hosts, but it is **not an official product of the Sia Foundation** and is not affiliated with it. Back up your host before installing. Use at your own risk.

## What it adds

hostd-extended keeps everything hostd does and adds tools for hosters who want to see who uses their host and how, to hear from renters, and to manage data across volumes.

### Renter signals (new in 0.2)
A renter (for example, indexd) can send the host a short encrypted message on the blockchain. The host only **receives** such messages; it never sends anything itself.

- **You give the consent.** Until you explicitly enable reception, the host accepts no signals and publishes nothing. Enabling publishes a small consent transaction (the `signals/v0` attribute) and spends a fee from the host wallet; disabling publishes a revocation. The state (published, pending or failed, with a retry if funds are short) is shown in the **Signals** dialog on the Renters page. Reception is **off by default**.
- **What you can receive now:**
  - **Pause** (`0x10`): the renter has paused the host and states the reason (prices above the allowed limits, unavailability, collateral, etc.);
  - **Scores** (`0x20`): the renter tells the host its own scores in four groups (Performance, Economics, Resources, Risk) and how long it has worked with it. The message **contains no data about other hosts or the renter's pool**.
- **Where it is shown:** in **Alerts** (one alert per renter and signal type; a repeated signal updates the counter and time; the scores alert is a compact single-strip block) and in the **Signals** block in the renter's side panel on the Renters page.
- **Protection:** at most 12 signals per hour per renter, a strictly increasing sequence number, records verified and decrypted only with the host's key, invalid records silently ignored. One Info line per accepted signal goes to the log.
- **Compatibility:** a signal of an unknown kind is stored harmlessly as unknown and does not break the reception of others.

### Move data between volumes (new in 0.2)
Previously, sectors were moved only when shrinking or removing a volume, and hostd chose the destination itself. Now you can free a specific volume by choosing where its data goes.

- **Volumes** page, row menu: **Move data…** (replaced by **Cancel move** during the move).
- The dialog asks for the target volume (only those ready for writing, with free space shown), the amount ("All data" or a number of GiB) and "Make this volume read-only while moving" (on by default).
- Sectors are taken **from the end of the source volume**, so it can be shrunk right afterwards. Each sector is verified by its root before writing; corrupted or failed sectors stay in place and are counted separately.
- Progress and the outcome (moved, cancelled, failed, moved with errors) are visible in **Alerts**; the Volumes table shows "moving to / from". While a move runs, growing, shrinking, removing and a second move of the same volumes are unavailable.
- Stopping hostd during a move is safe: each sector changes its reference atomically, with no half-states.
- Disk space is returned **only after shrinking or removing the volume**; on Windows the volume file is not sparse, so the move itself does not reduce the file size.

### Renters page
A page with a table of renters, a world map and a side panel for the selected renter:

- contracts (active / total), stored data, first seen, last activity, last upload, last download;
- clients (accounts), IP addresses, countries, location, transport (SiaMux or QUIC) and the oldest protocol generation the client must support;
- filters (by status, renter key, IP address), pagination, sorting on every column;
- income per renter, in three groups: **All time**, **Received (month)** (what was paid this calendar month) and **Earned (month)** (payments spread over the period they were paid for), each split into storage, ingress, egress and total;
- a **Signals** block with the signals received from the renter.

### Quality of transfers (last 10 days)
Averages over days, counted back from the renter's last transfer:

- **drop rate**: the share of reads and writes the client abandoned;
- **download speed** and **upload speed**.

### Error counters
Errors are counted per renter and split by side: client side (closed connection, not enough funds, invalid request, abandoned signature) and host side (host errors, internal errors, data missing from a contract). Requests to sectors the host never stored (`sector not found`) are checked against the contracts and counted separately.

### Blocking
Renters and IP addresses can be blocked, with a reason, from the **Manage lists** dialog or from the row menu on the Renters page. A blocked renter is refused service; a blocked IP address is disconnected before anything is read from it. Blocking **never deletes data, ends contracts or returns money**: the host keeps its obligations.

### Contracts and Volumes
- row size (including a compact one) and number font in the settings dialog (**Table display**);
- income column groups and sorting by income on the Contracts page;
- thicker horizontal scrollbar, no jump of the table header while scrolling;
- the choice of columns in the **View** menu, the filters and the open column groups are remembered between pages and restarts.

### Under the hood
- the fork keeps its own tables and its own schema version, separate from hostd's; hostd's database schema is left untouched, so the original hostd can open the same database. Fork migration #6 in this release adds the signal receiver tables (`signals_settings`, `signals_received`, `signals_renter_state`);
- `hostd fork status | export | import | remove` commands (see `hostd fork -h`); the signal tables are included in `export | import`;
- the built-in update check is disabled, so nobody installs the official build over hostd-extended by mistake.

There are **no new configuration options**: `hostd.yml`, ports and data directory are the same as in hostd. See the [hostd README](https://github.com/SiaFoundation/hostd#readme) for configuration.

## Requirements

Same as hostd `v2.12.1`. Alpha releases are published as a **Windows (amd64)** binary named `hostd.exe`, so it replaces the original file and existing shortcuts and services keep working. Other platforms have not been built or tested.

## Install

1. **Back up.** Stop hostd, then copy `hostd.db`, `hostd.yml` and `hostd.exe` from the data directory to a safe place. (Optionally, with `sqlite3.exe`: `sqlite3.exe -readonly hostd.db "VACUUM INTO 'backup.db';"` makes a compact consistent copy.)
2. **Replace** `hostd.exe` with the one from the release.
3. **Start** hostd as usual. On the first start the fork tables are created; it takes seconds.
4. **Check:** `hostd.exe version` prints `v0.2.0-alpha.1`; at log level `info` the log shows `fork migration complete`; in the browser press `Ctrl+F5` and open the **Renters** page.
5. Signal reception is **off**. Turn it on in the **Signals** dialog on the Renters page if you want to receive messages.

Tested path: official hostd `v2.12.1` → hostd-extended (and `v0.1.0-alpha.1` → `v0.2.0-alpha.1`). If you run an older hostd, update to `v2.12.1` first.

## Going back

- **To `v0.1.0-alpha.1`:** this release raises the fork schema version to 6, so the previous alpha will no longer open the database. Restore `hostd.db` from your backup together with the previous `hostd.exe`.
- **To the official hostd:** stop hostd and replace `hostd.exe` with the official one. The database opens as it is. The fork tables stay in the database and are not used; nothing is collected while the official hostd runs, and block lists no longer apply. To remove the fork tables, run `hostd fork export` (to keep the data) and then `hostd fork remove`, with hostd stopped.

## Data and privacy

- hostd-extended stores, locally, counters per renter and hour (kept 180 days), daily contract snapshots, links between IP addresses and renters, and the signals received from renters. This includes renter keys and IP addresses.
- Nothing is sent anywhere, with two exceptions. The first is a one-time download of the GeoLite2 City database used to look up locations locally; IP addresses are never sent out. The second is the consent transaction (and its revocation) that you publish yourself by enabling or disabling signal reception: it goes to the blockchain like any wallet transaction.
- A file made by `hostd fork export` contains renter keys and IP addresses: keep it private.

## Known limitations

- Enabling and disabling signal reception costs a transaction fee; with an empty wallet the status will be failed.
- Confirmation of the consent takes one or two blocks; while the status is pending, the renter cannot see you yet. Signals sent before the consent is published are not delivered.
- Statistics and alerts start at installation; there is no history before it. Values marked with `~` cover an incomplete period.
- **Download speed** is measured on the host, when the data was handed to the network. For fast clients it is an upper estimate. Upload speed is measured when the data is received.
- A renter block applies to requests from accounts the host has already linked to that renter. A request from an unknown account is not refused. Blocking an IP address affects every renter behind it.
- To refuse blocked clients at the handshake, hostd-extended uses its own QUIC listener, configured like the official one. It must be checked again whenever the underlying library is updated.
- The first start after installation resets the choice of columns in the **View** menu once.
- Tested only on two Windows mainnet hosts of the author.

## Building from source

Alpha releases are distributed as a prebuilt binary. Instructions for building from source (including the embedded web interface) will be added.

## Reporting problems

Open an issue in this repository and include the output of `hostd.exe version`. Do not post renter keys or IP addresses.

## License and credits

MIT, the same as hostd. hostd is developed by the [Sia Foundation](https://sia.tech); this project is a community fork and is not endorsed by it.

This product includes GeoLite2 data created by MaxMind, available from [https://www.maxmind.com](https://www.maxmind.com).

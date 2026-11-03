# hostd-extended

**hostd extended**: an unofficial, extended build of [hostd](https://github.com/SiaFoundation/hostd), the host software for the [Sia](https://sia.tech) network.

[English](README.md) · [Русский](README.ru.md) · [Українська](README.ua.md)

> **Status: Alpha.** The first release is `v0.1.0-alpha.1`, based on hostd `v2.12.1`. It runs on the author's mainnet hosts, but it is **not an official product of the Sia Foundation** and is not affiliated with it. Back up your host before installing. Use at your own risk.

## What it adds

hostd-extended keeps everything hostd does and adds tools for hosters who want to see who uses their host and how.

### Renters page
A new page with a table of renters, a world map and a side panel for the selected renter:

- contracts (active / total), stored data, first seen, last activity, last upload, last download;
- clients (accounts), IP addresses, countries, location, transport (SiaMux or QUIC) and the oldest protocol generation the client must support;
- filters (by status, renter key, IP address), pagination, sorting on every column;
- income per renter, in three groups: **All time**, **Received (month)** (what was paid this calendar month) and **Earned (month)** (payments spread over the period they were paid for), each split into storage, ingress, egress and total.

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
- the fork keeps its own tables and its own schema version, separate from hostd's; hostd's database schema is left untouched, so the original hostd can open the same database;
- `hostd fork status | export | import | remove` commands (see `hostd fork -h`);
- the built-in update check is disabled, so nobody installs the official build over hostd-extended by mistake.

There are **no new configuration options**: `hostd.yml`, ports and data directory are the same as in hostd. See the [hostd README](https://github.com/SiaFoundation/hostd#readme) for configuration.

## Requirements

Same as hostd `v2.12.1`. Alpha releases are published as a **Windows (amd64)** binary named `hostd.exe`, so it replaces the original file and existing shortcuts and services keep working. Other platforms have not been built or tested.

## Install

1. **Back up.** Stop hostd, then copy `hostd.db`, `hostd.yml` and `hostd.exe` from the data directory to a safe place. (Optionally, with `sqlite3.exe`: `sqlite3.exe -readonly hostd.db "VACUUM INTO 'backup.db';"` makes a compact consistent copy.)
2. **Replace** `hostd.exe` with the one from the release.
3. **Start** hostd as usual. On the first start the fork tables are created; it takes seconds.
4. **Check:** `hostd.exe version` prints `v0.1.0-alpha.1`; at log level `info` the log shows `fork migration complete`; in the browser press `Ctrl+F5` and open the **Renters** page.

Tested path: official hostd `v2.12.1` → hostd-extended. If you run an older hostd, update to `v2.12.1` first.

## Going back to the official hostd

Stop hostd and replace `hostd.exe` with the official one. The database opens as it is. The fork tables stay in the database and are not used; nothing is collected while the official hostd runs, and block lists no longer apply. To remove the fork tables, run `hostd fork export` (to keep the data) and then `hostd fork remove`, with hostd stopped.

## Data and privacy

- hostd-extended stores, locally, counters per renter and hour (kept 180 days), daily contract snapshots and links between IP addresses and renters. This includes renter keys and IP addresses.
- Nothing is sent anywhere. The only network request added is a one-time download of the GeoLite2 City database used to look up locations locally. IP addresses are never sent out.
- A file made by `hostd fork export` contains renter keys and IP addresses: keep it private.

## Known limitations

- Statistics start at installation; there is no history before it. Values marked with `~` cover an incomplete period.
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

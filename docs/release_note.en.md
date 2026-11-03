# hostd-extended v0.1.0-alpha.1

[English] · [Русский](release_note.ru.md) · [Українська](release_note.ua.md)

**First Alpha release.** November 2026.

- Based on: hostd `v2.12.1` (coreutils `v0.25.3`)
- Platform: Windows (amd64)
- Not an official product of the Sia Foundation. **Back up your host before installing.**

## Highlights

- **Renters page**: a table of renters, a world map and a side panel; income per renter (All time, Received this month, Earned this month); activity, clients, IP addresses, countries, transport; filters, pagination and sorting.
- **Quality of transfers** over the last 10 days: drop rate, download speed, upload speed.
- **Error counters** per renter, split into client side and host side, with a check that tells a sector the host never stored from a sector it lost.
- **Blocking** of renters and IP addresses, with a reason: from the **Manage lists** dialog or the row menu. Blocking never deletes data, ends contracts or returns money.
- **Contracts and Volumes**: row size (including a compact one) and number font, income column groups and sorting on Contracts, a thicker horizontal scrollbar, remembered column choice, filters and column groups, shorter relative times.

## Under the hood

- The fork keeps its own tables and schema version, separate from hostd's. The original hostd can open the same database.
- New commands: `hostd fork status | export | import | remove`.
- The built-in update check is disabled.
- No new configuration options.

## Upgrade notes

1. Stop hostd and copy `hostd.db`, `hostd.yml` and `hostd.exe` to a safe place.
2. Replace `hostd.exe` with the one from this release and start hostd. The fork tables are created on the first start (seconds).
3. Check: `hostd.exe version` prints `v0.1.0-alpha.1`; press `Ctrl+F5` in the browser and open **Renters**.

Tested path: official hostd `v2.12.1` → hostd-extended. If you run an older hostd, update to `v2.12.1` first.

**Going back:** stop hostd and put the official `hostd.exe` back; the database opens as it is.

## Known limitations

- Statistics start at installation; values marked `~` cover an incomplete period.
- Download speed is measured on the host and is an upper estimate for fast clients.
- A renter block applies to accounts the host has already linked to that renter; blocking an IP address affects every renter behind it.
- The first start resets the **View** column choice once.
- Tested only on two Windows mainnet hosts of the author.

## Files

| File | SHA-256 |
|---|---|
| `hostd-extended-v0.1.0-alpha.1-windows-amd64.zip` (contains `hostd.exe`) | `4A0F3BF27058CFB3C8C0727825188CD80AD2936AE020839A0ACC8D9CEC8983C6` |

## Credits

hostd is developed by the [Sia Foundation](https://sia.tech) (MIT). This product includes GeoLite2 data created by MaxMind, available from [https://www.maxmind.com](https://www.maxmind.com).

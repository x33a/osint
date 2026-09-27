# Tracking and transactions

Techniques for public ledgers and position data.

- [Bitcoin transactions](#bitcoin-transactions)
- [Past flights (ADS-B)](#past-flights-ads-b)

---

## Bitcoin transactions

Every Bitcoin transaction is public, so a block explorer shows an address's full history.

**Easiest:** search the address on **[Blockchair](https://blockchair.com)**. The summary shows *first seen* and *last seen* as normal dates.

**In the browser:** on [mempool.space](https://mempool.space), transactions are listed newest first. Scroll down and click *Load more* until it stops — the last one is the first transaction.

**From the command line** (mempool.space API, no login):

```bash
# newest 25 transactions
curl -s https://mempool.space/api/address/<ADDRESS>/txs \
  | jq '.[] | {txid, time: .status.block_time}'

# page further back using the last txid you saw; [] means you've reached the start
curl -s https://mempool.space/api/address/<ADDRESS>/txs/chain/<LAST_TXID> \
  | jq '.[] | {txid, time: .status.block_time}'

# convert the timestamp
date -u -d @<timestamp> +"%B %-d, %Y %H:%M UTC"
```

- The time is when the transaction was **confirmed in a block**, not when it was sent.
- Block timestamps can be off by up to about two hours.
- Explorers show local time; the API gives UTC. Near midnight, the date can differ.

---

## Past flights (ADS-B)

Aircraft broadcast their position (ADS-B), and archives let you replay flights years later. Identify the aircraft first.

1. **Registration:** read it from photos or video — it's painted on the rear fuselage and under the wings. Prefixes identify the country (e.g. `A6-` = UAE, `N` = US, `G-` = UK).
2. **No registration visible?** Search [planespotters.net](https://www.planespotters.net) or [jetphotos.com](https://www.jetphotos.com) for the event; spotters tag registration and date. Distinctive liveries narrow it down fast.
3. **Replay:** on **[ADS-B Exchange](https://globe.adsbexchange.com)**, search the registration, open history and set the date. The track shows departure and arrival.
4. Check the event's official schedule for the likely date (a video's publish date may be days after the flight).

Flightradar24 needs a paid plan for flights older than 7 days; ADS-B Exchange is free. [Flightera](https://www.flightera.net) sometimes lists an aircraft's past flights by date for a quick cross-check.

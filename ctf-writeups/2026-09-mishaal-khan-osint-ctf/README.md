# OSINT CTF Writeup — September 26, 2026

A learning log from a 30-challenge OSINT (open-source intelligence) CTF, covering the question behind each challenge, the technique that cracks it, and the tools used. The focus is on *how* to find things, so the approaches are written to be reusable on other investigations.

**Final score:** 27 of 30 confirmed · 4,250 of 4,890 points · 2 unconfirmed · 1 unsolved

> **Spoiler policy:** where an answer is included, it sits inside a collapsed "Show answer" block. If you're working a similar challenge, read the *Approach* sections and leave the answers closed.

## Credits

This CTF was created and run by **[Mishaal Khan](https://www.mishaalkhan.com/)**, and announced on his [LinkedIn](https://www.linkedin.com/in/mish-aal/). All challenges and questions are his; this writeup only documents how I approached them. Thanks to Mishaal for putting together a fun, well-designed set of challenges.

---

## Contents

- [Credits](#credits)
- [Scoreboard](#scoreboard)
- [API](#api) · [Breaches](#breaches) · [Domain](#domain) · [Email](#email) · [Finance](#finance) · [Geo](#geo) · [Gov](#gov) · [Image](#image) · [Network](#network) · [Social](#social) · [Vehicles](#vehicles)
- [Lessons learned](#lessons-learned)
- [Tools and related notes](#tools-and-related-notes)

---

## Scoreboard

| Category | Challenge | Points | Status |
| --- | --- | ---: | --- |
| API | Github | 350 | ✅ Solved |
| API | Time | 220 | ✅ Solved |
| Breaches | How could you! | 100 | ✅ Solved |
| Breaches | Plates | 70 | ✅ Solved |
| Domain | First | 50 | ✅ Solved |
| Domain | Legal Ownership | 120 | ✅ Solved |
| Domain | No Privacy! | 80 | ✅ Solved |
| Domain | The internet never forgets | 60 | ✅ Solved |
| Email | Provider | 120 | ✅ Solved |
| Email | Tooth | 300 | ✅ Solved |
| Finance | First! | 250 | ✅ Solved |
| Finance | Magic Money | 150 | ✅ Solved |
| Geo | Big Plane | 250 | ⚠️ Unconfirmed (timed out) |
| Geo | Earthlings | 200 | ✅ Solved |
| Geo | Smashing! | 300 | ✅ Solved |
| Geo | Yummy | 190 | ✅ Solved |
| Gov | Doctor Who? | 150 | ✅ Solved |
| Gov | Fraud | 150 | ✅ Solved |
| Gov | Registered to vote | 150 | ❌ Unsolved |
| Image | 316 | 200 | ✅ Solved |
| Image | Easy as Pi | 240 | ⚠️ Unconfirmed |
| Image | Phone A Friend | 80 | ✅ Solved |
| Image | Viral | 130 | ✅ Solved |
| Network | IP | 60 | ✅ Solved |
| Network | MFA | 200 | ✅ Solved |
| Network | Return of the MAC | 150 | ✅ Solved |
| Social | Edu | 80 | ✅ Solved |
| Social | Kids | 100 | ✅ Solved |
| Vehicles | VIN | 250 | ✅ Solved |
| Vehicles | Vroom | 140 | ✅ Solved |

---

## API

> Techniques: [Git commit emails](../../techniques/email-and-accounts.md#git-commit-emails) · [Proton account age](../../techniques/email-and-accounts.md#proton-account-age)

### Github — 350

**Question:** What's the email address of Vijayasingam in the ProtonMail WebClients GitHub repository?

**Approach:** Every git commit records the author's name and email, and anyone can read them on a public repo.

1. Open `github.com/ProtonMail/WebClients/commits` and filter by author (the "All users" dropdown, or `?author=<username>`).
2. Open one of their commits and append **`.patch`** to the URL. The `From:` line shows the author's name and email.
3. Alternative — a blobless clone keeps it fast:
   ```bash
   git clone --filter=blob:none --no-checkout https://github.com/ProtonMail/WebClients
   cd WebClients
   git log --all --format='%an <%ae>' | grep -i vijayasingam | sort -u
   ```
4. A `@users.noreply.github.com` address means the email is hidden — check older commits.

### Time — 220

**Question:** Using Proton's key server API, on what date was a given Proton address issued a certificate (indicating how old the account is)? Format: `January 1, 2026`.

**Approach:** Proton creates a PGP key when an account is set up, so the key's creation date approximates the account's age.

```bash
curl "https://api.protonmail.ch/pks/lookup?op=index&search=user@proton.me"
```

The output is HKP format. On the `pub:` line, the 5th colon-separated field is the creation date as a Unix timestamp:

```
pub:<fingerprint>:<algorithm>:<length>:<CREATED>:<expires>:
```

Convert it (use `-u` for UTC, to avoid an off-by-one day near midnight):

```bash
date -u -d @1650000000 +"%B %-d, %Y"
```

With several `pub` lines, the oldest timestamp reflects the account's age. `info:1:0` means no key was found.

---

## Breaches

> Techniques: [Breach lookups](../../techniques/email-and-accounts.md#breach-lookups)

### How could you! — 100

**Question:** Which data breach was so bad it was linked to deaths?

**Approach:** Well-known breach history, cross-checked on [Have I Been Pwned](https://haveibeenpwned.com), which lists every breach an email appears in along with the exposed data types.

<details>
<summary>Show answer</summary>

**Ashley Madison (2015).** The leak of ~30 million users of the affair-dating site was linked to several suicides.

</details>

### Plates — 70

**Question:** Which breach included vehicle license plates?

**Approach:** Check HIBP's "compromised data" types for each breach and look for license plates.

<details>
<summary>Show answer</summary>

**ParkMobile (2021)** — ~21 million users of the parking app; exposed data included license plate numbers.

</details>

---

## Domain

> Techniques: [Domains and websites](../../techniques/domains-and-websites.md)

### First — 50

**Question:** When was the domain registered?

**Approach:**

```bash
whois example.com | grep -i creat
# or RDAP (JSON):
curl -s https://rdap.verisign.com/com/v1/domain/example.com | jq '.events'
```

A creation date newer than expected can mean the domain was dropped and re-registered — i.e., there was an earlier owner.

### Legal Ownership — 120 · No Privacy! — 80

**Question:** Who owned the domain previously, and what do the registrant details show without privacy protection?

**Approach:** Regular `whois` only shows the current record; historical whois services keep snapshots.

- **[whoxy.com](https://www.whoxy.com)** — free whois history.
- **ViewDNS** no longer offers whois history, but its **IP History** (hosting changes) and **Reverse Whois** (other domains owned by the same name/email) still help.
- **SecurityTrails** — historical DNS with a free account.
- **Wayback Machine** — old copies of the site often name the owner.

Since GDPR (2018), most registrant data is redacted, so real names usually appear only in older records.

### The internet never forgets — 60

**Question:** What was the version number listed in the footer of the data broker site radaris.com?

**Approach:** The live site had been replaced with a placeholder page, so the old footer only exists in the **Wayback Machine** — hence the challenge title.

1. Open `web.archive.org/web/*/radaris.com` and pick a snapshot from before the site changed.
2. View source (`Ctrl+U`) and search for `ver`, `version` or `footer`.
3. Add `id_` after the timestamp (`/web/20260909152343id_/https://radaris.com/`) to strip Wayback's own injected code.

**Red herring:** `"version":"2024.11.0"` inside `data-cf-beacon` belongs to Cloudflare's analytics script, which appears on every Cloudflare-hosted site.

<details>
<summary>Show answer</summary>

**14126090** — found in the source of the September 9, 2026 snapshot.

</details>

---

## Email

> Techniques: [Email provider (MX)](../../techniques/domains-and-websites.md#email-provider-mx-records) · [Email to Google account](../../techniques/email-and-accounts.md#email-to-google-account)

### Provider — 120

**Question:** Who is the email provider for mishaalkhan.com?

**Approach:** MX records name the servers that receive a domain's mail.

```bash
dig MX mishaalkhan.com +short
```

Or in a browser: `https://dns.google/resolve?name=mishaalkhan.com&type=MX`. Common patterns: `protonmail.ch` → Proton Mail, `google.com` → Google Workspace, `outlook.com` → Microsoft 365.

<details>
<summary>Show answer</summary>

**Proton Mail** (`mail.protonmail.ch`, `mailsec.protonmail.ch`).

</details>

### Tooth — 300

**Question:** What is the address of the dentist that ironman@gmail.com recently left a Google review at? (House number only.)

**Approach:** Google search can't link an email to a review — reviews show a profile name, not an email. But Google services can resolve an email to the account's internal **Gaia ID**, and that ID opens the public Maps contributions page:

```
https://www.google.com/maps/contrib/<GAIA_ID>/reviews
```

- **[Epieos](https://epieos.com)** — no install; returns the Gaia ID and Maps link.
- **[GHunt](https://github.com/mxrch/GHunt)** — `ghunt email ironman@gmail.com` (see [tools](../../tools/README.md) for setup).

Open the Reviews tab, find the dentist, and click through to its address.

---

## Finance

> Techniques: [Bitcoin](../../techniques/tracking-and-transactions.md#bitcoin-transactions) · [PayPal names](../../techniques/email-and-accounts.md#paypal-account-names)

### First! — 250

**Question:** When was the first transaction on a given Bitcoin address?

**Approach:** Every Bitcoin transaction is public.

- **Easiest:** search the address on **[Blockchair](https://blockchair.com)** — the summary shows *first seen* as a normal date.
- **Browser:** on [mempool.space](https://mempool.space), scroll to the bottom and click *Load more* until it stops.
- **CLI:**
  ```bash
  curl -s https://mempool.space/api/address/<ADDRESS>/txs | jq '.[] | {txid, time: .status.block_time}'
  # page back with /txs/chain/<last_txid> until you get []
  date -u -d @<timestamp> +"%B %-d, %Y %H:%M UTC"
  ```

Times are when the transaction was confirmed in a block (not sent), and block times can be off by up to ~2 hours.

### Magic Money — 150

**Question:** Whose name is the PayPal address magictricksrevealedofficial@gmail.com registered to?

**Approach:** PayPal has no public lookup API or CLI tool that shows names. What worked:

1. Log in and open **Send & Request** (`paypal.com/myaccount/transfer/homepage`), or tap Send in the app.
2. Type the **full** email into the search box. If the profile is public, the dropdown shows the account name and photo.
3. Stop there — don't enter an amount.

`paypal.me/<username>` also shows a display name without logging in, if you can guess the username.

---

## Geo

> Techniques: [Image and geolocation](../../techniques/image-and-geolocation.md) · [Flights](../../techniques/tracking-and-transactions.md#past-flights-ads-b) · [Property](../../techniques/public-records.md#property-ownership)

### Big Plane — 250

**Question:** An Emirates Airbus A380 flew at the 2021 Dubai Airshow in a YouTube video published November 16, 2021. Where did that flight depart from?

**Approach:** Aircraft broadcast their position (ADS-B), and archives let you replay flights years later.

1. Read the registration from the video. Emirates registrations start with `A6-E…`; the frame showed "6-EVO", so **A6-EVO** (an A380, MSN 268).
2. Note that the static-display A380 that year was a different aircraft (A6-EVQ), so A6-EVO was likely a flying-display visitor. The airshow opened with an Emirates A380 flypast on **November 14, 2021**.
3. Replay it on **[ADS-B Exchange](https://globe.adsbexchange.com)**: search the registration, open history, set the date, and follow the track back to takeoff.

Flightradar24 needs a paid plan for flights older than 7 days; ADS-B Exchange is free.

<details>
<summary>Show answer</summary>

**Los Angeles** *(unconfirmed — the question timed out before submitting)*.

</details>

### Earthlings — 200

**Question:** What does it say at `51.48449044020304, 0.08799148642372842`?

**Approach:**

1. Paste the coordinates into Google Maps. The spot is in Plumstead, southeast London.
2. Switch to **Satellite** and zoom in: the text is painted on a rooftop.
3. Google Maps' imagery was too blurry and the pin covered part of the text. **[Google Earth](https://earth.google.com)** had sharper imagery and a smaller marker — rotate until the text reads left to right.
4. Other options: Bing/Apple Maps aerial imagery, or Google Earth Pro's historical imagery.

<details>
<summary>Show answer</summary>

**"What are you fuckin lookin at"** — exact spelling matters; an earlier guess of "…fucks looking at" was rejected.

</details>

### Smashing! — 300

**Question:** Which famous person owns a given house in Malibu, CA? *(Answer format: 3 names, not the same spelling as the name on file.)*

> The exact address is left out of this writeup to avoid publicly tying a named person to their home.

**Approach:**

1. Listing history (Zillow, Redfin, Compass) gave the sale date and MLS number but no owner.
2. A plain Google search of the address surfaced the owner.
3. The challenge title confirmed it: **"Smashing!"** refers to the 2019 Cybertruck unveiling, where the armored windows famously smashed on stage.
4. The answer-format hint means property records spell the name differently from the correct public spelling — submit the correct one.

<details>
<summary>Show answer</summary>

**Franz von Holzhausen**, Tesla's chief designer.

</details>

### Yummy — 190

**Question:** Which country is this fridge in?

**Approach:** Read the labels.

1. Script first — Arabic on nearly every product.
2. Local brands: **Almarai** and **NADEC** are both Saudi dairy companies; two together is strong evidence.
3. International brands (Alpro, Heinz) don't narrow it down.
4. Confirm with barcodes if visible — Saudi EAN prefixes start with **628**.

<details>
<summary>Show answer</summary>

**Saudi Arabia.**

</details>

---

## Gov

> Techniques: [Public records](../../techniques/public-records.md)

### Doctor Who? — 150

**Question:** In which state does "Jane Doe" practice as a licensed medical healthcare provider?

**Approach:** Every US provider who bills insurance has a public NPI record.

```
https://npiregistry.cms.hhs.gov/api/?version=2.1&first_name=Jane&last_name=Doe
```

Or search by name at [npiregistry.cms.hhs.gov](https://npiregistry.cms.hhs.gov).

<details>
<summary>Show answer</summary>

**Florida.**

</details>

### Fraud — 150

**Question:** How many jobs did "Black Angus Steakhouses LLC" in California report to get a $10 million PPP loan?

**Approach:** The SBA published every PPP loan; ProPublica made them searchable.

1. Search the business at [projects.propublica.org/coronavirus/bailouts](https://projects.propublica.org/coronavirus/bailouts/).
2. Match by city, state and amount — this company had two loans ($2M and $10M).
3. The loan page lists jobs reported, lender, approval date and forgiveness.

<details>
<summary>Show answer</summary>

**63 jobs** — $10,000,000 loan approved April 10, 2020 (Sherman Oaks, CA; lender Stearns Bank).

</details>

### Registered to vote — 150 · ❌ Unsolved

**Question:** Where is a given celebrity registered to vote? (House number only.)

Not solved. The answer is effectively a real person's home address, so this writeup doesn't cover how to find it.

---

## Image

> Techniques: [Image and geolocation](../../techniques/image-and-geolocation.md)

### 316 — 200

**Question:** Which celebrity also has this studded skull patch?

**Approach:** Reverse image search — and read the challenge title as a clue.

1. **Google Lens** with the image *plus* the question text and "316" as keywords found it.
2. "316" points to **"Austin 3:16"**, a famous wrestling catchphrase whose logo is a skull.

<details>
<summary>Show answer</summary>

**Stone Cold Steve Austin.**

</details>

### Easy as Pi — 240 · ⚠️ Unconfirmed

**Question:** In which country was this photo taken? (Raspberry Pi B+ boxes and Pibow cases in a shipping box.)

**Approach:** The obvious clues all pointed the wrong way.

- **Product origin:** Pibow cases say "Designed & made in Sheffield, UK"; Raspberry Pis are British → **UK was wrong.**
- **Barcode:** a UPC starting with `0` → US/Canada → **both wrong.**
- **Metadata:** the file was a 779×1038 PNG screenshot, so all EXIF/GPS had been stripped.

What worked was **finding the original poster**: an AI-assisted image search surfaced a 2014 Reddit post ("Pibow for my B+ arrived") by user *bjornkeizers*, where other users call him a "fellow dutchie."

**Lesson:** product origin, barcodes and label language show where goods were *made or registered*, not where a photo was taken.

<details>
<summary>Show answer</summary>

**The Netherlands** *(unconfirmed)*.

</details>

### Phone A Friend — 80

**Question:** What brand is the phone in this photo?

**Approach:** The image was too small to read a logo. Shape analysis (slim glossy bar/slider, metallic end band) narrowed the era to ~2007–2009 luxury phones. Nokia and Samsung were both wrong. The reliable route is a reverse image search for a larger original of the press photo.

### Viral — 130

Not covered in this writeup.

---

## Network

> Techniques: [Finding an organization's vendors](../../techniques/domains-and-websites.md#finding-an-organizations-vendors)

### IP — 60 · Return of the MAC — 150

Not covered in this writeup.

### MFA — 200

**Question:** Which vendor's MFA product does the City and County of Denver use?

**Approach:** Organizations leak their vendors through help pages, DNS and job listings.

1. **Dorks:** `site:denvergov.org MFA`, `site:denvergov.org "multi-factor authentication"`, `site:denvergov.org Duo OR Okta OR "Microsoft Authenticator"`.
2. **DNS TXT records:** `dig TXT denvergov.org +short` — look for `duo_sso_verification`, `okta-verification`, `MS=`.
3. **Subdomains:** search `%.denvergov.org` on [crt.sh](https://crt.sh) for names like `okta.`, `duo.`, `sso.`.
4. **Job postings:** IT listings often name the products staff must know.

Two matching sources give a confident answer.

---

## Social

> Techniques: [Facebook IDs](../../techniques/email-and-accounts.md#facebook-numeric-ids) · [Planted profile details](../../techniques/email-and-accounts.md#planted-profile-details)

### Edu — 80

**Question:** "In 1921, what school did I attend?" — asked by the CTF author, Mishaal Khan.

**Approach:** An impossible date means a **planted** detail. CTF authors often plant jokes on their own profiles.

1. Start from the author's site and collect their social links.
2. Check **LinkedIn → Education** (sign in and click "Show all"). The answer is the entry dated 1921.

### Kids — 100

**Question:** What is the numeric Facebook ID of Ms Rachel?

**Approach:**

1. Confirm the official, verified page first.
2. **About → Page transparency** shows the Page ID.
3. Or view source (`Ctrl+U`) and search for `"userID"`, `"pageID"`, `delegate_page_id` or `profile_id`.
4. Verify: `facebook.com/<ID>` should open the same page.

---

## Vehicles

> Techniques: [Plate to VIN](../../techniques/public-records.md#license-plate-to-vin)

### VIN — 250

**Question:** Look up the VIN for Florida plate **KHAN**.

**Approach:** A VIN identifies the vehicle, not the owner (owner data is protected by the Driver's Privacy Protection Act).

1. **[FaxVIN](https://www.faxvin.com)** → search by license plate → state: Florida → `KHAN`.
2. Confirm with the **[NHTSA VIN decoder](https://vpic.nhtsa.dot.gov/decoder/)** — make, model and year should match.
3. A VIN is 17 characters and never contains I, O or Q.

### Vroom — 140

Not covered in this writeup.

---

## Lessons learned

- **Read the challenge title.** "Smashing!", "316" and "The internet never forgets" each pointed straight at the answer.
- **Product origin ≠ photo location.** Labels, barcodes and "made in" text tell you where goods came from, not where the camera was.
- **Screenshots strip metadata.** If the file is a screenshot, EXIF won't help — go find the original post.
- **Sharper imagery exists.** When Google Maps is blurry, try Google Earth, Bing or Apple Maps.
- **Beware red herrings in source code.** Third-party scripts (Cloudflare, analytics) carry their own version numbers.
- **Exact spelling matters** — for rooftop text, names and anything free-text.

---

## Tools and related notes

- **Tool setup, GHunt login, troubleshooting and cleanup:** [tools/](../../tools/README.md)
- **Reusable technique pages** built from this CTF: [techniques/](../../techniques/README.md)
- **Other writeups:** [ctf-writeups/](../README.md)

# Email and accounts

Techniques for turning an email address or username into accounts, names and history.

- [Email to Google account](#email-to-google-account)
- [Git commit emails](#git-commit-emails)
- [Proton account age](#proton-account-age)
- [Breach lookups](#breach-lookups)
- [PayPal account names](#paypal-account-names)
- [Facebook numeric IDs](#facebook-numeric-ids)
- [Planted profile details](#planted-profile-details)

---

## Email to Google account

Search engines can't link an email to Google reviews, because reviews show a profile name, not an email. But Google services can resolve an email to the account's internal **Gaia ID**, and that ID opens the account's public Maps contributions:

```
https://www.google.com/maps/contrib/<GAIA_ID>/reviews
```

**Tools**

- **[Epieos](https://epieos.com)** — web-based, no install. Returns the Gaia ID and a Maps link.
- **[GHunt](https://github.com/mxrch/GHunt)** — command line. `ghunt email <address>`. Setup and login notes are in [tools](../tools/README.md#ghunt).

**Steps**

1. Look up the email with Epieos or GHunt.
2. Open the Maps contributions link and go to the Reviews tab.
3. Click a reviewed business to see its address.

If the profile shows no reviews, the account may have made its contributions private.

---

## Git commit emails

Every git commit records the author's name and email, and anyone can read them on a public repository.

**In the browser**

1. Open `github.com/<owner>/<repo>/commits` and filter by author (the "All users" dropdown, or `?author=<username>`).
2. Open a commit and append **`.patch`** to its URL. The `From:` line shows the name and email.

**From the command line**

```bash
git clone --filter=blob:none --no-checkout https://github.com/<owner>/<repo>
cd <repo>
git log --all --format='%an <%ae>' | grep -i <name> | sort -u
```

A blobless clone downloads history without file contents, so it's fast even on large repos.

**API**

```bash
curl -s "https://api.github.com/repos/<owner>/<repo>/commits?author=<username>" | jq '.[].commit.author'
```

A `@users.noreply.github.com` address means the author hides their email. Older commits often still show a real one.

---

## Proton account age

Proton creates a PGP key when an account is set up, and its public key server shows the key's creation date — a good proxy for the account's age.

```bash
curl "https://api.protonmail.ch/pks/lookup?op=index&search=user@proton.me"
```

The response is HKP format. On the `pub:` line, the fifth colon-separated field is the creation date as a Unix timestamp:

```
pub:<fingerprint>:<algorithm>:<length>:<CREATED>:<expires>:
```

Convert it:

```bash
date -u -d @1650000000 +"%B %-d, %Y"
```

- Use `-u` (UTC) so a key created near midnight doesn't land a day off.
- Several `pub` lines = the user made new keys over time. The oldest shows the account's age.
- `info:1:0` = no key found. Check the spelling.

---

## Breach lookups

**[Have I Been Pwned](https://haveibeenpwned.com)** lists every known breach that contained an email, and the data types exposed in each (passwords, phone numbers, license plates, and so on).

- Search the email, then read each breach's "compromised data" list.
- To answer "which breach exposed X", match X against those data types.

Well-known examples:

| Breach | Year | Notable because |
| --- | --- | --- |
| Ashley Madison | 2015 | ~30M users; linked to several suicides |
| ParkMobile | 2021 | ~21M users; included license plate numbers |
| Vastaamo (Finland) | 2020 | Therapy notes stolen; patients blackmailed |

---

## PayPal account names

PayPal has no public lookup API, and no command-line tool can show the name on an account. What works:

1. Log in and open **Send & Request** (`paypal.com/myaccount/transfer/homepage`), or tap Send in the app.
2. Type the **full** email into the recipient search box. If the profile is public, the dropdown shows the account's name and photo.
3. Stop there. Don't enter an amount or confirm anything.

- No result = the profile is hidden, or there's no account.
- `paypal.me/<username>` shows a display name without logging in, if you can guess the username.
- [holehe](https://github.com/megadose/holehe) can check whether an email is registered on many sites, but never shows names.

---

## Facebook numeric IDs

Every Facebook profile and page has a numeric ID behind its username.

1. Make sure it's the official, verified page.
2. **Pages:** About → Page transparency shows the Page ID.
3. **Page source:** `Ctrl+U`, then search for `"userID"`, `"pageID"`, `delegate_page_id` or `profile_id`.
4. **Profile photo URL:** look for `fbid=` or `id=`.
5. **Verify:** `facebook.com/<ID>` should open the same page.

---

## Planted profile details

CTF authors often plant joke details on their own public profiles. An impossible detail (school attended in 1921, a location that doesn't match anything else) means look for a deliberate entry, not real history.

1. Start from the author's own site and collect their social links.
2. Check LinkedIn **Education** and **Experience** (signed in, with "Show all" expanded), Facebook's **About → Work and education**, and bios on X or Mastodon.
3. If an entry has been removed, check Wayback Machine snapshots of the profile.

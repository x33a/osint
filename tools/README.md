# Tools

My OSINT toolkit: what's installed, how to set it up, fixes for problems I hit, and how to remove it all.

Set up on **CachyOS** (Arch-based, system Python 3.14, fish shell). Commands for other Arch-based distros are the same.

- [Command-line tools](#command-line-tools)
- [GHunt](#ghunt)
- [Web tools](#web-tools)
- [Troubleshooting](#troubleshooting)
- [Cleanup](#cleanup)

---

## Command-line tools

| Tool | Install | Purpose | Remove |
| --- | --- | --- | --- |
| uv | `sudo pacman -S uv` | Python tool and version manager | `sudo pacman -Rns uv` |
| Python 3.12 (uv-managed) | pulled in by the GHunt install | GHunt's dependencies don't build on 3.14 | `uv python uninstall 3.12` |
| [GHunt](https://github.com/mxrch/GHunt) | `uv tool install ghunt --python 3.12` | Google account OSINT | `uv tool uninstall ghunt` |
| whois | `sudo pacman -S whois` | Domain registration lookups | `sudo pacman -Rns whois` |
| jq | `sudo pacman -S jq` | Reading JSON from APIs | `sudo pacman -Rns jq` |
| exiftool | `sudo pacman -S perl-image-exiftool` | Image metadata (EXIF, GPS) | `sudo pacman -Rns perl-image-exiftool` |
| dig | `sudo pacman -S bind` (if missing) | DNS lookups (MX, TXT) | — |

Check what's installed: `pacman -Q uv whois jq perl-image-exiftool` and `uv tool list`.

---

## GHunt

[GHunt](https://github.com/mxrch/GHunt) looks up Google accounts from an email: Gaia ID, profile, and the link to public Maps reviews.

**Install**

```bash
sudo pacman -S uv
uv tool install ghunt --python 3.12
```

If `ghunt` isn't found afterwards (common on fish): `fish_add_path ~/.local/bin`, then reopen the terminal.

**Login without the browser extension**

GHunt needs a logged-in Google session. Use a **throwaway** Google account, never your main one.

1. In a private window, sign in at `https://accounts.google.com/EmbeddedSetup`. The page may hang afterwards — that's expected.
2. Open DevTools (F12) → Storage/Application → Cookies → `https://accounts.google.com`.
3. Copy the value of the **`oauth_token`** cookie (starts with `oauth2_4/`).
4. Right away, run `ghunt login`, choose the oauth_token option, and paste it.

The token is single-use and short-lived — treat it like a password. The session is saved in `~/.malfrats/ghunt/`.

**Use**

```bash
ghunt email someone@gmail.com
```

Look for the Maps section in the output for the contributions link.

---

## Web tools

| Site | Use |
| --- | --- |
| [Epieos](https://epieos.com) | Email → Google account, no install |
| [Have I Been Pwned](https://haveibeenpwned.com) | Breaches containing an email |
| [whoxy](https://www.whoxy.com) | Whois history |
| [Wayback Machine](https://web.archive.org) | Old versions of websites |
| [crt.sh](https://crt.sh) | Subdomains from certificate logs |
| [Blockchair](https://blockchair.com) · [mempool.space](https://mempool.space) | Bitcoin addresses and transactions |
| [ADS-B Exchange](https://globe.adsbexchange.com) | Replaying past flights |
| [Google Earth](https://earth.google.com) | Sharper satellite imagery |
| [Google Lens](https://lens.google.com) · [Yandex](https://yandex.com/images) · [TinEye](https://tineye.com) | Reverse image search |
| [jimpl](https://jimpl.com) · [exif.tools](https://exif.tools) | EXIF metadata without a terminal |
| [NPI Registry](https://npiregistry.cms.hhs.gov) | US healthcare providers |
| [ProPublica PPP tracker](https://projects.propublica.org/coronavirus/bailouts/) | PPP loan details |
| [FaxVIN](https://www.faxvin.com) · [NHTSA decoder](https://vpic.nhtsa.dot.gov/decoder/) | Plate → VIN, and VIN decoding |

---

## Troubleshooting

| Error | Cause | Fix |
| --- | --- | --- |
| `externally-managed-environment` on `pip install --user` | Arch blocks pip installs into system Python | Install with pacman (`python-pipx`) or use `uv` |
| `ghunt: command not found` | `~/.local/bin` isn't on PATH (fish) | `fish_add_path ~/.local/bin` |
| Pillow fails to build (`_webp.c … incompatible-pointer-types`) | GHunt pins an old Pillow with no build for Python 3.14 | Install on Python 3.12: `uv tool install ghunt --python 3.12` |
| `KeyError: 'container'` in `parsers/people.py` | Google changed its profile data format | Patch the parser (below), or install the latest from GitHub: `uv tool install --force --python 3.12 git+https://github.com/mxrch/GHunt` |
| GHunt rejects the login token | `oauth_token` expired or already used | Sign in to EmbeddedSetup again and copy a fresh token |

Patch for the `KeyError: 'container'` crash (reinstalling GHunt undoes it):

```bash
sed -i 's/\["metadata"\]\["container"\]/["metadata"].get("container", "PROFILE")/g' \
  ~/.local/share/uv/tools/ghunt/lib/python3.12/site-packages/ghunt/parsers/people.py
```

---

## Cleanup

```bash
uv tool uninstall ghunt
uv python uninstall 3.12
rm -rf ~/.malfrats                         # GHunt session
sudo pacman -Rns whois uv perl-image-exiftool jq
```

Also sign out of, or delete, the throwaway Google account.

# DefCamp CTF (D-CTF) Finals — complete guide

Everything about the **D-CTF finals** (DefCamp Capture the Flag, Bucharest): explained task writeups for the Jeopardy finals 2015–2019, the Attack & Defense finals 2022 onward, plus how to prepare — a roadmap, tooling, teams, real scenarios, and a cross-year technique index.

> **Scope:** "dctf finals" = DefCamp's D-CTF finals. If you meant a different CTF, this needs redoing.

## Writeups by year

| Year | Format | File | Depth |
|------|--------|------|-------|
| 2015 | Jeopardy | [writeups/2015.md](writeups/2015.md) | 6 tasks explained in full |
| 2016 | Jeopardy | [writeups/2016.md](writeups/2016.md) | LeCrypto + Shredder (p4); SMS + Personality (other teams); full task list |
| 2017 | Jeopardy | [writeups/2017.md](writeups/2017.md) | 6 p4 tasks + Infinity, Security CCTV, Silent, spock-lizard (other teams) |
| 2018 | Jeopardy | [writeups/2018.md](writeups/2018.md) | authenticator (p4) + Scribbles & TicketCore (full) + ico |
| 2019 | Jeopardy | [writeups/2019.md](writeups/2019.md) | 4 tasks explained |
| 2020–2021 | no on-site finals (COVID, then online "DCTF 21-22") | — | n/a |
| 2022–2025 | **Attack & Defense** | [writeups/attack-defense.md](writeups/attack-defense.md) | Format, lessons, results (no service writeups found) |

## Preparation & reference guides

| Guide | What it covers |
|-------|----------------|
| [guides/ad-finals-guide.md](guides/ad-finals-guide.md) | How a D-CTF Attack & Defense final actually works — parts, scoring, timeline, year-by-year results |
| [guides/preparation-roadmap.md](guides/preparation-roadmap.md) | 12-week Jeopardy skill plan + full A/D prep: infra, roles, first hour, steady-state loop |
| [guides/tooling.md](guides/tooling.md) | Exploit farms, flag submitters, Tulip/Flower traffic analysis, firewalling, patching, practice envs |
| [guides/teams-and-groups.md](guides/teams-and-groups.md) | Who documented what, A/D-era finalists, and where the missing writeups likely live |
| [guides/real-scenarios.md](guides/real-scenarios.md) | 11 "you are here → what you'd do" worked examples from real D-CTF tasks |
| [guides/techniques-index.md](guides/techniques-index.md) | Every technique in this repo, grouped by category, linked to its task |

## Per-category study packs

Full skill-builders per CTF category (techniques + tools + 100 scripts + roadmap), in [challenges/](challenges/):

| Category | File |
|----------|------|
| Crypto | [challenges/crypto/Crypto-Complete.md](challenges/crypto/Crypto-Complete.md) |
| Pwn | [challenges/pwn/Pwn-Complete.md](challenges/pwn/Pwn-Complete.md) |
| Reverse | [challenges/reverse/RE-Complete.md](challenges/reverse/RE-Complete.md) |
| Forensics | [challenges/forensics/Forensics-Complete.md](challenges/forensics/Forensics-Complete.md) |
| Stego (under forensics) | [challenges/forensics/stego/Stego-Complete.md](challenges/forensics/stego/Stego-Complete.md) |

## How reliable is this? (read before relying on it)

- **Read in full:** every p4-team writeup for 2015–2019 (from [p4-team/ctf](https://github.com/p4-team/ctf)), and the **2018 Scribbles + TicketCore** writeups (from the [balsn](https://github.com/balsn/ctf_writeup) and [w181496](https://github.com/w181496/CTF) GitHub mirrors). Those explanations are paraphrases of what the writeups say.
  One claim was re-checked by hand (2015 reverse 200: 961749023 is prime, digit sum 41) plus two small facts (key length 21, `0x20 ^ 0x27 = 0x07`). **No exploit was re-run.**
- **Search excerpts only, pages not opened:** the 2016 SMS/Personality details, the 2017 Infinity / Security CCTV / Silent / spock-lizard notes, the 2018 *ico* task, and all Attack & Defense material. Each is labelled in its file.
- **Why the mix:** this environment's network policy reached **GitHub + web-search** but **not** `ctftime.org`, `def.camp`, or most team blogs directly (`WebFetch`/`curl` to them failed). GitHub public repos were cloneable, which is how the 2018 web writeups got full depth.
- p4 (and each team) only documented the tasks *they solved*, so these are not complete task lists per year.
- Writeups are **explained and linked, not copied**; credit stays with the original authors. **Challenge flags and verbatim final exploit payloads are deliberately omitted** — follow the source links for those.

## Remaining gaps

1. **A/D service writeups (2022–2025)** — none found; a best-writeup prize exists since 2023, so they likely do. See [teams-and-groups.md](guides/teams-and-groups.md#where-to-hunt-for-the-still-missing-writeups).
2. Other teams' Jeopardy writeups still only linked (2016 WildImage/Personality content, 2017 spock-lizard beta/omega, 2018 remaining tasks).
3. The 2018 *ico* technique, and confirming its year (author URL says 2019; CTFtime files it under 2018).
4. The "unverified"/"conflict" items for 2025 in [attack-defense.md](writeups/attack-defense.md).

To close these, allow `ctftime.org`, `def.camp` and the team blogs in the cloud environment's **network settings** (environment menu → Edit → Network access / Allowed domains) and start a new session.

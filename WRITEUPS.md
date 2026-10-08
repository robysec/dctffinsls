# DefCamp CTF (D-CTF) Finals — writeup research index

Research index of public writeups and recaps for the **D-CTF finals** (DefCamp Capture the Flag, Bucharest),
covering both the **Jeopardy-style** finals (2016–2019) and the **Attack & Defense** finals (2022 onward).

> **Assumption:** "dctf finals" = DefCamp's D-CTF finals. If you meant a different CTF, tell me and this file needs redoing.

## How this was built (read this first)

- Compiled from web-search results on **2026-10-08**. The environment's network policy blocked direct access to
  `ctftime.org`, `def.camp` and most blog hosts, so **no linked page was opened and read in full**.
  Every summary below is derived from search-result excerpts of those pages.
- Technique descriptions are short paraphrases from those excerpts, **not** full solutions. To get the real step-by-step
  solution, open the linked original. Nothing here has been re-solved or re-run.
- Items marked **(unverified)** or **(conflict)** need checking against the source before you rely on them.
- Writeups are **linked, not copied**, so credit and copyright stay with the original authors.

## Overview

| Year | Finals format | Date | Writeups found |
|------|---------------|------|----------------|
| 2016 | Jeopardy, on-site | 10–11 Nov | None found |
| 2017 | Jeopardy, on-site | 9–10 Nov | Yes — several, per-challenge |
| 2018 | Jeopardy, on-site | 8–9 Nov | Yes — a few |
| 2019 | Jeopardy, on-site | 7–8 Nov | Yes — several, per-challenge |
| 2020 | No on-site finals (online, COVID) | — | n/a |
| 2021 | Postponed, ran online as "DCTF 21-22" | early 2022 | n/a for finals |
| 2022 | **Attack & Defense**, on-site (first A/D) | 10 Nov | Only organizer/team recaps, no per-service writeup |
| 2023 | **Attack & Defense**, on-site | 23 Nov | None found |
| 2024 | **Attack & Defense**, on-site | 28 Nov | None found |
| 2025 | **Attack & Defense**, on-site (joint with The Few Chosen) | 12–14 Nov **(conflict)** | None found |

Qualifiers have always been online Jeopardy; this index is about the finals only.

## Jeopardy finals

### 2016
- Event: [CTFtime event 393](https://ctftime.org/event/393) — 16 teams listed, scryptos first, CodiSec/CS16 second (per CTFtime/CodiLime).
- Recap: [CodiLime — team takes 2nd in the final](https://codilime.com/news/codilime-team-takes-2nd-in-global-cybersecurity-competition-final-topping-14-teams-including-current-world-1/)
  (categories: reversing, web, pwn, crypto, IoT protocols).
- Writeups: none found. p4's GitHub index files some material under "Defcamp D-CTF 2016 Finals" but dates the folder 2017-11-09 **(conflict — check which year it is)**.

### 2017
- Event: [CTFtime event 541](https://ctftime.org/event/541) — 11 teams, dcua first, p4 second (per CTFtime excerpt).
- All tasks and writeups: [CTFtime tasks page](https://ctftime.org/event/541/tasks/).
- Tasks listed: Security CCTV, Silent, spock-lizard (Ethereum series), Infinity, Fedora shop, Hack tac toe, Adversarial, Audio captcha, Caesar favourite song, State agency.
- Writeups:
  - **Infinity** (web, 400 pts) — [CTFtime writeup 8018](https://ctftime.org/writeup/8018) (winw / 0x90r00t). Goal: bypass a firewall; the target host was scannable.
  - **Adversarial** (misc/ppc) — [CTFtime writeup 8001](https://ctftime.org/writeup/8001) (Pharisaeus / p4). Summary: pick a vector that maximizes a dot product with the given one.
  - **Security CCTV** (misc, 374 pts) — [task page](https://ctftime.org/task/6284); one writeup listed, content not visible in the excerpt.
  - Team blog: [0x90r00t — DefCamp Finals 2017](https://0x90r00t.com/category/ctf/2017-ctf/defcamp-finals/) (finished 5th, per the excerpt).
  - Team repo: [p4-team/ctf](https://github.com/p4-team/ctf) — full p4 writeups.

### 2018
- Event: [CTFtime event 698](https://ctftime.org/event/698).
- Writeups:
  - **Scribbles** (web) — [Balsn](https://balsn.tw/ctf_writeup/20181108-defcampctffinal/). PHP upload chain: null-byte injection drops the extension, an operator-precedence mistake in the filename logic,
    predictable `uniqid()`/`time()` names, and PHP's base64 decoder ignoring invalid characters to smuggle a webshell past filters; a size limit on the shell has to be worked around.
  - **Authenticator** (reverse) — [jiancanxuepiao](https://jiancanxuepiao.github.io/2018/11/08/defcamp-finals/). Binary compares input to a crypto-derived value; author identifies AES-128-CTR by tracing virtual calls
    and reproduces it in Python.
  - **ico** (smart contract) — [CTFtime writeup 12129](https://ctftime.org/writeup/12129) (Maojui / DoubleSigma). Technique not visible in the excerpt. The author's own URL says "Defcamp-2019-SmartContract" while CTFtime files it under
    Finals 2018 **(conflict — check the year)**.

### 2019
- Event: [CTFtime event 925](https://ctftime.org/event/925). p4 placed 2nd (excerpt says "of 16" — **unverified**).
- Writeups (all p4, also in [p4-team/ctf](https://github.com/p4-team/ctf) under `2019-11-07-defcamp-finals`):
  - **Crypto** — [writeup 17113](https://ctftime.org/writeup/17113). Z3-based. Rotating short ints makes the XOR keystream zero; Z3 handles the add/sub and and/xor operations; the rest of the flag is brute-forced against a SHA-1.
  - **Lucky** — [writeup 17115](https://ctftime.org/writeup/17115). Side channel: `404.php` answered with HTTP/1.0 vs 1.1 depending on flag bits; bits recovered over hours from a server that kept crashing.
  - **Treasure map** (forensics/OSINT, 136 pts) — [writeup 17116](https://ctftime.org/writeup/17116). Place names at the start and end of the PDF content spell a DCTF prefix/suffix; map locations to letters/digits.
  - **Simple notes** (web, 50 pts) — [writeup 17114](https://ctftime.org/writeup/17114), [task 9705](https://ctftime.org/task/9705). Command injection via a base64 `cmd` parameter under a strict character whitelist; shell wildcards reach the flag file.

### 2020 and 2021
No on-site finals. 2020 had a COVID break, 2021 was postponed and run fully online as "DCTF 21-22"
(per DefCamp's [infographic](https://def.camp/wp-content/uploads/2022/02/DCTF-21-22-EN-infographic-compressed.pdf)).
Not covered here.

## Attack & Defense finals

No per-service exploit/patch writeups were found for any A/D final. What exists is organizer and team commentary.

### How the DefCamp A/D finals work (from the 2022 recaps)
- Each team gets VMs running intentionally vulnerable services. In 2022: two VMs per team, one running services in Docker containers and one running them directly on the host under dedicated users; services contained vulnerabilities, misconfigurations and backdoors.
- Flags are refreshed by an organizer bot about every two minutes; stolen flags score for the attacker and cost the victim.
- Patching is encouraged but risky: breaking a feature costs **SLA** (uptime/functionality) points, and the checker's logic was not documented.
- Organizer stats for 2022: 12 teams, 30 VMs, one hour of preparation, 39,271 stolen flags (~80/min), average service uptime 58%.

Sources: [DefCamp 2022 wrap-up, part one](https://def.camp/the-defcamp-2022-wrap-up-part-one-how-we-spiced-things-up-for-d-ctf-2022/) and
[Wavestone / YoloSw4g feedback](https://www.riskinsight-wavestone.com/en/2022/11/defcamp-finals-2022-feedback-on-our-first-attack-defense-ctf/).

Lessons the Wavestone team reported (useful prep notes):
- Spend the first hour finding quick wins and scripting exploits, but expect each flag to have **more theft paths than the one you found** — many teams patched a bug yet kept losing flags through others.
- Hotfixes made in a hurry were often ineffective or unclear.
- An exploit can silently die when a rival makes flag files unreadable to the vulnerable service, even if you still have code execution.

### 2022
- [CTFtime event 1824](https://ctftime.org/event/1824), [qualifier 1755](https://ctftime.org/event/1755). Finals hosted on CyberEDU. 480 qualifier teams. Top finalists per DefCamp: The Few Chosen, Wreck the Line, Lucky Lucian.
- Recaps: the two links above, plus [wrap-up part two](https://def.camp/the-defcamp-2022-wrap-up-part-two-your-defcamp-12-highlights-thank-you/).

### 2023
- [CTFtime event 2182](https://ctftime.org/event/2182), qualifier [2106](https://ctftime.org/event/2106). 16 teams on the scoreboard; PTB_WTL_0T first (3142 pts), then Lucky Lucian and The Few Chosen (per CTFtime excerpt).
- A prize for **best writeup** (top 30 teams only) was offered, so writeups should exist — none were located.

### 2024
- [Qualifier on CTFtime](https://ctftime.org/event/2480). Final on 28 Nov: Hackemus Papam (Vatican) 1st, The Few Chosen 2nd, Wreck the Line 3rd
  ([DefCamp 2024 highlights](https://def.camp/defcamp-2024-highlights-over-2000-infosec-experts-attended-first-missing-persons-search-party-and-we-got-hacked/)).
- Finalist count: 15 announced vs 16 in the recap **(conflict; recap taken as correct)**.
- Qualifier-only writeup: [kimg00n](https://blog.pwnable.net/defcampctfqual_2024/) (not a finals writeup).

### 2025
- [Qualifier on CTFtime](https://ctftime.org/event/2866); [DefCamp 2025 wrap-up](https://def.camp/?p=16196). 19 teams from 12 countries; run jointly with The Few Chosen ("D-CTF x TFCCTF").
- Reported podium: TRX (Italy) and OctalO (Romania) shared the top spot per a Forbes Romania report, `> r0/dev/null` third **(unverified — could be a tie or a reporting shorthand)**.
- A personal blog post, [defcamp CTF 2025 writeup](https://blog.lkan.onl/posts/defcamp_2025/), has crypto solutions but it is **unclear whether it covers the quals or the final**.

## Gaps and next steps

1. **Read the originals.** Allow `ctftime.org`, `def.camp` and the blog hosts above under the cloud environment's *Allowed domains*,
   then these summaries can be replaced with full, verified explanations of each challenge.
2. **A/D years (2022–2025):** look for finalist team blogs/repos (TRX, OctalO, Hackemus Papam, PTB_WTL_0T, Lucky Lucian, The Few Chosen)
   and CyberEDU service descriptions; none were found through search alone.
3. **2016:** locate a writeup for any task; none surfaced.
4. Confirm every **(conflict)** and **(unverified)** item.
5. Optionally GitHub-search for repos named `dctf`/`defcamp` + year. I did not do this because this session is scoped to this one repository.

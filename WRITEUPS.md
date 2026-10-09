# DefCamp CTF (D-CTF) Finals — writeup guide

Explained writeups for the **D-CTF finals** (DefCamp Capture the Flag, Bucharest): Jeopardy finals 2015–2019 and Attack & Defense finals 2022 onward.

> **Assumption:** "dctf finals" = DefCamp's D-CTF finals. If you meant a different CTF, this needs redoing.

## Contents

| Year | Format | Files | Depth |
|------|--------|-------|-------|
| 2015 | Jeopardy | [writeups/2015.md](writeups/2015.md) | 6 tasks explained from full writeups |
| 2016 | Jeopardy | [writeups/2016.md](writeups/2016.md) | 2 tasks (1 explained, 1 image-only) |
| 2017 | Jeopardy | [writeups/2017.md](writeups/2017.md) | 6 tasks explained; 4 more only linked |
| 2018 | Jeopardy | [writeups/2018.md](writeups/2018.md) | 1 task explained; 2 only summarised/linked |
| 2019 | Jeopardy | [writeups/2019.md](writeups/2019.md) | 4 tasks explained |
| 2020–2021 | no on-site finals (COVID, then online "DCTF 21-22") | — | n/a |
| 2022–2025 | **Attack & Defense** | [writeups/attack-defense.md](writeups/attack-defense.md) | Format, lessons and results only; no service writeups exist/found |

## How reliable is this? (read before relying on it)
- **Read in full:** every p4 team writeup for 2015, 2016, 2017, 2018 and 2019 (from [p4-team/ctf](https://github.com/p4-team/ctf)). The explanations in `writeups/2015–2019` for those tasks are my paraphrase of what the writeups say.
  One claim was re-checked by hand (2015 reverse 200: 961749023 is prime with digit sum 41) and two small facts (key length 21, `0x20 ^ 0x27 = 0x07`). **None of the exploits were re-run.**
- **Search excerpts only, pages not opened:** Balsn's 2018 *Scribbles*, the 2017 Infinity / Security CCTV links, the 2016 results, and all of the Attack & Defense material. They are labelled in each file.
- **Why:** the cloud environment's network policy blocked `ctftime.org`, `def.camp` and most blogs; only GitHub's raw/API hosts were reachable.
- p4 only documented the tasks *they solved*, so these are not the complete task lists for each year (for example CTFtime lists 10 tasks for 2017, p4 covers 6).
- Writeups are **explained and linked, not copied**; the p4 repository has no license file, so credit stays with the original authors. Challenge flags are deliberately left out.

## Gaps
1. Other teams' writeups (CTFtime tasks and writeup pages) for every year, especially 2017's Infinity, Silent and spock-lizard, 2018's remaining tasks and the 2020/2021 online events' scope.
2. Any A/D service writeups (2022–2025).
3. The 2018 *ico* smart-contract task, and which year it belongs to.
4. Confirm the "unverified" and "conflict" items in `writeups/attack-defense.md`.

To close gaps 1–2, allow `ctftime.org`, `def.camp` and the team blogs in the cloud environment's network settings and start a new session.

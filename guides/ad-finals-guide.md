# How a D-CTF Attack & Defense final works

Since 2022 the DefCamp **qualifier** stays online Jeopardy, but the **on-site final in Bucharest is Attack & Defense (A/D)**. This guide explains the A/D format in depth — the moving parts, the scoring, and what a team actually does minute to minute — so the per-year writeups and the preparation roadmap make sense.

> Sources: DefCamp's [2022 wrap-up](https://def.camp/the-defcamp-2022-wrap-up-part-one-how-we-spiced-things-up-for-d-ctf-2022/), [Wavestone/YoloSw4g feedback](https://www.riskinsight-wavestone.com/en/2022/11/defcamp-finals-2022-feedback-on-our-first-attack-defense-ctf/), and general A/D primers ([FAUST beginners' guide](https://2018.faustctf.net/information/attackdefense-for-beginners/), [MapleBacon primer](https://maplebacon.org/2025/09/maple-attack-defense-primer/), [CTF Wiki](https://ctf-wiki.mahaloz.re/introduction/experience/)). D-CTF-specific numbers are **as reported**, not independently verified.

## The core idea

Every team runs the **same set of intentionally vulnerable services**. You have two jobs at once:

- **Attack** — find a vulnerability in a service, write an exploit, and run it against *every other team* to steal their flags.
- **Defend** — patch the same vulnerability on *your* copy without breaking the service, and watch your traffic to catch attackers.

The same bug is usually both your way in and the hole you must close. The tension is that a patch which breaks functionality costs you points too.

## The moving parts

| Component | Who runs it | What it does |
|-----------|-------------|--------------|
| **Vulnbox** | You (hosted by organizers) | Your server(s) running the vulnerable services. In D-CTF 2022: **two VMs per team** — one running services in Docker containers, one running them directly on the host under dedicated users. |
| **Game network / VPN** | Organizers | Connects all teams so you can reach each other's vulnboxes. Your VPN access also exposes the scoreboard and flag submitter. |
| **Gameserver / checker** | Organizers | Every *tick*, plants fresh flags into each service (using the service's normal features), later retrieves them, and runs **SLA checks** that the service still works. |
| **Flags** | Organizers | Short tokens with a fixed format (e.g. `DCTF{...}`). In D-CTF 2022 flags were rotated about **every 2 minutes**. Only the current tick's flag scores. |
| **Exploit farm** | You | Runs your exploits against all targets on a schedule, collects the stolen flags, and submits them (see [tooling.md](tooling.md)). |
| **Traffic capture** | You | `tcpdump`/Suricata on the vulnbox feeding an analyzer (Tulip/Flower) so you can see how you are being attacked and steal the attacks. |

## Scoring: three axes

Most A/D scoreboards (and D-CTF's) combine three components:

1. **Attack / offense points** — for each valid enemy flag you submit before it expires. Early exploits are worth more; value drops as more teams exploit the same service.
2. **Defense points** — penalties avoided by *not* having your flags stolen. You lose standing when others steal from you.
3. **SLA / availability points** — the checker marks each service each tick as `OK`, `DOWN`, `FAULTY`, `FLAG_NOT_FOUND`, or `RECOVERING`. A patch that breaks a feature the checker exercises drops you to `FAULTY`/`DOWN` and bleeds SLA points. In D-CTF 2022 the average service uptime across the final was only **58%** — proof that keeping services alive is genuinely hard.

The checker's exact logic is **not published** (D-CTF teams specifically noted this), so you learn what it tests by watching your own status change as you patch.

## Timeline of a final

- **Setup / hardening hour.** D-CTF 2022 gave roughly **one hour** of preparation before attacks were allowed (Wavestone describe a 10:00–11:00 hardening slot, then play until 19:00). Use it to get the vulnbox password, bring services up, snapshot/back up everything, start packet capture, change default creds, and close obvious backdoors.
- **Opening rush.** The easiest bugs get exploited the instant the network opens — points are highest while a service is still unexploited, so first-blood matters.
- **Steady state.** Each tick: run the farm, submit flags, read new captures, push patches, confirm SLA stays green. Repeat for hours.
- **Late game.** Harder services (crypto, smart contracts) and defensive attrition decide the podium.

## What makes D-CTF's A/D distinctive

- **Two VMs with two deployment styles** (Docker vs. host users) means two patching workflows in parallel.
- **Backdoors and misconfigurations** are planted on top of code vulnerabilities — expect more than one theft path per flag. Wavestone's #1 lesson: *many teams patched the bug they knew and still lost flags through another path.*
- **~2-minute flag rotation** rewards automation; manual stealing can't keep up.
- **Undocumented SLA checker** punishes over-aggressive patching.
- **Scale (2022):** 12 teams, 30 VMs, **39,271 stolen flags** over the final (~80/minute).

## Lessons reported by finalists (2022)

- Spend the first hour on **quick wins** and **script your exploits** immediately — a one-off manual hack is worthless against 2-minute rotation.
- Expect **each flag to have more theft paths than the one you found**; patch defensively, not just the hole you walked through.
- Hurried hotfixes are often **ineffective or unclear**; there is rarely time for the "proper" fix a pentest client would get.
- Exploits **stop working silently**: one team kept code execution but lost flag reads because a rival made the flag files unreadable to the vulnerable service. Monitor your own exploits' success, not just write them once.

## Results by year (A/D era)

| Year | Event | Reported result |
|------|-------|-----------------|
| 2022 | [CTFtime 1824](https://ctftime.org/event/1824), 10 Nov, on CyberEDU; first A/D final | Finalists named by DefCamp incl. The Few Chosen, Wreck the Line, Lucky Lucian (that list is the **qualifier** leaderboard; final placings not published) |
| 2023 | [CTFtime 2182](https://ctftime.org/event/2182), 23 Nov, 16 teams | **PTB_WTL_0T** 1st (3142), **Lucky Lucian** 2nd (~2369); rest of order not confirmed |
| 2024 | 28–29 Nov, 16 finalists ([recap](https://def.camp/defcamp-2024-highlights-over-2000-infosec-experts-attended-first-missing-persons-search-party-and-we-got-hacked/)) | **Hackemus Papam** (Vatican) 1st; **The Few Chosen** 2nd + Best Romanian Team; **Wreck the Line** 3rd |
| 2025 | Joint **D-CTF x TFCCTF** final, [wrap-up](https://def.camp/?p=16196) | 19 teams / 12 countries; a Forbes Romania report names **TRX** (Italy) and **OctalO** (Romania) at the top and `> r0/dev/null` third — **unverified** (could be a tie); dates conflict (12–13 vs 13–14 Nov) |

From 2023 on, DefCamp offered a **best-writeup prize** (top 30 teams), so A/D service writeups probably exist somewhere — **none were found** in this research. See [teams-and-groups.md](teams-and-groups.md) for where to look.

## No per-service D-CTF A/D writeups found

Despite searching, **no public exploit/patch writeup for any D-CTF A/D final service exists that this research could reach.** Everything A/D-specific here is organizer/team commentary. The technical depth in this repo is therefore:

- **Deep** for the 2015–2019 **Jeopardy** finals (real task writeups — see the per-year files and [techniques-index.md](techniques-index.md)).
- **Format + strategy only** for 2022+ **A/D**, supplemented by general A/D practice ([preparation-roadmap.md](preparation-roadmap.md), [tooling.md](tooling.md), [real-scenarios.md](real-scenarios.md)).

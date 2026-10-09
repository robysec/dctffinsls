# D-CTF Attack & Defense finals (2022 onward)

From 2022 the qualifier stayed online Jeopardy, but the on-site final became **Attack & Defense (A/D)**.
**No per-service exploit/patch writeups were found for any A/D final.** Everything below comes from organizer and team commentary, **seen only as search-result excerpts** (the pages were not opened),
so treat the numbers as reported, not verified.

## How a DefCamp A/D final works (2022 recaps)
- Each team gets VMs running intentionally vulnerable services. In 2022: two VMs per team, one running services in Docker containers and one running them directly on the host under dedicated users;
  the services were modified to contain vulnerabilities, misconfigurations and backdoors.
- An organizer bot refreshes the flags about every two minutes. Stealing another team's flag scores for you and costs the victim.
- Teams are encouraged to **patch**, but a patch that breaks a feature costs **SLA** points (service uptime/functionality checks), and the checker's logic was not documented.
- Organizer stats for 2022: 12 teams, 30 VMs, one hour of preparation, 39,271 stolen flags (~80 per minute), average service uptime 58%.

Sources: [DefCamp 2022 wrap-up, part one](https://def.camp/the-defcamp-2022-wrap-up-part-one-how-we-spiced-things-up-for-d-ctf-2022/),
[Wavestone / YoloSw4g feedback](https://www.riskinsight-wavestone.com/en/2022/11/defcamp-finals-2022-feedback-on-our-first-attack-defense-ctf/).

## Lessons reported by the Wavestone team (2022)
- Spend the first hour on quick wins and script the exploits, but expect **each flag to have more theft paths than the one you found**: many teams patched the bug they knew and still lost flags through others.
- Hurried hotfixes were often ineffective or unclear, and there was rarely time to apply the "proper" fixes a client would be given after a pentest.
- An exploit can silently stop working: one team had code execution but could no longer read the flag files because a rival had made them unreadable to the vulnerable service.

## Results by year
| Year | Event | Reported result |
|------|-------|-----------------|
| 2022 | [CTFtime 1824](https://ctftime.org/event/1824), 10 Nov, hosted on CyberEDU | Top finalists named by DefCamp: The Few Chosen, Wreck the Line, Lucky Lucian |
| 2023 | [CTFtime 2182](https://ctftime.org/event/2182), 23 Nov | 16 teams on the scoreboard; PTB_WTL_0T first (3142 pts), then Lucky Lucian and The Few Chosen |
| 2024 | 28 Nov ([DefCamp highlights](https://def.camp/defcamp-2024-highlights-over-2000-infosec-experts-attended-first-missing-persons-search-party-and-we-got-hacked/)) | Hackemus Papam (Vatican) 1st, The Few Chosen 2nd, Wreck the Line 3rd. Finalists: 15 announced vs 16 in the recap (recap taken as correct) |
| 2025 | Joint "D-CTF x TFCCTF" final, [DefCamp 2025 wrap-up](https://def.camp/?p=16196) | 19 teams from 12 countries. A Forbes Romania report names TRX (Italy) and OctalO (Romania) for the top spot and `> r0/dev/null` third. **Unverified**: could be a tie or shorthand. Dates conflict: 12–13 vs 13–14 Nov |

The 2023 and later editions offered a prize for the best writeup (top 30 teams only), so writeups probably exist somewhere; none were found.

## Where to look next
- Finalist team blogs and repos (TRX, OctalO, Hackemus Papam, PTB_WTL_0T, Lucky Lucian, The Few Chosen).
- CyberEDU's descriptions of the final's services.
- CTFtime event pages' writeup tabs ([2023](https://ctftime.org/event/2182), plus the 2022, 2024 and 2025 finals).

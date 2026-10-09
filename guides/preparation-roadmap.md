# CTF finals preparation roadmap

A practical roadmap for preparing for the **D-CTF finals** — both the Jeopardy skills the 2015–2019 finals rewarded and the Attack & Defense skills the 2022+ finals demand. Built from the per-year writeups in this repo plus established A/D practice (sources at the bottom).

> Golden rule from the D-CTF finalists: **automate early, patch defensively, keep services alive.** A one-off manual hack is worthless when flags rotate every ~2 minutes.

---

## 0. Know the event first

- **Format per stage:** D-CTF qualifier = online Jeopardy; final = on-site A/D in Bucharest (since 2022). Older finals (2015–2019) were Jeopardy.
- **Read the rules the moment they drop:** flag format and regex, tick length, flag rotation interval, flag-submission rate limits, whether **AI tools are allowed during play** (some A/D events ban them), VM count and deployment style, scoring weights.
- **Confirm logistics:** team size, VPN access, on-site vs remote, power/network at the venue.

---

## 1. Twelve-week skill roadmap (Jeopardy foundations)

Even the A/D final rewards the same primitives. Rotate through categories; for each, solve past D-CTF finals tasks in this repo, then similar tasks elsewhere.

| Weeks | Focus | Drill against |
|-------|-------|---------------|
| 1–2 | **Web**: SQLi (blind/boolean/error/time), WAF bypass, XSS → admin bot, SSRF, file upload / webshells, HTTP side-channels | 2015 web200, 2017 state_agency + fedora_shop, 2018 Scribbles + TicketCore, 2019 simple-notes + lucky |
| 3–4 | **Crypto**: XOR/stream-cipher reuse, keyspace reduction, linearity, CTR malleability, solver-assisted (Z3), hash-fix brute force | 2015 crypto300/400, 2016 LeCrypto, 2017 hack_tac_toe, 2018 authenticator, 2019 crypto |
| 5–6 | **Reverse**: anti-debug/timing removal, destructors/`.fini`/`atexit`, RTTI/vtable library ID, math shortcutting | 2015 re_200/re_300, 2017 Silent, 2018 authenticator |
| 7–8 | **Pwn**: stack overflow, off-by-one, partial RET overwrite under PIE, format string, heap | 2016 SMS + Personality; then classic pwn ladders |
| 9 | **Misc/PPC/stego/forensics**: exact-matching over ML, audio/image carving, OSINT, protocol oddities | 2015 crypto100, 2017 audio_captcha + adversarial + Security CCTV, 2019 treasure-map |
| 10 | **Smart contracts**: Solidity pitfalls, reentrancy, on-chain tooling | 2017 spock-lizard, 2018 ico |
| 11–12 | **Full mock finals**: timed, multi-category, as a team | see §3 |

See [techniques-index.md](techniques-index.md) for the concrete trick behind each task.

---

## 2. Attack & Defense preparation

### 2a. Build reusable infrastructure *before* the event
These must exist and be tested in advance — not written during the hardening hour:

1. **Flag utilities:** a flag-regex matcher, a `submit_flags(list)` function speaking the organizer's submission protocol, respecting the rate limit and any required delay.
2. **Exploit runner / farm:** picks up each exploit, runs it against every target IP every tick, pipes matched flags to the submitter. Use a proven farm (DestructiveFarm / S4DFarm / ExploitFarm) rather than rolling your own under pressure — see [tooling.md](tooling.md).
3. **Exploit template:** reads a target IP, prints flags to stdout, times out cleanly, never crashes the farm.
4. **Traffic capture + analyzer:** rotating `tcpdump`/Suricata on the vulnbox, feeding Tulip or Flower so you can read attacks and replay them.
5. **Patch/deploy scripts:** one command to back up, patch, rebuild (`docker compose ... --force-recreate`) and verify a service is still `OK`.

### 2b. Assign roles (for a ~4–8 person team)
- **Infra/captain** — VPN, farm server, Tulip, capture, scoreboard watch, calls priorities.
- **Exploit writers** (1+ per service) — find bugs, write stdout-flag exploits.
- **Defenders/patchers** — harden services without breaking the checker; mirror patches across both VMs.
- **Farm/submitter operator** — keeps exploits running, watches accepted vs rejected flags, honors limits.
- **Traffic/IDS analyst** — reads captures/alerts, turns *incoming* attacks into *your* exploits ("steal the exploit").

### 2c. The first hour (hardening) — checklist
- [ ] Get vulnbox password; confirm you can reach all services and the scoreboard.
- [ ] **Snapshot / back up** every service binary and source before touching anything.
- [ ] Start packet capture immediately (you want the opening attacks on tape).
- [ ] Inventory services, ports, languages, deployment (Docker vs host user).
- [ ] Change default credentials; remove obvious planted backdoors.
- [ ] Stage your farm against your own box to confirm plumbing works.
- [ ] Prioritize: which service looks easiest to exploit (go offensive) and which is leaking worst (go defensive).

### 2d. Steady-state loop (every tick)
1. Run farm → submit flags → check accepted/rejected counts.
2. Read new captures/alerts → identify attacks hitting you → **steal and weaponize** them.
3. Patch the highest-value hole; **verify SLA stays green** before moving on.
4. Watch for your own exploits going silent (rival patched, or made flag files unreadable) and re-tool.

---

## 3. How to run a mock finals

- Spin up 2–3 local VMs (or Docker) running sample vulnerable services; one plays organizer/checker.
- Reuse a public A/D training environment (e.g. FAUST/ENOWARS past infra, iHack/Cyber Jawara service repos — see [tooling.md](tooling.md)).
- Time-box it, rotate flags on a timer, and force yourselves to use the farm + Tulip for real.
- Debrief: which bugs did you miss, which patches broke SLA, where did the pipeline stall.

---

## 4. Habits that win (and the mistakes that lose)

**Win:**
- Automate before you optimize a single manual exploit.
- Patch *defensively* — assume multiple theft paths per flag.
- Back up before patching; test before pushing; keep services `OK`.
- Log everything; align fragments (the 2019 *Lucky* task took 3 hours of fragment reassembly — logging mattered).
- Use solvers (Z3) for bit-twiddly crypto instead of hand-inverting.
- Use the known **flag format as a crib** (2017 favourite_song, 2019 treasure_map).

**Lose:**
- Hammering a shared service with heavy queries and crashing it (2015 web200).
- Writing a wrong firewall `DROP` and black-holing your own service — delete bad rules instantly.
- Patching only the hole you used and leaking through the others.
- Forgetting the transport can mangle payloads (`socat`/proxies — 2017 Silent).

---

## Sources
- DefCamp [2022 wrap-up](https://def.camp/the-defcamp-2022-wrap-up-part-one-how-we-spiced-things-up-for-d-ctf-2022/); [Wavestone feedback](https://www.riskinsight-wavestone.com/en/2022/11/defcamp-finals-2022-feedback-on-our-first-attack-defense-ctf/).
- A/D primers: [FAUST beginners](https://2018.faustctf.net/information/attackdefense-for-beginners/), [MapleBacon primer](https://maplebacon.org/2025/09/maple-attack-defense-primer/) and [patcher post](https://maplebacon.org/2024/09/faustctf-patcher/), [CTF Wiki A/D experience](https://ctf-wiki.mahaloz.re/introduction/experience/), [LosFuzzys intro (PDF)](https://losfuzzys.net/resources/trainings/2025/intro_to_ad.pdf), [rabac.tf basics](https://rabac.tf/blog/attack-defense-basics/).
- Per-year task writeups: the `writeups/` folder of this repo (primarily [p4-team/ctf](https://github.com/p4-team/ctf)).

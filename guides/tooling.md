# Attack & Defense tooling reference

The software that makes an A/D final survivable. Everything here should be set up and rehearsed **before** the event (see [preparation-roadmap.md](preparation-roadmap.md) §2a). None of these are D-CTF-specific, but they are what competitive A/D teams actually run.

## Exploit farms (attack automation)

An exploit farm runs your exploits against every target on a schedule, collects the flags they print, and submits them before they expire. This is non-negotiable when flags rotate every ~2 minutes.

| Tool | Notes |
|------|-------|
| [DestructiveFarm](https://github.com/DestructiveVoice/DestructiveFarm) | The classic. A **farm client** periodically runs exploits against other teams; a **farm server** collects flags, submits to the checksystem, tracks quotas, and shows accepted/rejected stats. Simple, battle-tested, easy to adapt the submitter to a new protocol. |
| [S4DFarm](https://github.com/C4T-BuT-S4D/S4DFarm) | A DestructiveFarm derivative with a nicer UI; widely recommended. |
| [ExploitFarm (Pwnzer0tt1)](https://github.com/pwnzer0tt1/exploitfarm) | Attacker + flag submitter with a **submit limit** (avoid flooding the platform) and a **submitter delay** (throttle when organizers ask). |
| [ExploitFarm (icc23-asia)](https://github.com/icc23-asia/ExploitFarm) | Based on S4DFarm/DestructiveFarm, tuned for ICC. |

**Exploit contract:** each exploit takes a target IP (and sometimes flag-ids from the game's attack-info endpoint), prints found flags to stdout, times out cleanly, and never hangs the farm.

## Traffic analysis (defense + "steal the exploit")

You capture your own traffic so you can see how you are being attacked — then replay/adapt those attacks against everyone else.

| Tool | Notes |
|------|-------|
| [Tulip](https://github.com/OpenAttackDefenseTools/tulip) | Web UI over captured flows; filter by service, flag the tick window, and it auto-generates Python snippets to replicate an attack. Needs a rotating sniffer (`tcpdump`/Suricata) writing `.pcap` into its traffic folder. Configure it with the flag format and tick start time. |
| [Flower](https://github.com/secgroup/flower) | TCP flow analyzer with A/D sugar. Note it provides **no security of its own** — firewall the box it runs on. |
| [Caronte](https://github.com/eciavatta/caronte) | Another flow analyzer used by A/D teams. |
| `tcpdump` | The capture workhorse: `tcpdump -i eth0 -w cap.pcap -s 0` (full snaplen). Rotate files so Tulip/Flower can ingest them. |
| `ngrep` / Wireshark | Quick ad-hoc inspection: `ngrep` matches regex/hex against payloads; in Wireshark sort by protocol and use `frame contains "..."`. |
| Suricata | IDS alerts on suspicious payloads feeding the analyzer. |

> **No-AI rule:** some A/D events forbid AI tools at runtime. Forks like [w4rya](https://github.com/JCaleb2001/w4rya) (a Tulip hard-fork) exist specifically to strip runtime AI. Check your event's rules.

## Defense: firewalling and patching

- **Firewall carefully.** `iptables -j DROP` silently black-holes matching packets. A wrong rule can take *your own* service down and cost SLA — if a rule misbehaves, delete it immediately and restore the previous config. Application-layer filtering (e.g. a WAF in front of a leaky endpoint) can buy time while you write the real patch.
- **Patch without breaking the checker.** For source: edit, recompile if needed, redeploy (`docker compose down && docker compose up -d --build --force-recreate`). For binaries: `patchelf`, `pwntools` patching, or raw byte edits. **Back up the original first** so exploit-writers can still analyze it.
- **Expect downtime.** A push may take a service down for a tick — test on a copy before deploying, and confirm the service returns to `OK` after.

## Flag submitter

The component that sends stolen flags to the organizers' checksystem. Usually built into the farm. Must:
- speak the event's submission protocol (TCP socket or HTTP API),
- respect the **rate limit** and any requested delay,
- de-duplicate and only resubmit flags still within their validity window,
- surface accepted vs rejected counts so you notice when an exploit has gone stale.

## Practice environments

- [FAUST CTF](https://faustctf.net/) and [ENOWARS](https://enowars.com/) — recurring online A/D events with reusable past infrastructure.
- [farisv/CJ2018-Final-CTF](https://github.com/farisv/CJ2018-Final-CTF) — Cyber Jawara 2018 A/D services in Docker.
- [WesleyWong420/iHack-Attack-Defense](https://github.com/WesleyWong420/iHack-Attack-Defense) — iHack 2022 A/D prep.
- [BurningNetel/attack-defense-CTF-framework](https://github.com/BurningNetel/attack-defense-CTF-framework) — a pure-Python A/D framework for building your own mock game.

## Cheatsheets
- [Attack & Defense CTF Cheatsheet (HackMD)](https://hackmd.io/@Masamune/SyiHF1qcA)
- [attacking-lab A/D wiki](https://wiki.attacking-lab.com/attack-defense/)
- [Web CTF Cheatsheet](https://github.com/duckstroms/Web-CTF-Cheatsheet)

> Links are to upstream projects; vet and pin versions before an event. Nothing here is affiliated with DefCamp.

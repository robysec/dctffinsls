# Challenges — per-category study material

CTF challenge-category references, grouped by category. Each file is a complete pack for its category: **techniques** (what the trick is + a plain scenario), a **catalog** (tools & scripts), **copy-paste scripts**, and a **DCTF roadmap** (what to study, how to practise, and how it maps to the D-CTF quals/finals).

These are category skill-builders; for the actual D-CTF finals task writeups and the A/D preparation kit, see the repo root [WRITEUPS.md](../WRITEUPS.md).

## Jeopardy skill categories

The pull-the-flag-out categories you grind in the online quals.

| Category | Folder | File | Covers |
|----------|--------|------|--------|
| **Crypto** | [`crypto/`](crypto/) | [Crypto-Complete.md](crypto/Crypto-Complete.md) | encoding/classical, XOR, RSA, symmetric modes, PRNG/stream, hashing/MAC, DH/ECC, lattices, JWT |
| **Pwn** | [`pwn/`](pwn/) | [Pwn-Complete.md](pwn/Pwn-Complete.md) | stack overflow, ROP/ret2libc, format string, protection bypass, heap (tcache/house-of), kernel, seccomp |
| **Reverse** | [`reverse/`](reverse/) | [RE-Complete.md](reverse/RE-Complete.md) | static (Ghidra/r2), dynamic (gdb), transforms, angr/Z3, packing/anti-debug, patching, VM/bytecode, per-language |
| **Forensics** | [`forensics/`](forensics/) | [Forensics-Complete.md](forensics/Forensics-Complete.md) | file triage, carving, image/audio stego, pcap, memory (Volatility), disk/DFIR, docs/archives |
| &nbsp;&nbsp;↳ **Stego** | [`forensics/stego/`](forensics/stego/) | [Stego-Complete.md](forensics/stego/Stego-Complete.md) | image LSB/bit-planes, keyed extractors, visual transforms, audio, text/whitespace/unicode, polyglots, QR |
| **Networking** | [`networking/`](networking/) | [Networking-Complete.md](networking/Networking-Complete.md) | pcap triage, protocol analysis, extraction, covert channels, wireless, USB, scanning/crafting |
| **OSINT** | [`osint/`](osint/) | [OSINT-Complete.md](osint/OSINT-Complete.md) | search/dorking, username/email pivots, domain/DNS/cert, image + geolocation, archives, code leaks |

## Competition formats

The live, interactive game modes — not "solve a puzzle" but "hold a box / run services under attack". These train the skills the **D-CTF final (Attack & Defense)** rewards.

| Format | Folder | File | Covers |
|--------|--------|------|--------|
| **Attack & Defense** | [`koth+ad/attack-defense/`](koth+ad/attack-defense/) | [Attack-Defense-Complete.md](koth+ad/attack-defense/Attack-Defense-Complete.md) | prep/roles, first-hour hardening, find-the-bug per class, exploit farms, traffic-stealing, patching under SLA, monitoring, anti-persistence |
| **King of the Hill** | [`koth+ad/koth/`](koth+ad/koth/) | [KoTH-Complete.md](koth+ad/koth/KoTH-Complete.md) | fast initial access, privesc, claiming/holding the hill, patching your own entry, persistence, evicting rivals within rules |

> The **D-CTF final is Attack & Defense.** This A/D pack is the general study material; the D-CTF-specific A/D walkthrough and preparation kit live in [`../guides/`](../guides/) ([how A/D finals work](../guides/ad-finals-guide.md), [preparation roadmap](../guides/preparation-roadmap.md), [tooling](../guides/tooling.md)). KoTH isn't a D-CTF format, but it drills the same root-fast / patch-your-entry / hold-under-contention muscles.

## How it's grouped

- **Jeopardy skill categories** — one folder each (crypto, pwn, reverse, forensics, networking, osint). **Stego** is nested under `forensics/` because it's a sub-category of forensics (the material treats it that way: same triage-first discipline, roadmap points back to forensics).
- **Competition formats** — grouped together under `koth+ad/` (attack-defense, koth), because they're live game modes rather than puzzle categories, and the two packs cross-reference each other.

## Notes on the source

- These files come from an Obsidian-style vault, so some internal links are wiki-links like `[[Crypto Catalog]]` or `[[Pwn-Complete]]` — they won't resolve as clickable links on GitHub, but the referenced content is included inline in the same file (techniques → catalog → scripts → roadmap).
- The script snippets are standard CTF tooling for **authorised** practice only (your own challenges, CTF targets and lab/scope you're allowed to test). The A/D and KoTH packs repeat this: stay in the assigned scope, never touch game infra or the scoring service, respect each event's rules.
- Flags shown in examples are placeholders (`flag{}` / `dctf{}`), not real solutions.

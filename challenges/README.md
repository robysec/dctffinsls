# Challenges — per-category study material

CTF challenge-category references, grouped by category. Each file is a complete pack for its category: **techniques** (what the trick is + a plain scenario), a **catalog** (tools & scripts), **100 copy-paste scripts**, and a **DCTF roadmap** (what to study, how to practise, and how it maps to the D-CTF quals/finals).

These are category skill-builders; for the actual D-CTF finals task writeups and the A/D preparation kit, see the repo root [WRITEUPS.md](../WRITEUPS.md).

## Categories

| Category | Folder | File | Covers |
|----------|--------|------|--------|
| **Crypto** | [`crypto/`](crypto/) | [Crypto-Complete.md](crypto/Crypto-Complete.md) | encoding/classical, XOR, RSA, symmetric modes, PRNG/stream, hashing/MAC, DH/ECC, lattices, JWT |
| **Pwn** | [`pwn/`](pwn/) | [Pwn-Complete.md](pwn/Pwn-Complete.md) | stack overflow, ROP/ret2libc, format string, protection bypass, heap (tcache/house-of), kernel, seccomp |
| **Reverse** | [`reverse/`](reverse/) | [RE-Complete.md](reverse/RE-Complete.md) | static (Ghidra/r2), dynamic (gdb), transforms, angr/Z3, packing/anti-debug, patching, VM/bytecode, per-language |
| **Forensics** | [`forensics/`](forensics/) | [Forensics-Complete.md](forensics/Forensics-Complete.md) | file triage, carving, image/audio stego, pcap, memory (Volatility), disk/DFIR, docs/archives |
| &nbsp;&nbsp;↳ **Stego** | [`forensics/stego/`](forensics/stego/) | [Stego-Complete.md](forensics/stego/Stego-Complete.md) | image LSB/bit-planes, keyed extractors, visual transforms, audio, text/whitespace/unicode, polyglots, QR |

## How they're grouped

Five top-level CTF categories, one folder each — **crypto, pwn, reverse, forensics**. **Stego** is nested under `forensics/` because it is a sub-category of forensics (the material itself treats it that way: the same triage-first discipline, and its roadmap points back to forensics). The other four stand alone.

## Notes on the source

- These files come from an Obsidian-style vault, so some internal links are wiki-links like `[[Crypto Catalog]]` or `[[Scripts Index#...]]` — they won't resolve as clickable links on GitHub, but the referenced content is included inline in the same file (techniques → catalog → scripts → roadmap).
- The script snippets are standard CTF tooling for **authorised** practice (your own challenges, CTF targets you're allowed to test). Keep them to that use.
- Flags shown in examples are placeholders (`flag{}` / `dctf{}`), not real solutions.

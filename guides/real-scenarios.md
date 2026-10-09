# Real scenarios — worked examples from D-CTF finals

Concrete, real situations that happened at D-CTF finals, framed as "you are here → what you'd do". Each is drawn from a documented task or finalist account in this repo. They're meant to rehearse decision-making, not to hand you flags (flag values and verbatim exploit payloads are omitted — follow the source links for those).

---

## Scenario 1 — Blind SQLi where nothing echoes back (2015, web 200)
**You are here.** An image uploader reads the EXIF `Software` field into a `WHERE` clause. Injection length is capped ~50 chars, `UNION` is unavailable, and the queried table is empty — nothing comes back on screen.

**The move.** You have no output channel, so manufacture a **1-bit oracle**. First attempt: a timing oracle with `BENCHMARK()` (not `SLEEP`) — but heavy queries crashed the shared server five times and got `BENCHMARK` blocked. Better: an **error oracle** — craft the query so a true/false condition makes it *valid or syntactically broken* (`rlike(if(<cond>,'',1))`), and read the yes/no from how the page renders. Loop over position × charset, rewriting the EXIF and re-uploading each time.

**Lesson.** When time-based and `UNION` are closed, any visible valid-vs-error difference is an oracle. Don't DoS a shared service.

---

## Scenario 2 — "I passed every check but the flag is rejected" (2015, reverse 300)
**You are here.** A binary validates `argv[1]`: length 37, prefix `DCTF{`, eight 4-char groups each matching a small regex. You satisfy all of it, it prints "Well done" — and the server still says no.

**The move.** The program keeps running after `main`. A **destructor** performs one more check (a per-char sum reduced mod a constant must equal a target). Don't reverse the math by hand — enumerate every candidate the regexes allow and brute-force the destructor condition.

**Lesson.** Always inspect constructors, destructors, `.init`/`.fini`, `atexit`. A "success" message that the server rejects means there's another gate.

---

## Scenario 3 — A long hash chain that *looks* unbreakable (2016, LeCrypto)
**You are here.** RC4 is implemented correctly; the key is derived through nested MD5s. Reversing MD5 is hopeless.

**The move.** Count the **actual unknown bits**, not the hashes. The key reduces to `md5(h1[:8] + fixed)` — only 8 hex chars = **32 bits** of real entropy. Brute-force all 2³², decrypt a known field (`v` is 16 printable chars), and keep only keys whose output is printable. Decrypt the message with survivors.

**Lesson.** A hash chain is only as strong as its smallest unknown. A known-plaintext property (printable range) confirms the key without knowing the plaintext.

---

## Scenario 4 — Egress is firewalled but there's an admin bot (2017, Fedora shop)
**You are here.** Stored XSS fires in an admin's browser, but the admin can't call out to your server, and your payload won't fit in one field.

**The move.** **Exfiltrate through the app itself.** Split the script across fields (a tiny loader `eval`s text from another cell). The cart is server-side and keyed by `PHPSESSID`; make the admin's browser POST "add to cart" using *your* session id, stuffing stolen page text into the `quantity` field (never validated as numeric). Then open *your* cart and read it.

**Lesson.** When egress is blocked, find a feature that stores attacker-readable data. Re-read your own state channel.

---

## Scenario 5 — The exploit works locally but not remotely (2017, Silent)
**You are here.** A byte-at-a-time leak works on your machine; against the real service it fails.

**The move.** The organizers forwarded the service through `socat`, which **rewrote the payload** in transit. Diff what you send vs what arrives (capture both ends), and encode around the mangling.

**Lesson.** Suspect the transport — proxies, `socat`, encoding/line-ending rewrites — before doubting your logic. (This is doubly true in A/D, where you're routed through game infra.)

---

## Scenario 6 — The "one open port" that nmap won't show (2017, Infinity)
**You are here.** A box you can scan; the hint insists there's a way "past the firewall" and really only one meaningful port — but you've already found three.

**The move.** A default nmap run skips **port 0**. The real service listened there. Scan the edges of the range explicitly and use tooling that can actually address port 0.

**Lesson.** Default scans have blind spots. "There's one more port" + obvious ones exhausted → check the edges (port 0, full UDP, high ranges).

---

## Scenario 7 — Forging is impossible, so go *through* the signer (2018, Scribbles)
**You are here.** Uploads are HMAC-signed with the secret flag as the key, and the signing request is made server-side to localhost. You can't forge the signature or read it.

**The move.** Attack the *construction*, not the crypto. Your `data` is concatenated unescaped into the signed `data=...&name=...` string, so inject `&name=...` + NUL to control the output filename (curl truncates at NUL). Force a `.php` name by making `base64_decode(contents)` empty (PHP returns empty for input with no valid base64 chars), which skips the extension append. Write a ≤37-byte **non-alphanumeric webshell** (`<?=`, backticks, XOR/NOT to synthesise `_GET`).

**Lesson.** If you can't forge a signature, make the legitimate signer sign *your* smuggled data. NUL truncation differs between layers (curl vs PHP) — that gap is the bug.

---

## Scenario 8 — A WAF blocking every useful keyword (2018, TicketCore)
**You are here.** Boolean-blind SQLi, but `and`/`or`/`if`, the literal `DCTF`, and `0x` hex are all filtered; almost every string function is blocked.

**The move.** Swap `and`/`or` for `&&`/`||`. Represent your comparison string as a **binary literal** `0b...` instead of `'DCTF'`/hex. If you must use a string function, note the one survivor (`to_base64`) and **double-encode** to dodge the filtered `if` inside the base64. Extract char-by-char with range comparisons.

**Lesson.** Blacklists always miss alternatives: `&&`/`||`, `0b` literals, the one un-filtered function. Watch case-insensitive comparisons destroying information.

---

## Scenario 9 — A keystream reused across messages (2017, Hack tac toe / 2015 crypto 400)
**You are here.** A cookie/ciphertext where changing one plaintext byte changes ~one ciphertext byte. The keystream doesn't change between requests.

**The move.** Chosen-plaintext: set a long known value (100 `a`s), diff old vs new ciphertext to locate and recover keystream bytes, then XOR it back. If the recovered keystream **repeats** (period 16), you hold the whole key and can decrypt everything. For the keyless/linear variant, differences between ciphertexts that mirror plaintext differences reveal a fixed XOR mask — one known pair unlocks all.

**Lesson.** Always inspect the recovered keystream for a short period. Keystream reuse = chosen-plaintext decryption.

---

## Scenario 10 — A side channel you'd never guess (2019, Lucky)
**You are here.** A locked-down router; only `index.php`/`404.php` exist. `/404.php` over raw HTTP/1.0 sometimes replies `HTTP/1.0`, sometimes `HTTP/1.1`.

**The move.** The server **broadcasts the flag as a bitstream via the protocol version** (`1.0`=0, `1.1`=1), flipping ~every 3s while you poll ~1/s. It crashes and replays random fragments. Log everything, normalize run-lengths to multiples of 3, try all 8 bit offsets, score bytes as flag-like, and jigsaw overlapping fragments together (~3 hours of collection).

**Lesson.** Side channels can be as weird as a protocol-version label. Log exhaustively and reassemble by overlap.

---

## A/D meta-scenario — "I patched the bug and still lost flags" (2022 final)
**You are here.** You found a vuln, patched your copy, confirmed SLA green — and the scoreboard still shows your flags being stolen.

**The move.** Assume **multiple theft paths per flag.** Read your captures to see *which* request is actually exfiltrating, not the one you imagined. Also check your own exploit hasn't gone silent because a rival made the flag files unreadable to the vulnerable service (a real 2022 occurrence). Patch defensively across both VMs (Docker and host).

**Lesson.** In A/D the bug you found is rarely the only door. Monitor outcomes (captures + farm accept rate), not intentions.

> Full technique details and sources per scenario are in the matching `writeups/<year>.md` file and [techniques-index.md](techniques-index.md).

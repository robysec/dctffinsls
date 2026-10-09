# Techniques index — every trick, by category

A cross-year index of the actual techniques behind the D-CTF finals tasks in this repo, grouped by category so you can study a skill rather than a year. Each entry links to the task where it appears. Use this with the [preparation-roadmap](preparation-roadmap.md).

## Web
| Technique | Where | One-liner |
|-----------|-------|-----------|
| SQLi via EXIF metadata | [2015 web200](../writeups/2015.md#web-200--sql-injection-through-exif) | File metadata flows into a query. |
| Error-based boolean oracle | [2015 web200](../writeups/2015.md#web-200--sql-injection-through-exif) | Valid-vs-syntax-error = 1 bit/request when nothing echoes. |
| SQLi via `Host` header | [2017 state_agency](../writeups/2017.md#state-agency-web) | The injectable param isn't in the visible URL. |
| `PROCEDURE ANALYSE()` for schema | [2017 state_agency](../writeups/2017.md#state-agency-web) | Leaks table/column names when `information_schema` is blocked. |
| Output-filter bypass with `to_base64` | [2017 state_agency](../writeups/2017.md#state-agency-web), [2018 TicketCore](../writeups/2018.md#ticketcore-web-sql-injection--other-team) | Re-encode the flag so the response filter can't match it. |
| Boolean-blind with `&&`/`||`, `0b` literals | [2018 TicketCore](../writeups/2018.md#ticketcore-web-sql-injection--other-team) | Beat keyword/`0x`/string-function blacklists. |
| Stored XSS → admin bot | [2017 fedora_shop](../writeups/2017.md#fedora-shop-web) | Unsanitized field runs JS in an admin browser. |
| Payload split across fields + `eval` chaining | [2017 fedora_shop](../writeups/2017.md#fedora-shop-web) | Beat per-field length limits. |
| Exfiltrate through the app's own state (cart) | [2017 fedora_shop](../writeups/2017.md#fedora-shop-web) | When egress is firewalled, store-and-read-back. |
| Parameter injection into a signed string + NUL truncation | [2018 Scribbles](../writeups/2018.md#scribbles-web--other-teams) | curl vs PHP disagree on where a NUL ends the data. |
| `base64_decode`→empty filter bypass | [2018 Scribbles](../writeups/2018.md#scribbles-web--other-teams) | Non-base64 input decodes to "" and skips the `.ext` append. |
| Non-alphanumeric PHP webshell (`<?=`, backticks, XOR/NOT) | [2018 Scribbles](../writeups/2018.md#scribbles-web--other-teams) | Build `_GET`/`cat` without letters, under a byte cap. |
| Command injection with char-whitelist + globs | [2019 simple-notes](../writeups/2019.md#simple-notes-web-50-pts-16-solves) | Name files/binaries you can't spell; `awk 4` instead of `cat`. |
| HTTP protocol-version side channel | [2019 lucky](../writeups/2019.md#lucky-web-304-pts-5-solves) | `HTTP/1.0` vs `1.1` as a 0/1 bitstream. |
| Port-0 / scan blind spots | [2017 Infinity](../writeups/2017.md#infinity-webnetwork-400-pts--other-team) | Default nmap skips port 0. |

## Crypto
| Technique | Where | One-liner |
|-----------|-------|-----------|
| Keyspace reduction + known-plaintext filter | [2016 LeCrypto](../writeups/2016.md#lecrypto-crypto-250) | A hash chain is only as strong as its smallest unknown (32 bits). |
| High-bit leakage of key material | [2015 crypto300](../writeups/2015.md#crypto-300--leaking-key-bits-through-the-high-bit) | Bit 7 can only come from the key → read the key off. |
| Keyless/linear cipher = fixed XOR mask | [2015 crypto400](../writeups/2015.md#crypto-400--a-keyless-cipher-is-linear) | One known pair decrypts all. |
| Stream-cipher keystream recovery + short period | [2017 hack_tac_toe](../writeups/2017.md#hack-tac-toe-webcrypto) | Chosen-plaintext diff; period 16 = whole key. |
| AES-CTR malleability (encrypt = decrypt) | [2018 authenticator](../writeups/2018.md#authenticator-pwn-category-really-reverse--crypto--2-solves) | Symmetric XOR with keystream; pass the check, decrypt the flag. |
| Brute-force one unknown byte online | [2018 authenticator](../writeups/2018.md#authenticator-pwn-category-really-reverse--crypto--2-solves) | ASLR flips 1 key byte → 256 reconnects. |
| Z3 / SMT for bit-op ciphers | [2019 crypto](../writeups/2019.md#crypto-136-pts-12-solves) | Model add/xor/and/rotate as constraints instead of inverting. |
| Embedded checksum to fix ambiguity | [2019 crypto](../writeups/2019.md#crypto-136-pts-12-solves) | SHA-1 prefix brute-forces the last unknown chars. |
| Hidden Unicode / zero-width separators | [2015 crypto100](../writeups/2015.md#crypto-100--morse-cest) | Look for invisible chars before brute-forcing. |

## Reverse
| Technique | Where | One-liner |
|-----------|-------|-----------|
| Strip timing/anti-shortcut obfuscation | [2015 re200](../writeups/2015.md#reverse-200--time-is-not-your-friend) | Delete sleeps, do the math directly. |
| Recognize the underlying math and skip work | [2015 re200](../writeups/2015.md#reverse-200--time-is-not-your-friend) | Prime-count + digit-sum → jump ahead. |
| Hidden check in destructor/`.fini` | [2015 re300](../writeups/2015.md#reverse-300--try-harder) | The real gate runs after `main`. |
| RTTI/vtable strings identify the crypto lib | [2018 authenticator](../writeups/2018.md#authenticator-pwn-category-really-reverse--crypto--2-solves) | `CryptoPP::...` names the algorithm in minutes. |
| Byte-at-a-time leak oracle | [2017 Silent](../writeups/2017.md#silent-reverse--other-team) | Simple in concept; mind the transport. |

## Pwn
| Technique | Where | One-liner |
|-----------|-------|-----------|
| Off-by-one → saved-RET reach | [2016 SMS](../writeups/2016.md#sms-pwn-200) | One extra byte past the buffer. |
| Partial RET overwrite under PIE | [2016 SMS](../writeups/2016.md#sms-pwn-200) | Overwrite low 2 bytes; brute ~4 bits. |

## Misc / PPC / stego / forensics
| Technique | Where | One-liner |
|-----------|-------|-----------|
| Exact byte-matching instead of ML | [2017 audio_captcha](../writeups/2017.md#audio-captcha-miscppc) | Captcha reuses identical clips; skip ambiguous ones. |
| Adversarial sign-matching dot product | [2017 adversarial](../writeups/2017.md#adversarial-miscppc) | `+1`/`-1` matching the row's signs maxes the sigmoid. |
| Flag-format as a crib | [2017 favourite_song](../writeups/2017.md#caesars-favourite-song-misccryptostego), [2019 treasure_map](../writeups/2019.md#treasure-map-forensicsosint-136-pts-12-solves) | Known prefix/suffix reveals the mapping. |
| Real-world structure (musical scale) as the key | [2017 favourite_song](../writeups/2017.md#caesars-favourite-song-misccryptostego) | Notes → D-major scale → bytes. |
| Perspective transform to rebuild a distorted QR | [2017 Security CCTV](../writeups/2017.md#security-cctv-misc-374-pts--other-teams) | 4 corners → clean square → decode. |
| "Data is visual" — map shapes spell letters | [2019 treasure_map](../writeups/2019.md#treasure-map-forensicsosint-136-pts-12-solves) | Place-name shapes form the flag. |
| Fragment reassembly by overlap | [2019 lucky](../writeups/2019.md#lucky-web-304-pts-5-solves) | Jigsaw overlapping samples from a crashy stream. |

## Smart contracts (Ethereum)
| Technique | Where | One-liner |
|-----------|-------|-----------|
| On-chain challenge series (alpha/beta/omega) | [2017 spock-lizard](../writeups/2017.md#spock-lizard-crypto--ethereum-smart-contract--other-teams) | Only alpha was solved by one team — hard for most. |
| Solidity/ICO contract exploitation | [2018 ico](../writeups/2018.md#ico-smart-contract--not-explained) | Technique not documented in reachable sources. |

## A/D-specific skills (2022+)
These aren't tied to a single task writeup; see [ad-finals-guide.md](ad-finals-guide.md), [tooling.md](tooling.md) and [real-scenarios.md](real-scenarios.md):
exploit-farm automation · flag submitter rate-limiting · traffic capture + flow analysis (Tulip/Flower) · "steal the exploit" from incoming traffic · non-SLA-breaking patching · defensive firewalling · snapshot/rollback discipline · assuming multiple theft paths per flag.

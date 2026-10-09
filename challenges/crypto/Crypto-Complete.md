# Crypto Techniques

Each: plain explanation + a *Scenario* anyone can follow. Pair with [[Crypto Catalog]] (tools) and [[Crypto]] (commands).

> Mental model: CTF crypto is rarely "break AES." It's spotting where someone used real crypto *wrong* — a reused number, a tiny key, a leak — and exploiting that mistake.

## A. Recognize & decode (do first)

1. **encoding vs encryption** — is it reversible with no key? *Scenario:* looks scrambled but it's just base64; decode, no attack needed.
2. **base64 detection** — `A-Za-z0-9+/=`, length %4. *Scenario:* a block ending in `=` decodes straight to the flag.
3. **base32** — A-Z2-7. *Scenario:* uppercase+digits blob decodes with `base32 -d`.
4. **hex** — pairs of 0-9a-f. *Scenario:* `666c6167` is just "flag" in hex.
5. **base58/62/85** — crypto/URL variants. *Scenario:* a Bitcoin-style string is base58; decode to bytes.
6. **ROT13/Caesar** — shifted letters. *Scenario:* "synt" → rotate 13 → "flag".
7. **Atbash** — alphabet reversed. *Scenario:* "zyxw" patterns hint at Atbash.
8. **Morse** — dots/dashes. *Scenario:* a string of `.-` decodes to text.
9. **frequency analysis** — letter counts reveal substitution. *Scenario:* most common symbol = 'e'; unravel the map.
10. **CyberChef Magic** — auto-detect layered encodings. *Scenario:* base64→gzip→hex nesting peeled automatically.
11. **multi-layer peeling** — decode repeatedly. *Scenario:* each decode reveals another encoded blob until plaintext.

## B. Classical ciphers

12. **Caesar brute (25 shifts)** — try all. *Scenario:* short ciphertext; one shift reads as English.
13. **Vigenère with known key** — subtract the keyword. *Scenario:* key is hinted in the prompt; decrypt directly.
14. **Vigenère keylen (Kasiski/IoC)** — find period, then solve columns. *Scenario:* no key given; repeating patterns reveal keylen 5.
15. **substitution solve (quipqiup)** — map symbols to letters. *Scenario:* a 1:1 symbol cipher cracked by frequency.
16. **transposition (rail fence/columnar)** — letters reordered, not changed. *Scenario:* all the right letters, wrong order; undo the route.
17. **Playfair/Hill/ADFGVX** — classic grid/matrix ciphers. *Scenario:* digraph patterns point to Playfair; dcode solves it.
18. **Bacon cipher** — binary hidden in A/B or case. *Scenario:* MiXeD cAsE encodes bits → letters.
19. **book/running-key cipher** — key is a text. *Scenario:* a referenced poem is the key stream.

## C. XOR

20. **single-byte XOR brute** — try 256 keys, score English. *Scenario:* ciphertext XOR'd with one byte; best-scoring key reveals the flag.
21. **known-plaintext XOR** — recover key from known part. *Scenario:* you know it starts `flag{`; XOR to get the key.
22. **repeating-key XOR keylen** — Hamming distance per candidate length. *Scenario:* find keylen 7, then solve each of 7 columns.
23. **repeating-key XOR solve** — per-column single-byte. *Scenario:* split into 7 streams; brute each → full key.
24. **crib-dragging** — slide a guessed word across two XOR'd-with-same-key texts. *Scenario:* two messages, one keystream; drag "the" to peel both.
25. **XOR with structure** — exploit null bytes/known headers. *Scenario:* a PNG header XOR'd gives away the key bytes.

## D. RSA basics

26. **decrypt with p,q** — compute d, `m=c^d mod n`. *Scenario:* challenge leaks p and q; textbook decrypt.
27. **factordb lookup** — n already factored online. *Scenario:* paste n, get p and q for free.
28. **small-n factoring (yafu/sympy)** — n is tiny. *Scenario:* 64-bit n factors in seconds locally.
29. **Fermat factorization** — p,q close together. *Scenario:* p and q differ by a little; Fermat splits n fast.
30. **common modulus** — same n, two e, two c. *Scenario:* message sent twice with different e; combine with ext-gcd, no factoring.
31. **common prime (shared factor)** — gcd of two moduli. *Scenario:* two keys reuse a prime; `gcd(n1,n2)` cracks both.
32. **small e, no padding (cube root)** — e=3, m small. *Scenario:* c is a perfect cube; just take the cube root.
33. **Håstad broadcast** — same m, e recipients. *Scenario:* same message to 3 people with e=3; CRT + cube root.
34. **Wiener's attack** — d too small. *Scenario:* huge e, tiny d; continued fractions recover d.
35. **Boneh-Durfee** — d < n^0.292. *Scenario:* d slightly bigger than Wiener allows; lattice attack still wins.
36. **partial key exposure** — some bits of p/d known. *Scenario:* half of p leaked; Coppersmith recovers the rest.
37. **Coppersmith stereotyped message** — most of m known. *Scenario:* message is "flag is XXXX" with few unknowns; small-roots finds them.
38. **Franklin-Reiter related messages** — m and f(m) both sent. *Scenario:* two linearly-related plaintexts under same key; solve polynomial GCD.
39. **LSB/parity oracle** — server tells if m is even/odd. *Scenario:* repeatedly halving via the oracle binary-searches the plaintext.
40. **signature forgery (no padding)** — multiplicative RSA. *Scenario:* get sigs of a,b, multiply for sig of a*b.
41. **Bleichenbacher PKCS#1 (ROBOT)** — padding-validity oracle on RSA. *Scenario:* server leaks PKCS#1 validity; decrypt/sign with many queries.

## E. Symmetric block ciphers

42. **ECB repeated-block detection** — equal plaintext blocks look equal. *Scenario:* cipher shows two identical 16-byte chunks → ECB confirmed.
43. **ECB cut-and-paste** — rearrange blocks. *Scenario:* reorder ciphertext blocks to make `role=admin`.
44. **ECB byte-at-a-time** — recover a secret appended to your input. *Scenario:* align so one unknown byte sits at a boundary; brute it, repeat.
45. **CBC bit-flipping** — flip a byte in block N to change plaintext N+1. *Scenario:* flip a bit so `;admin=0;` becomes `;admin=1;`.
46. **CBC IV manipulation** — IV controls first block. *Scenario:* changing IV flips the first plaintext block to what you want.
47. **CBC padding oracle** — validity leak decrypts without the key. *Scenario:* server says "bad padding"; byte-by-byte you decrypt the cookie.
48. **CTR/GCM nonce reuse** — same nonce twice = XOR of plaintexts. *Scenario:* two messages, same nonce; XOR cancels keystream.
49. **GCM forbidden attack** — nonce reuse recovers the auth key. *Scenario:* forge a valid tag after seeing two messages on one nonce.
50. **CFB-8 quirks** — byte-feedback edge cases. *Scenario:* a per-byte mode leaks structure you exploit.
51. **weak/known key** — key is guessable/hardcoded. *Scenario:* key is "0000..." or in the source; just decrypt.
52. **DES brute / weak keys** — 56-bit or weak-key classes. *Scenario:* a DES challenge with a tiny keyspace falls to brute force.

## F. Stream ciphers & PRNGs

53. **LFSR recovery (Berlekamp-Massey)** — rebuild the register from output. *Scenario:* given enough keystream bits, recover the LFSR and predict the rest.
54. **RC4 biases/known-plaintext** — recover keystream. *Scenario:* known header lets you peel RC4 output.
55. **Mersenne Twister state recovery** — rebuild MT19937 from 624 outputs. *Scenario:* 624 "random" numbers reveal the full state; predict future keys.
56. **MT untempering** — invert the output transform. *Scenario:* convert observed outputs back to internal state words.
57. **srand(time) prediction** — seed is the timestamp. *Scenario:* reproduce the "random" key by guessing the second it ran.
58. **LCG cracking** — linear congruential generator is invertible. *Scenario:* a few outputs reveal the LCG parameters.
59. **nonce reuse in stream ciphers** — same keystream twice. *Scenario:* two ciphertexts XOR to kill the keystream.

## G. Hashing & MACs

60. **hash identification** — length/charset → algorithm. *Scenario:* 32 hex chars = MD5; pick the right cracker.
61. **dictionary cracking (hashcat/john)** — hash of a common word. *Scenario:* the secret is MD5("password"); rockyou finds it.
62. **rule-based cracking** — mutate wordlist. *Scenario:* "Flag2024!" found by applying leet/append rules.
63. **rainbow/online lookup** — precomputed tables. *Scenario:* CrackStation instantly reverses a common hash.
64. **length-extension attack** — `H(secret||msg)` MAC forgery. *Scenario:* append `&admin=1` to a signed message without the secret using hashpump.
65. **hash collision (MD5/SHA1)** — two inputs, same hash. *Scenario:* submit a different file with the same checksum.
66. **CRC forgery** — CRC is linear; craft a target CRC. *Scenario:* tweak bytes so a file hits a required CRC32.
67. **bcrypt 72-byte truncation** — long passwords collide. *Scenario:* two different long passwords authenticate the same.

## H. Public-key beyond RSA

68. **Diffie-Hellman small/smooth prime** — weak group. *Scenario:* p-1 is smooth; Pohlig-Hellman solves the shared secret.
69. **discrete log (baby-step giant-step)** — medium DLP. *Scenario:* small group order; BSGS finds the exponent.
70. **Pohlig-Hellman** — order factors into small primes. *Scenario:* break DLP piecewise via CRT.
71. **Pollard's rho/kangaroo** — DLP in bounded range. *Scenario:* exponent known to be in a range; kangaroo finds it.
72. **ElGamal nonce reuse** — same k twice leaks the key. *Scenario:* two signatures share k; algebra recovers the private key.

## I. Elliptic curves

73. **ECDLP on weak curve** — small/smooth order. *Scenario:* curve order factors nicely; Pohlig-Hellman on the curve.
74. **Smart's attack (anomalous curve)** — order == p. *Scenario:* trace-one curve; lift to p-adics, solve in linear time.
75. **MOV attack** — low embedding degree. *Scenario:* pairing maps ECDLP to an easy finite-field DLP.
76. **invalid-curve attack** — server accepts off-curve points. *Scenario:* feed points from weak curves to leak the key piecewise.
77. **singular curve** — degenerate curve reduces to easy group. *Scenario:* a "curve" that's actually additive/multiplicative DLP.
78. **ECDSA nonce reuse** — same k in two signatures. *Scenario:* reused nonce → solve two equations for the private key.
79. **ECDSA biased nonce (HNP)** — leaked nonce bits. *Scenario:* top bits of k leak; lattice (LLL) recovers the key over many sigs.

## J. Lattices (the scary-looking ones)

80. **LLL basis reduction** — find short vectors = hidden small values. *Scenario:* a problem's secret is "small"; LLL surfaces it.
81. **CVP via Babai** — nearest lattice point. *Scenario:* model the key as closest vector to a target.
82. **knapsack/subset-sum** — low-density knapsack via LLL. *Scenario:* a Merkle-Hellman knapsack cipher falls to lattice reduction.
83. **approximate GCD** — hidden common factor with noise. *Scenario:* several near-multiples of a secret; LLL extracts it.
84. **hidden number problem** — recover secret from partial leaks. *Scenario:* MSBs of many products leak; lattice recovers the number.
85. **Coppersmith via lattices** — small roots of modular polynomials. *Scenario:* most of a prime known; find the small missing part.
86. **LWE/NTRU basics** — lattice-based scheme attacks. *Scenario:* a toy LWE with small noise is solvable by reduction.

## K. Padding, oracles, protocols

87. **padding oracle (CBC)** — see §E, core technique. *Scenario:* decrypt via validity leak, no key.
88. **compression oracle (CRIME/BREACH style)** — length leaks secret. *Scenario:* response size changes with your guess; recover a token char by char.
89. **timing attack** — response time leaks comparison progress. *Scenario:* `==` on secrets returns faster on a wrong first byte; time it.
90. **MAC-then-encrypt flaws** — order enables oracles. *Scenario:* wrong construction order opens a padding oracle.
91. **replay attack** — resend a valid message. *Scenario:* no nonce/timestamp; replay an admin action.
92. **downgrade/parameter confusion** — force weak params. *Scenario:* make the server use `alg=none` or a weak curve.

## L. Misc / modern

93. **JWT alg=none** — strip the signature. *Scenario:* set `alg:none`, forge an admin token.
94. **JWT weak HMAC secret** — crack the signing key. *Scenario:* `hashcat -m 16500` finds the secret, you sign anything.
95. **JWT RS→HS confusion** — sign with public key as HMAC secret. *Scenario:* server uses the public key as the HMAC key; forge tokens.
96. **secret sharing (Shamir)** — reconstruct from enough shares. *Scenario:* you have k-of-n shares; interpolate the secret.
97. **CRT recombination** — combine mod-pieces into one answer. *Scenario:* message recovered modulo several primes; CRT merges them.
98. **Z3 for crypto constraints** — brute logic the smart way. *Scenario:* a custom check expressed as equations; Z3 solves instantly.
99. **side-channel from error messages** — different errors leak secrets. *Scenario:* "invalid padding" vs "invalid mac" distinguishes cases.
100. **triage checklist habit** — identify → decode → classify → pick attack. *Scenario:* you waste no time: each challenge routed to the right tool fast.

## Sources
- ljagiello/ctf-skills (ctf-crypto): RSA, symmetric, ECC, lattice/LWE, PRNG, LFSR
- hacktricks crypto-symmetric; cryptopals; CryptoHack categories

> Tags: #ctf #crypto #techniques


---

# Crypto Catalog — Cryptography Tools & Scripts

Separate catalog from search. 100+ entries. Format: **name** — purpose `source`.

## All-in-one / auto

1. **CyberChef** — encode/decode/crypto web app, "Magic" op `gchq.github.io/CyberChef`
2. **FeatherDuster** — automated modular cryptanalysis `github.com/nccgroup/featherduster`
3. **Ciphey** — auto-decrypt/decode (AI-ish) `github.com/Ciphey/Ciphey`
4. **Ares** — Ciphey successor `github.com/bee-san/Ares`
5. **dcode.fr** — huge classical cipher solver site
6. **quipqiup.com** — substitution/frequency solver
7. **boxentriq.com** — cipher identifier + solvers
8. **cryptii.com** — chained conversions

## Classical ciphers

9. **Caesar/ROT-n** — brute 25 shifts (CyberChef ROT13 Brute)
10. **rot13/tr** — `tr 'A-Za-z' 'N-ZA-Mn-za-m'`
11. **Vigenère** — dcode / online kasiski
12. **vigenere-solver scripts** — keylen + freq
13. **Substitution** — quipqiup
14. **Atbash** — reverse alphabet (CyberChef)
15. **Rail fence / transposition** — dcode
16. **Playfair / Hill / ADFGVX** — dcode
17. **Bacon cipher** — dcode
18. **Morse** — CyberChef "From Morse Code"
19. **Baudot / ITA2** — CyberChef
20. **Enigma** — CyberChef "Enigma" / online emulators

## Encoding

21. **base64/32/16** — `base64 -d`, `base32 -d`, `xxd -r -p`
22. **base58 / base62 / base85 (ascii85)** — CyberChef
23. **base45** — CyberChef
24. **URL / HTML entity** — CyberChef
25. **UUencode / xxencode** — CyberChef
26. **Quoted-printable** — CyberChef
27. **Punycode** — CyberChef
28. **Brainfuck/esolang** — dcode interpreters
29. **basecrack** — auto base-N detect+decode `github.com/mufeedvh/basecrack`

## XOR

30. **xortool** — analyze multi-byte XOR `pip install xortool`
31. **xor-analyze** — keylen via IoC
32. **CyberChef XOR Brute Force**
33. **your xor_brute.py** — [[Scripts Index#Crypto — single-byte xor]]
34. **your xor_repeat.py** — repeating-key

## RSA

35. **RsaCtfTool** — many RSA attacks `github.com/RsaCtfTool/RsaCtfTool`
36. **factordb** — precomputed factorizations `factordb.com`
37. **your rsa_toolkit.py** — pq/gcd/cube/fermat/factordb
38. **yafu** — automated integer factorization `github.com/bbuhrow/yafu`
39. **cado-nfs** — number field sieve factoring
40. **msieve** — factoring (SIQS/NFS)
41. **primefac** — python factoring `pip install primefac`
42. **sympy.factorint** — small-n factoring
43. **Wiener's attack** — small d (RsaCtfTool / owiener) `pip install owiener`
44. **Boneh-Durfee** — d < N^0.292 (sage script)
45. **Coppersmith / small roots** — sage `small_roots`
46. **Håstad broadcast** — same m, multiple keys (CRT + root)
47. **Franklin-Reiter** — related messages
48. **common modulus attack** — same n, two e (ext. gcd)
49. **partial key exposure** — sage/Coppersmith
50. **LSB oracle / parity oracle** — binary-search decrypt

## Symmetric (AES/DES/etc.)

51. **ECB detection** — find repeated 16-byte blocks
52. **ECB byte-at-a-time** — decryption oracle script
53. **CBC bit-flipping** — flip plaintext via cipher XOR
54. **padding oracle: padbuster** — `padbuster` perl tool
55. **padding-oracle-attacker** — `npm i -g padding-oracle-attacker`
56. **padding oracle: python-paddingoracle** — lib
57. **CTR/GCM nonce reuse** — keystream/XOR recovery; forbidden-attack
58. **nonce-reuse GCM: crypto-attacks** — repo collection
59. **DES/3DES** — pycryptodome
60. **RC4** — keystream recovery scripts
61. **Salsa/ChaCha** — nonce reuse analysis

## Hashing / cracking

62. **hashcat** — GPU hash cracking `hashcat.net`
63. **john the ripper** — CPU cracking `openwall.com/john`
64. **hashid / hash-identifier** — ID hash type `pip install hashid`
65. **name-that-hash** — modern identifier `pip install name-that-hash`
66. **CrackStation** — online rainbow lookup
67. **hashpump / hash_extender** — length extension `github.com/iagox86/hash_extender`
68. **HashPump** — py bindings
69. **rockyou.txt** — the wordlist (SecLists)
70. **zip2john / rar2john / ssh2john / *2john** — extract hashes
71. **pdfcrack** — PDF password brute
72. **fcrackzip** — zip brute
73. **bkcrack** — ZipCrypto known-plaintext `github.com/kimci86/bkcrack`
74. **pkcrack** — classic PKZIP known-plaintext

## Math / advanced

75. **SageMath** — the big math toolkit `sagemath.org`
76. **cocalc.com** — Sage in the browser
77. **PARI/GP** — number theory CAS
78. **fpylll** — LLL/BKZ lattice reduction `pip install fpylll`
79. **flatter** — fast lattice reduction `github.com/keeganryan/flatter`
80. **Inequality/CVP solver (rkm0959)** — lattice CVP util
81. **g6k** — lattice sieving
82. **sage LLL/BKZ** — `.LLL()`, `.BKZ()`
83. **Z3 / z3-solver** — SMT for constraints
84. **Coppersmith (defund/coppersmith)** — multivariate `github.com/defund/coppersmith`
85. **lbc / lattice-based-cryptanalysis** — toolkit `github.com/josephsurin/lattice-based-cryptanalysis`

## Elliptic curve

86. **sage EllipticCurve** — ECC core
87. **Smart attack** — anomalous curve (p-adic) script
88. **MOV attack** — pairing-based ECDLP
89. **invalid curve attack** — scripts
90. **Pohlig-Hellman** — smooth-order DLP (sage `discrete_log`)
91. **ecdsa nonce reuse** — recover private key
92. **lattice ECDSA biased nonce** — HNP via LLL
93. **tsujihacking/ecc scripts** — ECC CTF utilities

## Discrete log / DH

94. **sage discrete_log** — generic DLP
95. **baby-step giant-step** — scripts
96. **Pollard's rho / kangaroo** — DLP
97. **CADO / index calculus** — large DLP

## Libraries / helpers

98. **pycryptodome** — `pip install pycryptodome`
99. **cryptography (pyca)** — `pip install cryptography`
100. **gmpy2** — fast bignum `pip install gmpy2`
101. **Crypto.Util.number** — long_to_bytes, inverse, getPrime
102. **sympy** — symbolic math, ntheory
103. **pwntools xor** — `from pwn import xor`
104. **CryptoHack toolkit** — practice + utilities `cryptohack.org`
105. **cryptopals** — the challenge set `cryptopals.com`

## Your scripts

- [[Scripts Index#Crypto — single-byte xor]] (`xor_brute.py`)
- [[Scripts Index#Crypto — repeating-key xor]] (`xor_repeat.py`)
- `rsa_toolkit.py` — [[Scripts Index#Crypto — RSA with p,q]]

## Sources

- awesome-ctf-resources (devploit), awesome-ctf (apsdehal)
- ljagiello/ctf-skills (ctf-crypto); medium CTF toolkit guide

> Tags: #ctf #crypto #catalog


---

# Crypto Scripts (100)

Copy-paste snippets for CTF crypto. 🟢 easy / 🟡 medium / 🔴 hard. Each is self-contained; edit the input line. Needs: `pip install pycryptodome gmpy2 requests`. Pairs with [[Crypto]], [[Crypto Techniques]], [[Crypto Catalog]].

> Flag grep after any decrypt: `print(re.findall(rb'[A-Za-z0-9_]*\{[^}]+\}', out))`

---

## Encoding / decoding

🟢 1. base64 decode
```python
import base64; print(base64.b64decode("ZmxhZ3t9"))
```

🟢 2. base32 decode
```python
import base64; print(base64.b32decode("MZXW6==="))
```

🟢 3. hex → bytes
```python
print(bytes.fromhex("666c6167"))
```

🟢 4. bytes → hex
```python
print(b"flag".hex())
```

🟢 5. base58 decode
```python
import base58; print(base58.b58decode("StV1DL"))   # pip install base58
```

🟢 6. ascii85 / base85 decode
```python
import base64; print(base64.a85decode("Bl7Q")); print(base64.b85decode("cmV3"))
```

🟢 7. URL decode
```python
from urllib.parse import unquote; print(unquote("fl%61g%7B%7D"))
```

🟢 8. HTML entity decode
```python
import html; print(html.unescape("flag&#123;&#125;"))
```

🟢 9. ROT13
```python
import codecs; print(codecs.decode("synt", "rot13"))
```

🟢 10. binary string → ascii
```python
b="0110011001101100"; print(bytes(int(b[i:i+8],2) for i in range(0,len(b),8)))
```

🟢 11. decimal list → ascii
```python
print(bytes([102,108,97,103]))
```

🟢 12. octal → ascii
```python
print(bytes(int(o,8) for o in "146 154 141 147".split()))
```

🟢 13. base62 decode
```python
import string
A=string.digits+string.ascii_uppercase+string.ascii_lowercase
def b62(s):
    n=0
    for c in s: n=n*62+A.index(c)
    return n.to_bytes((n.bit_length()+7)//8,'big')
print(b62("SsV1"))
```

🟡 14. auto-detect/decode (tool)
```bash
ciphey -t "ZmxhZ3t9"        # or: ares -t "..."
```

🟢 15. multi-base peel (try all, show printable)
```python
import base64
s="..."
for f in (base64.b64decode, base64.b32decode, base64.b16decode, base64.a85decode, base64.b85decode):
    try:
        d=f(s)
        if all(32<=c<127 for c in d): print(f.__name__, d)
    except Exception: pass
```

---

## Classical ciphers

🟢 16. Caesar brute (all 25)
```python
ct="iodj"
for k in range(26):
    print(k, "".join(chr((ord(c)-97-k)%26+97) if c.isalpha() else c for c in ct))
```

🟢 17. ROT-n on bytes (shift digits too)
```python
def rot(s,k): return "".join(chr((ord(c)-97+k)%26+97) if c.islower() else c for c in s)
print(rot("synt",13))
```

🟢 18. Atbash
```python
print("".join(chr(219-ord(c)) if c.islower() else c for c in "uozt"))
```

🟡 19. Vigenère decrypt (known key)
```python
def vig_dec(ct,key):
    o=""; j=0
    for c in ct:
        if c.isalpha():
            o+=chr((ord(c.lower())-ord(key[j%len(key)].lower()))%26+97); j+=1
        else: o+=c
    return o
print(vig_dec("rijvs","key"))
```

🔴 20. Vigenère key-length (Index of Coincidence)
```python
def ic(s): 
    from collections import Counter; n=len(s)
    return sum(v*(v-1) for v in Counter(s).values())/(n*(n-1)) if n>1 else 0
ct="".join(c for c in open("ct.txt").read().lower() if c.isalpha())
for k in range(1,20):
    avg=sum(ic(ct[i::k]) for i in range(k))/k
    print(k, round(avg,4))   # ~0.066 ⇒ likely keylen
```

🔴 21. Vigenère break (keylen known, freq per column)
```python
ct="".join(c for c in open("ct.txt").read().lower() if c.isalpha()); K=5
key=""
for i in range(K):
    col=ct[i::K]; best=(0,1e9)
    for g in range(26):
        d=[chr((ord(c)-97-g)%26+97) for c in col]
        chi=sum((d.count(ch)-len(d)*f)**2 for ch,f in zip("etaoin"," "*6)) # rough
        # simpler: pick shift making 'e' most common
    # practical: use 'e' heuristic
    from collections import Counter
    g=(Counter(col).most_common(1)[0][0].__class__)  # placeholder
print("use dcode.fr if this stalls")
```

🟢 22. substitution solve (tool)
```bash
# paste ciphertext at quipqiup.com  — frequency auto-solver
```

🟡 23. Rail fence decrypt
```python
def rf_dec(ct,rails):
    pat=list(range(rails))+list(range(rails-2,0,-1)); idx=[]
    for i in range(len(ct)): idx.append(pat[i%len(pat)])
    order=sorted(range(len(ct)), key=lambda i:(idx[i],i))
    out=[None]*len(ct)
    for pos,ch in zip(order,ct): out[pos]=ch
    return "".join(out)
print(rf_dec("fcalg",3))
```

🟡 24. Columnar transposition (known key order)
```python
def col_dec(ct,key):
    ncol=len(key); nrow=-(-len(ct)//ncol)
    cols=[ct[i*nrow:(i+1)*nrow] for i in range(ncol)]
    order=sorted(range(ncol), key=lambda i:key[i])
    grid=[""]*ncol
    for i,o in enumerate(order): grid[o]=cols[i]
    return "".join("".join(r) for r in zip(*grid))
```

🟢 25. Affine decrypt
```python
from sympy import mod_inverse
def affine_dec(ct,a,b):
    ai=mod_inverse(a,26)
    return "".join(chr(ai*((ord(c)-97)-b)%26+97) if c.isalpha() else c for c in ct)
print(affine_dec("ihhots",5,8))
```

🟡 26. Affine brute (a∈units, b 0-25)
```python
from sympy import mod_inverse
units=[a for a in range(1,26) if __import__("math").gcd(a,26)==1]
for a in units:
    for b in range(26):
        try: print(a,b,affine_dec("ihhots",a,b))
        except: pass
```

🟢 27. Bacon cipher decode
```python
m={}; import itertools,string
for i,(w) in enumerate(itertools.product("AB",repeat=5)):
    if i<26: m["".join(w)]=string.ascii_uppercase[i]
s="AABBBAAAAB"  # 5-char groups
print("".join(m[s[i:i+5]] for i in range(0,len(s),5)))
```

🟢 28. A1Z26 decode
```python
print("".join(chr(int(n)+96) for n in "6 12 1 7".split()))
```

🟢 29. Polybius (5x5, i/j) decode
```python
sq="ABCDEFGHIKLMNOPQRSTUVWXYZ"
p="214434"  # row,col pairs
print("".join(sq[(int(p[i])-1)*5+int(p[i+1])-1] for i in range(0,len(p),2)))
```

🟡 30. Morse decode
```python
M={'.-':'A','-...':'B','-.-.':'C','-..':'D','.':'E','..-.':'F','--.':'G','....':'H','..':'I','.---':'J','-.-':'K','.-..':'L','--':'M','-.':'N','---':'O','.--.':'P','--.-':'Q','.-.':'R','...':'S','-':'T','..-':'U','...-':'V','.--':'W','-..-':'X','-.--':'Y','--..':'Z'}
print("".join(M.get(c,'?') for c in ".-. . .".split()))
```

---

## XOR

🟢 31. single-byte XOR brute
```python
ct=bytes.fromhex("1b37")
for k in range(256):
    out=bytes(c^k for c in ct)
    if all(32<=c<127 for c in out): print(k, out)
```

🟡 32. single-byte XOR (best English score)
```python
ct=bytes.fromhex("1b37")
score=lambda b: sum(chr(c).lower() in "etaoin shrdlu" for c in b)
print(max(((bytes(c^k for c in ct),k) for k in range(256)), key=lambda x:score(x[0])))
```

🟢 33. XOR two hex buffers
```python
a=bytes.fromhex("1c0111"); b=bytes.fromhex("686974")
print(bytes(x^y for x,y in zip(a,b)).hex())
```

🟡 34. recover key from known-plaintext
```python
ct=bytes.fromhex("..."); known=b"flag{"
print(bytes(c^p for c,p in zip(ct,known)))   # reveals key bytes
```

🟡 35. repeating-key XOR (key known)
```python
from itertools import cycle
ct=bytes.fromhex("..."); key=b"KEY"
print(bytes(c^k for c,k in zip(ct,cycle(key))))
```

🔴 36. repeating-key XOR keylen (Hamming)
```python
def ham(a,b): return sum(bin(x^y).count("1") for x,y in zip(a,b))
ct=open("ct.bin","rb").read()
best=min(range(2,41), key=lambda k: ham(ct[:k*4],ct[k*4:k*8])/k)
print("keylen≈",best)
```

🔴 37. repeating-key XOR solve columns
```python
ct=open("ct.bin","rb").read(); K=7
key=bytes(max(range(256), key=lambda g: sum(32<=(c^g)<127 for c in ct[i::K])) for i in range(K))
from itertools import cycle
print(key, bytes(c^k for c,k in zip(ct,cycle(key))))
```

🔴 38. two-time pad crib drag
```python
c1=bytes.fromhex("..."); c2=bytes.fromhex("...")
x=bytes(a^b for a,b in zip(c1,c2)); crib=b" the "
for i in range(len(x)-len(crib)):
    print(i, bytes(a^b for a,b in zip(x[i:],crib)))
```

🟢 39. pwntools xor helper
```python
from pwn import xor; print(xor(b"flag", 0x42))
```

🟡 40. XOR until flag prefix appears
```python
ct=bytes.fromhex("...")
for k in range(256):
    if bytes(c^k for c in ct[:5])==b"flag{": print("key",k); break
```

---

## RSA

🟢 41. decrypt with p, q
```python
from Crypto.Util.number import inverse, long_to_bytes
p=..;q=..;e=65537;c=..
d=inverse(e,(p-1)*(q-1)); print(long_to_bytes(pow(c,d,p*q)))
```

🟢 42. decrypt with d, n
```python
from Crypto.Util.number import long_to_bytes
print(long_to_bytes(pow(c,d,n)))
```

🟢 43. factordb lookup
```python
import requests
r=requests.get(f"http://factordb.com/api?query={n}").json(); print(r["factors"])
```

🟡 44. Fermat factorization (p,q close)
```python
from gmpy2 import isqrt
def fermat(n):
    a=isqrt(n)+1
    while not (a*a-n>=0 and isqrt(a*a-n)**2==a*a-n): a+=1
    b=isqrt(a*a-n); return a-b,a+b
print(fermat(n))
```

🟡 45. Pollard rho factor
```python
from math import gcd
def rho(n):
    x=y=2;d=1
    f=lambda v:(v*v+1)%n
    while d==1: x=f(x);y=f(f(y));d=gcd(abs(x-y),n)
    return d
print(rho(n))
```

🟡 46. Pollard p-1 factor
```python
from math import gcd
def pm1(n,B=10**5):
    a=2
    for j in range(2,B): a=pow(a,j,n)
    return gcd(a-1,n)
print(pm1(n))
```

🟢 47. small-e cube root (e=3, no pad)
```python
from gmpy2 import iroot; from Crypto.Util.number import long_to_bytes
m,ok=iroot(c,3); print(ok, long_to_bytes(int(m)))
```

🔴 48. Håstad broadcast (same m, e=3, 3 keys)
```python
from Crypto.Util.number import long_to_bytes
from sympy.ntheory.modular import crt
from gmpy2 import iroot
ns=[n1,n2,n3]; cs=[c1,c2,c3]
x,_=crt(ns,cs); m,_=iroot(int(x),3); print(long_to_bytes(int(m)))
```

🔴 49. common modulus (same n, e1,e2)
```python
from Crypto.Util.number import long_to_bytes, inverse
from math import gcd
g,a,b=__import__("sympy").gcdex(e1,e2)  # or egcd
# egcd:
def egcd(x,y):
    if y==0:return x,1,0
    g,p,q=egcd(y,x%y);return g,q,p-(x//y)*q
_,a,b=egcd(e1,e2)
m=pow(c1,a,n)*pow(inverse(c2,n) if b<0 else c2, abs(b),n)%n
print(long_to_bytes(m))
```

🟡 50. common factor across two n (gcd)
```python
from math import gcd; from Crypto.Util.number import inverse, long_to_bytes
p=gcd(n1,n2); q=n1//p
d=inverse(e,(p-1)*(q-1)); print(long_to_bytes(pow(c1,d,n1)))
```

🔴 51. Wiener attack (small d)
```python
# pip install owiener
import owiener; d=owiener.attack(e,n); print("d=",d)
```

🟡 52. recover p,q from n and φ
```python
from gmpy2 import isqrt
def pq(n,phi):
    s=n-phi+1; dsq=isqrt(s*s-4*n); return (s+dsq)//2,(s-dsq)//2
print(pq(n,phi))
```

🟡 53. recover p,q from e,d,n
```python
import random
def factor_edn(e,d,n):
    k=e*d-1; t=k
    while t%2==0: t//=2
    while True:
        g=random.randrange(2,n); x=pow(g,t,n)
        while x!=1 and x!=n-1 and pow(x,2,n)!=1: x=pow(x,2,n)
        if pow(x,2,n)==1 and x not in (1,n-1):
            from math import gcd; return gcd(x-1,n)
print(factor_edn(e,d,n))
```

🔴 54. CRT params (dp,dq,p,q) decrypt
```python
from Crypto.Util.number import inverse, long_to_bytes
m1=pow(c,dp,p); m2=pow(c,dq,q); qinv=inverse(q,p)
h=(qinv*(m1-m2))%p; print(long_to_bytes(m2+h*q))
```

🔴 55. LSB oracle (parity) decrypt
```python
from Crypto.Util.number import long_to_bytes
# oracle(ct)->bit (m is odd?). edit to your service.
lo,hi=0,n
mul=pow(2,e,n); ct=c
for _ in range(n.bit_length()):
    ct=ct*mul%n
    if oracle(ct): lo=(lo+hi)//2
    else: hi=(lo+hi)//2
print(long_to_bytes(hi))
```

🔴 56. Coppersmith stereotyped (SageMath)
```python
# sage: most of m known, few unknown bytes
# R.<x>=PolynomialRing(Zmod(n)); f=(known+x)^e - c; f.small_roots(X=2^kbits,beta=1)
```

🔴 57. Boneh-Durfee (SageMath / repo)
```bash
# use github.com/mimoo/RSA-and-LLL-attacks  (boneh_durfee.sage)
```

🟡 58. signature forge (textbook multiplicative)
```python
# sig(m1*m2)=sig(m1)*sig(m2) mod n
forged=(s1*s2)%n; print(forged)
```

🟡 59. multi-prime RSA decrypt (n=p*q*r)
```python
from Crypto.Util.number import inverse, long_to_bytes
phi=(p-1)*(q-1)*(r-1); d=inverse(e,phi); print(long_to_bytes(pow(c,d,p*q*r)))
```

🟢 60. long_to_bytes / bytes_to_long
```python
from Crypto.Util.number import long_to_bytes, bytes_to_long
print(long_to_bytes(0x666c6167), bytes_to_long(b"flag"))
```

🟡 61. RsaCtfTool (throw everything)
```bash
RsaCtfTool -n <n> -e <e> --uncipher <c> --attack all
```

🟡 62. yafu factor large n
```bash
echo "factor(<n>)" | ./yafu
```

---

## AES / block ciphers

🟢 63. AES-ECB decrypt (key known)
```python
from Crypto.Cipher import AES
print(AES.new(key,AES.MODE_ECB).decrypt(ct))
```

🟢 64. AES-CBC decrypt (key+iv known)
```python
from Crypto.Cipher import AES
print(AES.new(key,AES.MODE_CBC,iv).decrypt(ct))
```

🟡 65. detect ECB (repeated 16-byte blocks)
```python
ct=bytes.fromhex("...")
blks=[ct[i:i+16] for i in range(0,len(ct),16)]
print("ECB" if len(blks)!=len(set(blks)) else "not ECB")
```

🟢 66. PKCS7 pad / unpad
```python
from Crypto.Util.Padding import pad, unpad
print(pad(b"flag",16)); print(unpad(padded,16))
```

🔴 67. ECB byte-at-a-time (oracle)
```python
# oracle(prefix)->AES_ECB(prefix+secret). recover secret 1 byte at a time.
known=b""
for i in range(32):
    pad=b"A"*(15-(len(known)%16))
    target=oracle(pad)[:16*(len(known)//16+1)]
    for g in range(256):
        if oracle(pad+known+bytes([g]))[:len(target)]==target:
            known+=bytes([g]); break
print(known)
```

🟡 68. ECB cut-and-paste (make admin)
```python
# craft blocks so "admin\x0b*11" aligns to its own block, then swap
# profile_for(email) -> ct; splice the admin block in. (edit to target)
```

🔴 69. CBC bit-flip (flip plaintext byte)
```python
ct=bytearray(ciphertext); i=16+off      # byte in block N+1 lives in block N's cipher
ct[i]^= ord(cur) ^ ord(want)
# submit bytes(ct)
```

🔴 70. CBC padding oracle (byte-by-byte)
```python
# oracle(iv,ct)->True if padding valid. decrypt one block:
def dec_block(prev,blk):
    I=bytearray(16); P=bytearray(16)
    for pb in range(1,17):
        for g in range(256):
            iv=bytearray(16); iv[-pb]=g
            for k in range(1,pb): iv[-k]=I[-k]^pb
            if oracle(bytes(iv),blk):
                I[-pb]=g^pb; P[-pb]=I[-pb]^prev[-pb]; break
    return bytes(P)
```

🟡 71. padding oracle (tool)
```bash
padbuster <URL> <sample> 16 -cookies "auth=<sample>"
```

🔴 72. CTR nonce reuse (keystream recovery)
```python
# same nonce used twice: ks = c1 ^ p1 ; p2 = c2 ^ ks
ks=bytes(a^b for a,b in zip(c1,p1_known))
print(bytes(a^b for a,b in zip(c2,ks)))
```

🟢 73. AES-GCM decrypt (key,nonce,tag)
```python
from Crypto.Cipher import AES
print(AES.new(key,AES.MODE_GCM,nonce=nonce).decrypt_and_verify(ct,tag))
```

🟡 74. DES decrypt (key known)
```python
from Crypto.Cipher import DES
print(DES.new(key,DES.MODE_ECB).decrypt(ct))
```

🟡 75. RC4 keystream
```python
def rc4(key,data):
    S=list(range(256));j=0
    for i in range(256): j=(j+S[i]+key[i%len(key)])%256; S[i],S[j]=S[j],S[i]
    i=j=0;out=bytearray()
    for b in data:
        i=(i+1)%256;j=(j+S[i])%256;S[i],S[j]=S[j],S[i];out.append(b^S[(S[i]+S[j])%256])
    return bytes(out)
print(rc4(b"key",ct))
```

---

## Hashing / MAC

🟢 76. md5/sha of string
```python
import hashlib; print(hashlib.md5(b"flag").hexdigest(), hashlib.sha256(b"flag").hexdigest())
```

🟢 77. identify hash
```bash
name-that-hash -t '<hash>'      # or: hashid '<hash>'
```

🟡 78. crack hash with wordlist (python)
```python
import hashlib
h="5f4dcc3b5aa765d61d8327deb882cf99"
for w in open("rockyou.txt","rb"):
    if hashlib.md5(w.strip()).hexdigest()==h: print(w.strip()); break
```

🟢 79. hashcat / john
```bash
hashcat -m 0 hash.txt rockyou.txt         # 0=MD5, 100=SHA1, 1400=SHA256
john --wordlist=rockyou.txt hash.txt
```

🔴 80. hash length extension (tool)
```bash
hashpump -s <sig> -d <orig_data> -a '&admin=1' -k <keylen>
```

🟢 81. HMAC compute
```python
import hmac,hashlib; print(hmac.new(b"key",b"msg",hashlib.sha256).hexdigest())
```

🟡 82. bcrypt verify / crack
```python
import bcrypt; print(bcrypt.checkpw(b"guess", b"$2b$..."))
```

🟢 83. crc32
```python
import zlib; print(hex(zlib.crc32(b"flag")))
```

🟡 84. zip/pdf → john hash
```bash
zip2john file.zip > h.txt ; john --wordlist=rockyou.txt h.txt
```

🔴 85. reverse CRC32 (brute 4-byte preimage)
```python
import zlib,itertools,string
target=0xdeadbeef; A=string.printable.encode()
for t in itertools.product(A,repeat=4):
    if zlib.crc32(bytes(t))==target: print(bytes(t)); break
```

---

## Stream / PRNG

🔴 86. MT19937 untemper + predict
```bash
pip install mersenne-twister-predictor
# feed 624 consecutive 32-bit outputs, then it predicts the next
```

🟡 87. LCG next-value solve
```python
# given X_{n+1}=(a*X_n+c)%m and a few outputs, recover a,c:
def crack_lcg(x):
    import math
    diffs=[x[i+1]-x[i] for i in range(len(x)-1)]
    m=0
    for i in range(len(diffs)-2):
        m=math.gcd(m, abs(diffs[i+2]*diffs[i]-diffs[i+1]**2))
    return m
print(crack_lcg([...]))
```

🟡 88. Python random seed reproduce
```python
import random; random.seed(1337); print([random.randint(0,99) for _ in range(5)])
```

🔴 89. LFSR step
```python
def lfsr(state,taps,n):
    out=[]
    for _ in range(n):
        b=0
        for t in taps: b^=(state>>t)&1
        out.append(state&1); state=(state>>1)|(b<<(max(taps)))
    return out
```

🔴 90. Berlekamp-Massey (LFSR recover)
```python
# use sage: berlekamp_massey(list_of_GF2) -> connection polynomial
```

---

## Public key / misc

🔴 91. discrete log (SageMath)
```python
# sage: discrete_log(h, g, ord=p-1)  — or Pohlig-Hellman if order smooth
```

🟡 92. Diffie-Hellman shared secret
```python
print(pow(B, a, p))   # your private a, their public B
```

🟡 93. ElGamal decrypt
```python
from Crypto.Util.number import inverse, long_to_bytes
s=pow(c1,x,p); m=c2*inverse(s,p)%p; print(long_to_bytes(m))   # x=private
```

🔴 94. ECDSA nonce reuse → private key
```python
from Crypto.Util.number import inverse
# two sigs same k: (r,s1,z1),(r,s2,z2)
k=(z1-z2)*inverse(s1-s2,nord)%nord
d=(s1*k-z1)*inverse(r,nord)%nord; print("priv",d)
```

🟡 95. Shamir secret share reconstruct (Lagrange)
```python
def recon(shares,p):
    s=0
    for i,(xi,yi) in enumerate(shares):
        num=den=1
        for j,(xj,_) in enumerate(shares):
            if i!=j: num=num*(-xj)%p; den=den*(xi-xj)%p
        s=(s+yi*num*pow(den,-1,p))%p
    return s
```

🟡 96. CRT combine (generic)
```python
from sympy.ntheory.modular import crt
print(crt([3,5,7],[2,3,2]))
```

🟡 97. modular inverse / egcd
```python
def egcd(a,b):
    if b==0:return a,1,0
    g,x,y=egcd(b,a%b);return g,y,x-(a//b)*y
print(egcd(17,3120))
```

🟡 98. nth root mod prime (Tonelli-Shanks via sympy)
```python
from sympy.ntheory.residue_ntheory import sqrt_mod
print(sqrt_mod(c, p, all_roots=True))
```

🟢 99. JWT decode (crypto-adjacent)
```python
import base64,json
h,p,s="eyJ...".split("."); pad=lambda x:x+"="*(-len(x)%4)
print(json.loads(base64.urlsafe_b64decode(pad(p))))
```

🔴 100. frequency table (any monoalphabetic)
```python
from collections import Counter
print(Counter(c for c in open("ct.txt").read().lower() if c.isalpha()).most_common())
```

---
> Notes: snippets marked SageMath need `sage`; a few (padding/LSB oracle) need you to wire `oracle()` to the challenge. Prefer [[Crypto Catalog]] tools when a snippet stalls.
> Tags: #ctf #crypto #scripts


---

## 🗺️ DCTF Roadmap — Crypto


### What to study (in order)
1. **Prereqs** — modular arithmetic, gcd, primes, modular inverse, CRT. (you can't do RSA without these)
2. **Encoding + classical** — base-N, ROT/Vigenère/substitution, XOR single & repeating-key. (sections A–C)
3. **RSA** — decrypt with p,q; then weak-key attacks: small-e, common factor, Fermat, Wiener, Håstad. (section D)
4. **Symmetric** — ECB detection + byte-at-a-time, CBC bit-flip, padding oracle, nonce reuse. (section E)
5. **Hashing** — identify + crack, length extension. (section G)
6. **Advanced math** — discrete log, ECC (nonce reuse, Smart, invalid curve), lattices (LLL, Coppersmith). (sections H–K)

### How to study
- **Do the math by hand once**, then script it (the snippets above are your reusable kit).
- **CryptoHack is the single best path** — it teaches by doing, in order.
- Build a personal `crypto_toolkit.py` from the Scripts section; every challenge adds one function.
- When stuck, name the *mistake* first ("they reused a nonce"), then pick the attack from [[Crypto Techniques]].

### How to exercise
- **CryptoHack** (Intro→General→Symmetric→RSA→Diffie-Hellman→ECC→Lattices).
- **cryptopals** sets 1–8 (the canonical grind).
- **picoCTF** crypto for warm-ups.
- Drill: identify the scheme + weakness within 5 minutes of reading a challenge; only then open an editor.

### How to do things (per-challenge workflow)
1. **Identify**: encoding or encryption? (if reversible with no key → just decode). (scripts 1–15)
2. **Classify**: RSA? AES mode? hash? classical? Read the source for the *mistake*.
3. **Match** the mistake to an attack
4. **Run** the matching script (above) or the tool (RsaCtfTool, CyberChef, SageMath).
5. `long_to_bytes` the result; grep for the flag.

### In the DCTF A/D final
- Rare, but a service may use crypto wrong (weak token signing, nonce reuse). 

> Tags: #ctf #crypto #roadmap #dctf

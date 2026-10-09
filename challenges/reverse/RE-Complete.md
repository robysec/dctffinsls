# RE Techniques — Reverse Engineering 


> Mental model: a binary is a locked machine. RE = figure out which lever opens it. Usually: read it (static), run it (dynamic), or prove the input with math (symbolic).

## A. First-pass / triage

1. **strings scan** — list readable text in the file. *Scenario:* flag is literally sitting in the binary; `strings bin | grep flag` wins instantly.
2. **file type check** — `file` tells you ELF/PE/arch/stripped. *Scenario:* you don't know if it's a Linux binary or a firmware blob; `file` decides your toolchain.
3. **symbol listing (nm/readelf)** — see function names if not stripped. *Scenario:* a function named `check_password` tells you exactly where to look.
4. **ltrace** — log library calls. *Scenario:* you see `strcmp("hunter2", input)` — the password is right there.
5. **strace** — log syscalls. *Scenario:* program reads a file you didn't know about; strace reveals the path.
6. **entropy check (binwalk -E)** — high entropy = packed/encrypted. *Scenario:* flat high entropy screams "this is UPX-packed."
7. **imports/exports review** — which APIs it uses. *Scenario:* seeing `ptrace` import hints at anti-debug before you even run it.
8. **rabin2 -I / checksec** — protections + metadata. *Scenario:* confirms PIE/stripped so you plan addressing.

## B. Static analysis

9. **decompile main (Ghidra)** — read C-like pseudocode. *Scenario:* the whole flag check is a readable `if` you can just invert on paper.
10. **cross-reference (xrefs)** — find who calls a function. *Scenario:* you found `print_flag`; xrefs show the one branch that reaches it.
11. **rename + retype variables** — label things as you learn them. *Scenario:* turning `local_38` into `user_len` makes a 200-line function readable.
12. **identify the compare** — locate the `strcmp`/`memcmp`/loop that validates. *Scenario:* everything funnels into one comparison; that's your target.
13. **constant/magic recognition** — spot known constants. *Scenario:* `0x6A09E667` means SHA-256; now you know the algorithm without reading it.
14. **data section reading** — dump arrays/tables. *Scenario:* an encrypted flag blob lives in `.data`; you copy it out to decrypt.
15. **control-flow graph reading** — see branches as a map. *Scenario:* a maze of jumps becomes obvious when you view the CFG picture.
16. **call graph reading** — see the function hierarchy. *Scenario:* quickly find the "interesting" leaf functions far from `main`.
17. **string deobfuscation (FLOSS)** — recover strings built at runtime. *Scenario:* strings shows nothing, but FLOSS reveals the flag assembled byte-by-byte.
18. **switch/jump-table recovery** — reconstruct big switches. *Scenario:* a command dispatcher's cases map directly to features you need.
19. **struct recovery** — rebuild a struct from offsets. *Scenario:* `*(ptr+8)` everywhere becomes `user->score` once you define the struct.
20. **algorithm fingerprinting (capa/yara)** — auto-ID behaviors. *Scenario:* capa flags "encrypts data with AES" so you stop guessing.

## C. Dynamic analysis / debugging

21. **set a breakpoint at the check** — pause right before validation. *Scenario:* break on `strcmp`, read both args, one is the real flag.
22. **register inspection** — read CPU registers at a point. *Scenario:* `$rax` holds a pointer to the decrypted flag at the moment of comparison.
23. **memory examine (x/)** — dump memory at an address. *Scenario:* `x/s $rsi` prints the expected password string.
24. **patch a register live** — change a value to steer execution. *Scenario:* set the "is_correct" register to 1 and walk straight to the win branch.
25. **step over/into** — execute one line at a time. *Scenario:* watch a transform build your flag one operation at a time.
26. **conditional breakpoint** — break only when a condition holds. *Scenario:* loop runs 10000 times; break only when `i==target`.
27. **watchpoint** — break when a variable changes. *Scenario:* catch exactly when the flag buffer gets written.
28. **call a function from the debugger** — invoke code directly. *Scenario:* just `call print_flag()` instead of reaching it the hard way.
29. **dump memory region to file** — save a buffer. *Scenario:* decrypted data lives in RAM only; dump it before the program frees it.
30. **heap/stack inspection** — view runtime layout. *Scenario:* find where your input landed relative to a secret.
31. **ltrace/strace with filters** — focus on one call family. *Scenario:* `ltrace -e 'strcmp'` cuts noise to just the comparisons.
32. **core-dump analysis** — inspect a crashed state. *Scenario:* program crashes after decrypting; the core dump still holds the plaintext.
33. **environment/arg fuzzing by hand** — try edge inputs. *Scenario:* empty input or a huge input flips a hidden code path.
34. **time-travel/record-replay (rr)** — rewind execution. *Scenario:* you overshot the key moment; rewind instead of restarting.

## D. Beating the math (transforms & crypto-in-RE)

35. **reverse a XOR** — undo `x ^ k`. *Scenario:* flag stored XOR'd with a byte; XOR again to recover.
36. **reverse add/sub/rotate** — invert arithmetic obfuscation. *Scenario:* each char was `+3`; subtract 3 to read the flag.
37. **reverse a byte-swap/endian** — fix order. *Scenario:* bytes look scrambled until you flip endianness.
38. **reimplement the check in Python** — port the logic. *Scenario:* rewrite the 20-line validator in Python and brute the input offline.
39. **write a keygen** — produce valid serials. *Scenario:* you understand the serial formula; generate a correct key for any name.
40. **invert a hash by lookup** — crack weak hashes. *Scenario:* the "password" is an MD5 of a common word; hashcat finds it.
41. **break custom crypto** — attack a homemade cipher. *Scenario:* a toy cipher with a short key falls to brute force.
42. **recover a PRNG seed** — predict "random" values. *Scenario:* the key uses `srand(time)`; you reproduce the same sequence.
43. **constant-folding by hand** — precompute fixed math. *Scenario:* collapse a chain of operations into one lookup.

## E. Symbolic / automated solving

44. **symbolic execution (angr)** — let a solver find the input. *Scenario:* 16 interdependent byte checks; angr returns the exact flag.
45. **find/avoid addresses** — guide the solver to "Correct". *Scenario:* tell angr to reach the success print and dodge "Wrong".
46. **constraint solving (Z3)** — express checks as equations. *Scenario:* the validator is pure math; Z3 solves for the input.
47. **concolic execution** — mix real + symbolic runs. *Scenario:* huge program, but only a few bytes matter; concolic isolates them.
48. **taint tracking** — follow where input flows. *Scenario:* see which bytes of input actually reach the comparison.
49. **emulate a snippet (Unicorn)** — run a function in isolation. *Scenario:* run just the decrypt routine on your data without the whole binary.
50. **full emulation (Qiling)** — run a foreign/partial binary. *Scenario:* a MIPS router binary runs on your x86 laptop so you can debug it.

## F. Packing / protection

51. **UPX unpack** — `upx -d`. *Scenario:* packed binary becomes readable instantly.
52. **manual unpacking (dump at OEP)** — let it unpack in RAM, then dump. *Scenario:* a custom packer has no tool; you dump after it decompresses itself.
53. **fix imports after dump (Scylla)** — repair the IAT. *Scenario:* dumped PE won't run until imports are rebuilt.
54. **find the original entry point** — locate real code start. *Scenario:* set a breakpoint after the unpack stub jumps to OEP.
55. **self-modifying code handling** — code rewrites itself. *Scenario:* breakpoints vanish; you break after the rewrite completes.
56. **inline-decrypt stubs** — functions decrypt themselves on call. *Scenario:* dump each function's plaintext as it's invoked.

## G. Anti-analysis bypass

57. **anti-debug: ptrace bypass** — neutralize `PTRACE_TRACEME`. *Scenario:* patch the ptrace call to return 0 so the debugger attaches.
58. **anti-debug: TracerPid check** — fake `/proc/self/status`. *Scenario:* program quits when debugged; you spoof TracerPid=0.
59. **anti-debug: IsDebuggerPresent** — patch the Windows check. *Scenario:* flip the result so it thinks it's un-debugged.
60. **timing-check bypass** — defeat rdtsc/clock checks. *Scenario:* single-stepping is "too slow"; you patch out the timing gate.
61. **anti-VM bypass** — defeat CPUID/registry checks. *Scenario:* malware-style challenge refuses to run in a VM; you patch the detector.
62. **LD_PRELOAD hook** — replace a libc function. *Scenario:* override `ptrace` or `time` globally without touching the binary.
63. **nanomite handling** — int3-based control flow. *Scenario:* missing instructions are supplied by a debugger handler; you emulate that logic.
64. **nop out a check** — overwrite with `0x90`. *Scenario:* delete an anti-tamper call so execution continues.

## H. Patching

65. **flip a conditional jump** — `je`↔`jne`, or to `jmp`. *Scenario:* turn "jump if wrong" into "jump if right."
66. **force a return value** — patch `mov eax, 1; ret`. *Scenario:* make `is_valid()` always say yes.
67. **nop a call** — remove a nag/anti-debug call. *Scenario:* delete the "exit if debugged" call.
68. **redirect a branch to win** — point a jump at `print_flag`. *Scenario:* short-circuit straight to the success code.
69. **patch a constant/comparison** — change the expected value. *Scenario:* make the length check accept your input size.
70. **hot-patch in debugger vs on-disk** — temporary vs permanent. *Scenario:* test a patch live, then bake it into the file.

## I. Architecture & format specifics

71. **ARM/AArch64 reversing** — different registers/calling conv. *Scenario:* an IoT challenge binary is ARM; you adjust gadgets and regs.
72. **MIPS reversing** — delay slots, o32 ABI. *Scenario:* a router firmware function behaves oddly due to branch delay slots.
73. **x86 vs x64 conventions** — arg passing differs. *Scenario:* on x64 the first arg is `rdi`, not the stack — set your breakpoint reads accordingly.
74. **.NET decompilation (dnSpy)** — near-source C#. *Scenario:* read and even edit the flag check, recompile, done.
75. **Java/APK (jadx)** — decompile dex to Java. *Scenario:* the Android app's flag logic reads like source.
76. **Python bytecode (pycdc)** — decompile .pyc. *Scenario:* a compiled Python challenge becomes readable code.
77. **PyInstaller unpack** — extract the embedded .pyc. *Scenario:* an EXE is really Python; pull out the script.
78. **WebAssembly (wasm2wat)** — read wasm as text. *Scenario:* a browser challenge's logic is in a .wasm you convert and read.
79. **Go binary reversing (GoReSym)** — recover stripped Go symbols. *Scenario:* a Go binary looks like mush until symbols are restored.
80. **demangle C++/Rust symbols** — human-readable names. *Scenario:* `_ZN4core...` becomes a real function name.

## J. VM / bytecode challenges

81. **identify the dispatch loop** — find the opcode switch. *Scenario:* one big `while(1){switch(op)}` is the whole custom CPU.
82. **map opcodes to operations** — decode the instruction set. *Scenario:* opcode 0x05 = XOR; now you can read the "program."
83. **write a disassembler** — turn bytecode into readable ops. *Scenario:* convert the byte blob into pseudo-assembly you can follow.
84. **hook the inner compare** — skip full devirtualization. *Scenario:* just watch what the VM compares your input against.
85. **brute-force the VM input** — easier than reversing. *Scenario:* only 4 unknown bytes drive the VM; try all.

## K. Advanced deobfuscation

86. **control-flow unflattening** — restore real order. *Scenario:* a flattened dispatcher hides simple logic; unflatten to see it.
87. **opaque-predicate removal** — delete always-true/false branches. *Scenario:* fake branches bloat the code; prune them.
88. **MBA simplification (GOOMBA/msynth)** — simplify mixed boolean-arith. *Scenario:* a monstrous expression is really just `a ^ b`.
89. **API-hashing resolution** — map hashes back to API names. *Scenario:* malware resolves functions by hash; you rebuild the import list.
90. **string-encryption recovery** — bulk-decrypt obfuscated strings. *Scenario:* decrypt all strings at once to expose the logic.
91. **instruction-trace inversion (Unicorn+Keystone)** — rebuild clean code from a trace. *Scenario:* record what actually ran, reassemble it readable.

## L. Workflow / strategy

92. **easy-first pass** — strings → ltrace/strace before reversing. *Scenario:* a third of RE challenges fall to strings alone; always try it first.
93. **binary diffing (BinDiff/Diaphora)** — compare two versions. *Scenario:* patched vs unpatched binary reveals exactly the vuln.
94. **dynamic-first for crypto-heavy bins** — let it decrypt, then grab. *Scenario:* don't reverse the cipher; just read the plaintext in memory.
95. **scripting the decompiler (headless Ghidra)** — automate analysis. *Scenario:* 50 similar binaries; a script extracts each flag.
96. **note-taking / annotation** — track findings as you go. *Scenario:* a big binary over hours; your renamed symbols are your map.
97. **compare to source of a known lib** — match against open source. *Scenario:* a function is just zlib; stop reversing and read zlib.
98. **focus on input-reaching code** — ignore everything your input can't touch. *Scenario:* prune 90% of the binary to the 10% that matters.
99. **recognize standard checks** — length, charset, checksum first. *Scenario:* the first gate is just "len==32"; satisfy it and move on.
100. **know when to brute vs reverse** — pick the cheaper path. *Scenario:* 3 unknown bytes → brute; 30 → reverse or use angr.

## Sources
- zhaoxuya520/reverse-skill (patterns, anti-analysis, advanced-tools)
- ljagiello/ctf-skills (ctf-reverse); UniBuc RE class notes

> Tags: #ctf #reverse #techniques


---

	# RE Catalog — Reverse Engineering Tools & Scripts

Separate catalog, built from web search + the awesome-ctf ecosystem. 100+ entries. Verify each repo is alive before relying on it. Format: **name** — purpose `install/source`.

Sources at bottom.

## Disassemblers / decompilers

1. **Ghidra** — NSA RE suite, decompiler + scripting `ghidra-sre.org`
2. **IDA Free** — industry disassembler, cloud decompiler `hex-rays.com`
3. **IDA Pro** — full commercial, Hex-Rays decompiler
4. **radare2** — CLI RE framework `github.com/radareorg/radare2`
5. **rizin** — radare2 fork, cleaner API `rizin.re`
6. **Cutter** — GUI over rizin + Ghidra decompiler `cutter.re`
7. **Binary Ninja** — scriptable, strong IL `binary.ninja`
8. **Hopper** — macOS/Linux disassembler+decompiler
9. **objdump** — binutils disassembler `apt install binutils`
10. **r2ghidra** — Ghidra decompiler inside r2 `r2pm -ci r2ghidra`
11. **dogbolt.org** — online: many decompilers side by side
12. **RetDec** — Avast retargetable decompiler `github.com/avast/retdec`
13. **Snowman** — C/C++ decompiler
14. **Reko** — decompiler, many arches `github.com/uxmal/reko`
15. **Relyze** — commercial Windows RE
16. **Capstone** — disassembly engine (lib) `capstone-engine.org`
17. **diStorm3** — x86/x64 disassembler lib
18. **XED** — Intel encoder/decoder

## Debuggers / dynamic

19. **gdb** — core debugger `apt install gdb`
20. **pwndbg** — gdb plugin for exploit/RE `github.com/pwndbg/pwndbg`
21. **GEF** — gdb enhanced features `github.com/hugsy/gef`
22. **PEDA** — older gdb plugin (unmaintained)
23. **gdb-multiarch** — cross-arch gdb `apt install gdb-multiarch`
24. **x64dbg** — Windows user-mode debugger `x64dbg.com`
25. **OllyDbg** — classic 32-bit Windows debugger
26. **lldb** — LLVM debugger (macOS)
27. **WinDbg** — Windows kernel/user debugger
28. **ltrace** — trace library calls `apt install ltrace`
29. **strace** — trace syscalls `apt install strace`
30. **frida** — dynamic instrumentation toolkit `pip install frida-tools`
31. **frida-trace** — auto-hook functions
32. **drcov / DynamoRIO** — coverage + instrumentation `dynamorio.org`
33. **PIN** — Intel dynamic binary instrumentation
34. **qemu-user** — run+debug foreign-arch bins `apt install qemu-user`
35. **valgrind** — memory/callgrind analysis

## Symbolic / concolic execution

36. **angr** — binary analysis + symbolic exec `pip install angr`
37. **angr management** — GUI for angr
38. **Triton** — dynamic symbolic execution `github.com/JonathanSalwan/Triton`
39. **Miasm** — RE framework, symbolic + jit `github.com/cea-sec/miasm`
40. **KLEE** — LLVM symbolic execution engine
41. **S2E** — selective symbolic execution
42. **manticore** — symbolic exec (bins + EVM) `pip install manticore`
43. **Z3** — SMT solver `pip install z3-solver`
44. **claripy** — angr's solver abstraction
45. **maat** — symbolic execution framework

## Emulation

46. **Unicorn** — CPU emulator (multi-arch) `pip install unicorn`
47. **Qiling** — emulation w/ OS/syscall layer `pip install qiling`
48. **unicorn + capstone combo** — trace & disasm emulated code
49. **speakeasy** — Windows malware emulator (Mandiant)
50. **libffi/qemu-system** — full-system emulation

## Binary diffing / matching

51. **BinDiff** — Zynamics binary diffing `zynamics.com/bindiff`
52. **Diaphora** — IDA diffing plugin `github.com/joxeankoret/diaphora`
53. **Ghidriff** — Ghidra-based diffing
54. **bsdiff / bspatch** — binary patch generation

## Deobfuscation

55. **D-810** — IDA deobfuscation (MBA) plugin
56. **GOOMBA** — Ghidra MBA simplifier
57. **gooMBA / msynth** — mixed boolean-arith synthesis
58. **unlzma / upx -d** — unpack UPX `upx -d bin`
59. **de4dot** — .NET deobfuscator
60. **jsnice / de4js** — JS deobfuscation (for JS RE)

## .NET

61. **dnSpy** — decompile + edit + debug `github.com/dnSpyEx/dnSpy`
62. **ILSpy** — .NET decompiler `github.com/icsharpcode/ILSpy`
63. **dotPeek** — JetBrains .NET decompiler
64. **monodis** — Mono IL disassembler
65. ** dnlib** — lib to read/write .NET assemblies

## Java / Android

66. **jadx** — dex->java decompiler `github.com/skylot/jadx`
67. **jd-gui** — java decompiler GUI
68. **procyon** — java decompiler
69. **CFR** — java decompiler
70. **apktool** — decode/rebuild APK resources `github.com/iBotPeaches/Apktool`
71. **dex2jar** — dex to jar
72. **Androguard** — Android analysis `pip install androguard`
73. **smali/baksmali** — dex assembler/disassembler
74. **objection** — frida-based mobile runtime
75. **Ghidra + dex** — load dex directly

## Python / bytecode

76. **uncompyle6** — py<=3.8 decompiler `pip install uncompyle6`
77. **decompyle3** — py 3.7-3.8
78. **pycdc** — C++ py decompiler (3.9+) `github.com/zrax/pycdc`
79. **dis module** — stdlib bytecode disasm `python -m dis`
80. **pyinstxtractor** — unpack PyInstaller EXE `github.com/extremecoders-re/pyinstxtractor`
81. **unpy2exe** — unpack py2exe
82. **xdis** — cross-version disassembler lib

## Other-language / format

83. **wasm2wat / wabt** — WebAssembly toolkit `github.com/WebAssembly/wabt`
84. **wasm-decompile** — wasm to pseudo-C
85. **luadec / unluac** — Lua bytecode decompile
86. **ActionScript (JPEXS)** — Flash/SWF decompiler
87. **Go analysis: GoReSym** — recover Go symbols (Mandiant)
88. **redress** — recover Go/ Rust type info
89. **Rust: cargo-expand** + symbol demangling
90. **c++filt** — demangle C++ symbols `c++filt <sym>`
91. **rustfilt** — demangle Rust symbols

## Helpers / inspection

92. **strings** — printable strings `strings -n 6 bin`
93. **FLOSS** — decode obfuscated strings (Mandiant) `github.com/mandiant/flare-floss`
94. **binwalk** — embedded file scan `github.com/ReFirmLabs/binwalk`
95. **nm / readelf / rabin2** — symbols + headers
96. **checksec** — binary protections (pwntools)
97. **Detect It Easy (DIE)** — packer/compiler ID `github.com/horsicq/Detect-It-Easy`
98. **PEiD / PEview** — PE packer ID / structure
99. **CFF Explorer** — PE editor
100. **yara** — pattern rules for triage `pip install yara-python`
101. **capa** — identify capabilities in bins (Mandiant) `github.com/mandiant/capa`
102. **cwe_checker** — flag bug patterns in bins
103. **Awesome-Binary-Analysis-Automation** — meta-list `github.com/user1342/Awesome-Binary-Analysis-Automation`
104. **flare-on challenges** — advanced RE practice `flare-on.com`

## Your own scripts (in vault)

- [[Scripts Index#RE — xor brute]] (`xor_brute.py`)
- [[Scripts Index#RE — angr template]] (`angr_solve.py`)
- Patch-byte snippet in [[Reverse Engineering#Patch a binary (skip a check)]]

## Sources

- awesome-ctf (apsdehal), awesome-ctf-resources (devploit)
- HackTricks reversing-tools; myctf.digitalpress.blog/reversing
- ljagiello/ctf-skills (ctf-reverse); Awesome-Binary-Analysis-Automation

> Tags: #ctf #reverse #catalog


---

# RE Scripts (100)

Snippets for reverse engineering. 🟢 easy / 🟡 medium / 🔴 hard. Mix of shell, gdb, Python, r2, Ghidra. Pairs with [[Reverse Engineering]], [[RE Techniques]], [[RE Catalog]].

---

## Triage

🟢 1. file type
```bash
file ./bin
```

🟢 2. strings with flag grep
```bash
strings -n 6 ./bin | grep -iE 'flag|ctf|\{'
```

🟢 3. all strings incl. wide/utf-16
```bash
strings -e l ./bin ; strings -e b ./bin
```

🟢 4. headers / protections
```bash
rabin2 -I ./bin ; checksec --file=./bin
```

🟢 5. strings with addresses (r2)
```bash
rabin2 -z ./bin
```

🟢 6. imports / exports
```bash
rabin2 -i ./bin ; nm -D ./bin
```

🟢 7. symbols (if not stripped)
```bash
nm ./bin | grep -i ' t \| T '
```

🟢 8. library calls at runtime
```bash
ltrace ./bin
```

🟢 9. syscalls at runtime
```bash
strace ./bin
```

🟢 10. entropy / embedded data
```bash
binwalk -E ./bin ; binwalk ./bin
```

🟡 11. decode obfuscated strings (FLOSS)
```bash
floss ./bin
```

🟡 12. identify capabilities (capa)
```bash
capa ./bin
```

---

## Static — objdump / radare2

🟢 13. disassemble (Intel)
```bash
objdump -d -M intel ./bin | less
```

🟢 14. disasm one function
```bash
objdump -d ./bin | awk '/<main>:/,/ret/'
```

🟢 15. r2 open + analyze
```bash
r2 -A ./bin
```

🟢 16. r2 list functions
```
afl
```

🟢 17. r2 disasm function
```
s main; pdf
```

🟢 18. r2 visual / panels
```
VV      # graph    |   V!   # panels
```

🟡 19. r2 xrefs to a function
```
axt @ sym.check_flag
```

🟡 20. r2 find strings + xref
```
iz~flag ; axt @ <addr>
```

🟡 21. r2 patch a byte (write mode)
```
r2 -w ./bin
s 0x1234; wx 90 90
```

🟡 22. r2 cfg to dot
```
agfd > cfg.dot
```

🟡 23. Ghidra headless auto-analyze
```bash
analyzeHeadless /proj P -import ./bin -postScript Decompile.java
```

🔴 24. Ghidra script: dump decompiled main
```java
// in Ghidra Script Manager (Java): getfunction "main", DecompInterface().decompileFunction()
```

🟡 25. decompile online (many at once)
```bash
# upload to dogbolt.org  — Ghidra/IDA/Binja/RetDec side-by-side
```

🟡 26. find comparison constants
```bash
objdump -d ./bin | grep -iE 'cmp|xor|test' | head
```

🟡 27. extract .data / .rodata blob
```bash
objcopy -O binary --only-section=.rodata ./bin rodata.bin ; xxd rodata.bin | head
```

🟡 28. demangle C++ symbols
```bash
nm ./bin | c++filt
```

🟡 29. demangle Rust symbols
```bash
nm ./bin | rustfilt
```

🟡 30. diff two binaries (BinDiff/radiff2)
```bash
radiff2 -C ./old ./new
```

---

## Dynamic — gdb

🟢 31. run under gdb (pwndbg/GEF)
```bash
gdb ./bin
```

🟢 32. break at main, run
```
b main
run
```

🟡 33. break on strcmp, read args
```
b strcmp
run
x/s $rdi
x/s $rsi      # often the real flag
```

🟡 34. break on memcmp/strncmp
```
b memcmp
commands
 printf "%s vs %s\n",$rdi,$rsi
 continue
end
```

🟡 35. examine memory
```
x/20xw $rsp
x/s 0x4040a0
```

🟡 36. patch a register to pass check
```
set $rax = 1
```

🟡 37. conditional breakpoint
```
b *0x401234 if $rdi==0x1337
```

🟡 38. watchpoint on a variable
```
watch *0x4040a0
```

🟡 39. call a function directly
```
call (int)print_flag()
```

🟡 40. dump memory to file
```
dump binary memory out.bin 0x4040a0 0x4040c0
```

🟡 41. finish / step
```
ni    # next instr
si    # step into
finish
```

🟡 42. info on mappings (find base)
```
info proc mappings     # pwndbg: vmmap
```

🟡 43. breakpoint on every call to malloc
```
b malloc
```

🔴 44. scripted gdb (batch)
```bash
gdb -q -batch -x solve.gdb ./bin
```

🔴 45. record/replay (rr)
```bash
rr record ./bin ; rr replay      # reverse-continue to the key moment
```

---

## Transforms in Python

🟢 46. reverse a XOR flag
```python
enc=bytes([...]); print(bytes(b^0x42 for b in enc))
```

🟢 47. reverse add/sub obfuscation
```python
print(bytes((b-3)&0xff for b in enc))
```

🟢 48. reverse rotate (ROL/ROR byte)
```python
ror=lambda b,n:((b>>n)|(b<<(8-n)))&0xff
print(bytes(ror(b,3) for b in enc))
```

🟡 49. reimplement a check & brute 1 char at a time
```python
import string
flag=""
for pos in range(32):
    for c in string.printable:
        if check(flag+c+"."*(31-len(flag))):   # port check() from decompiler
            flag+=c; break
print(flag)
```

🟡 50. keygen from serial formula
```python
def keygen(name):
    return sum(ord(c)*i for i,c in enumerate(name)) ^ 0xcafe   # ported logic
print(keygen("admin"))
```

🟡 51. endian swap
```python
import struct; print(struct.pack(">I", struct.unpack("<I",enc[:4])[0]))
```

🟡 52. base-N table lookup custom alphabet
```python
A="ZYXW..."; print("".join(A[i] for i in idxs))
```

🟡 53. decode per-char lookup table from binary
```python
tbl=bytes([...])  # dumped array
print(bytes(tbl[b] for b in enc))
```

🟡 54. undo multiply mod 256 (mult inverse)
```python
inv=pow(k,-1,256); print(bytes((b*inv)&0xff for b in enc))
```

🟡 55. CRC/checksum recompute to match
```python
import zlib; print(hex(zlib.crc32(data)))
```

---

## Symbolic execution (angr)

🟡 56. angr auto-solve stdin
```python
import angr
p=angr.Project("./bin",auto_load_libs=False)
sm=p.factory.simulation_manager(p.factory.entry_state())
sm.explore(find=lambda s:b"Correct" in s.posix.dumps(1), avoid=lambda s:b"Wrong" in s.posix.dumps(1))
print(sm.found[0].posix.dumps(0))
```

🟡 57. angr find/avoid by address
```python
sm.explore(find=0x401337, avoid=[0x401300])
```

🟡 58. angr symbolic argv
```python
import claripy
arg=claripy.BVS("arg",8*20)
st=p.factory.entry_state(args=["./bin",arg])
```

🟡 59. angr with stdin constraints (printable)
```python
flag=claripy.BVS("flag",8*32); st=p.factory.full_init_state(stdin=flag)
for b in flag.chop(8): st.solver.add(b>=0x20, b<=0x7e)
```

🔴 60. angr hook a function (skip/replace)
```python
p.hook_symbol("check", angr.SIM_PROCEDURES['stubs']['ReturnUnconstrained']())
```

🔴 61. angr load blob at address (shellcode/firmware)
```python
p=angr.load_shellcode(open("blob.bin","rb").read(), arch="amd64")
```

🔴 62. Z3 solve recovered constraints
```python
from z3 import *
x=[BitVec(f"x{i}",8) for i in range(8)]; s=Solver()
s.add(x[0]^x[1]==0x13, x[2]+x[3]==0x80)   # ported from decompiler
print(s.check(), s.model() if s.check()==sat else "")
```

🔴 63. angr avoid state explosion (veritesting)
```python
sm=p.factory.simulation_manager(st, veritesting=True)
```

🔴 64. angr dump solution as bytes
```python
print(sm.found[0].solver.eval(flag, cast_to=bytes))
```

🔴 65. emulate a function with Unicorn
```python
from unicorn import *; from unicorn.x86_const import *
mu=Uc(UC_ARCH_X86,UC_MODE_64); mu.mem_map(0,0x1000)
mu.mem_write(0,open("func.bin","rb").read()); mu.emu_start(0,len_)
```

---

## Packing / anti-debug / patching

🟢 66. UPX unpack
```bash
upx -d ./bin -o unpacked
```

🟡 67. detect packer
```bash
die ./bin        # Detect It Easy  (or: peid on windows)
```

🔴 68. dump unpacked from memory (gdb)
```
b *oep ; run ; dump binary memory dump.bin $base $end
```

🟡 69. patch je→jmp on disk (python)
```python
d=bytearray(open("bin","rb").read()); d[0x1234]=0xeb; open("bin2","wb").write(d)
```

🟡 70. nop a call (5 bytes)
```python
d=bytearray(open("bin","rb").read()); d[0x1234:0x1239]=b"\x90"*5; open("bin2","wb").write(d)
```

🟡 71. force function return 1
```python
# patch prologue to: mov eax,1; ret  => b8 01 00 00 00 c3
d[off:off+6]=b"\xb8\x01\x00\x00\x00\xc3"
```

🟡 72. bypass ptrace anti-debug (LD_PRELOAD)
```c
// ptrace.c:  long ptrace(int r,...){return 0;}
// gcc -shared -fPIC ptrace.c -o p.so ; LD_PRELOAD=./p.so ./bin
```

🟡 73. spoof TracerPid (gdb set)
```
# or patch the open("/proc/self/status") read / compare
```

🟡 74. defeat IsDebuggerPresent (x64dbg)
```
# set return 0, or use ScyllaHide plugin
```

🔴 75. find OEP after unpack stub
```
# break on the tail jmp that leaves the packer section; that target = OEP
```

🟡 76. rebuild imports after dump (Scylla)
```
# x64dbg → Scylla → IAT autosearch → fix dump
```

🟡 77. keystone assemble a patch
```python
from keystone import *; ks=Ks(KS_ARCH_X86,KS_MODE_64)
print(bytes(ks.asm("xor eax,eax; ret")[0]))
```

🟡 78. find virtual→file offset (for patching)
```bash
readelf -S ./bin    # vaddr - (sh_addr - sh_offset) = file offset
```

🟡 79. self-modifying: break after decrypt
```
b *after_decrypt ; run ; x/40i $pc
```

🔴 80. VM challenge: dump opcode handler table
```python
# locate switch/dispatch; map each case addr → operation, then write a disassembler
```

---

## Language-specific

🟢 81. .NET decompile (dnSpy/ILSpy)
```bash
ilspycmd ./app.dll > app.cs      # or open in dnSpy GUI to edit+recompile
```

🟢 82. Java jar decompile
```bash
jadx ./app.jar -d out/           # or: jd-gui app.jar
```

🟢 83. Android APK
```bash
apktool d app.apk ; jadx app.apk -d out/
```

🟢 84. Python .pyc decompile
```bash
pycdc file.pyc ; uncompyle6 file.pyc
```

🟡 85. unpack PyInstaller EXE
```bash
python pyinstxtractor.py app.exe   # then pycdc on the .pyc
```

🟡 86. Python dis module
```python
import dis, marshal
dis.dis(marshal.loads(open("x.pyc","rb").read()[16:]))
```

🟡 87. Go recover symbols
```bash
GoReSym -t ./bin > syms.json      # redress ./bin also works
```

🟡 88. WebAssembly to text
```bash
wasm2wat mod.wasm -o mod.wat ; wasm-decompile mod.wasm
```

🟡 89. Lua bytecode decompile
```bash
unluac luac.out > src.lua
```

🟡 90. Flash/SWF (JPEXS)
```bash
# ffdec app.swf  → ActionScript
```

🟡 91. frida hook a function (dynamic)
```bash
frida-trace -i 'strcmp' ./bin
```

🟡 92. frida read return value
```js
Interceptor.attach(Module.getExportByName(null,"strcmp"),{onLeave(r){console.log(r)}})
```

---

## Automation / misc

🟡 93. batch strings over many files
```bash
for f in *; do echo "== $f"; strings "$f" | grep -i flag; done
```

🟡 94. grep flag across extracted tree
```bash
grep -rniE 'flag\{|ctf\{' . 2>/dev/null
```

🟢 95. hexdump a region
```bash
xxd -s 0x100 -l 64 ./bin
```

🟡 96. compare bytes of two files
```bash
cmp -l a.bin b.bin | head
```

🟡 97. extract function bytes for emulation
```bash
dd if=./bin of=func.bin bs=1 skip=$((0x1169)) count=120
```

🟡 98. scriptable r2 pipe (r2pipe)
```python
import r2pipe; r=r2pipe.open("./bin"); r.cmd("aaa"); print(r.cmd("pdf @ main"))
```

🟡 99. angr-based CFG recovery
```python
cfg=p.analyses.CFGFast(); print(len(cfg.graph.nodes()))
```

🟢 100. the universal RE flow
```python
# strings → ltrace/strace → decompile the check → reverse the math OR angr it → verify
```

---
> Some entries are shell (r2/gdb) rather than Python. Patch offsets and symbol names per binary. Prefer [[RE Catalog]] GUIs for big targets.
> Tags: #ctf #reverse #scripts


---

## 🗺️ DCTF Roadmap — Reverse Engineering

> DCTF (DefCamp CTF) = online **Jeopardy** quals → onsite **Attack & Defense** final. RE shows up in quals as crackmes/keygens/VMs, and in finals as "understand this binary service fast so you can find its bug." See [[Roadmap]] and [[DCTF Finals Playbook]].

### What to study (in order)
1. **Prereqs** — C (pointers, structs), x86-64 asm (registers, stack, System V args: rdi,rsi,rdx,rcx,r8,r9), how a stack frame works.
2. **Static reading** — Ghidra decompiler: read `main`, rename vars, find the compare. (sections A–B above)
3. **Dynamic** — gdb+pwndbg: breakpoints on strcmp/scanf, read registers/memory, patch a register. (section C)
4. **Transforms** — reverse XOR/add/rotate, reimplement the check in Python, write a keygen. (section D)
5. **Symbolic** — angr/Z3 to auto-solve multi-byte checks. (section E)
6. **Protections** — UPX + manual unpack, anti-debug (ptrace/TracerPid), patching. (sections F–H)
7. **Advanced** — VM/bytecode, obfuscation (flattening/MBA), ARM/MIPS, .NET/Java/Go/wasm. (sections I–K)

### How to study
- **Read then reproduce.** For every solved crackme, re-solve it a second way (static vs dynamic vs angr).
- **One tool deep, not ten shallow.** Master Ghidra + gdb first; add angr once those are reflex.
- **Keep a tricks note** — each new constant (e.g. `0x6A09E667`→SHA-256), each anti-debug pattern. Add it to [[RE Techniques]].
- **Read write-ups** of challenges you failed; redo them from scratch a day later.

### How to exercise
- **crackmes.one** (diff 1→4), **reversing.kr**, **pico CTF** reversing, **pwn.college** "reversing" modules.
- **flare-on** for a serious yearly grind (hard, great).
- Drill: given a binary, find the flag in <15 min using only `strings`+`ltrace`+Ghidra; time yourself.

### How to do things (per-challenge workflow)
1. `file` → `strings` → `ltrace`/`strace` (a third of RE falls here). (scripts 1–12 above)
2. Decompile `main` in Ghidra; find the one comparison that gates success.
3. Decide: **reverse the math** (few bytes) or **angr it** (many interdependent checks).
4. If it runs anti-debug/packing, patch it out, then go back to step 2.
5. Verify the recovered input against the binary before submitting.

### In the DCTF A/D final
- You won't "keygen" — you'll **read a provided binary service** to find its memory bug, then hand it to the pwn person. Speed of reading > elegance. Use the triage flow above, find the dangerous input path, done.

> Tags: #ctf #reverse #roadmap #dctf

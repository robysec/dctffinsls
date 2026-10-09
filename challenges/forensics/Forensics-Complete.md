# Forensics Techniques — explained + scenarios


> Mental model: a flag is hidden *in* a file, image, capture, or dump. Your job is to notice where it's tucked and pull it out. Start wide (triage), then narrow by file type.

## A. File triage (always first)

1. **file-type check** — `file` reads magic bytes. *Scenario:* a ".txt" is really a PNG; now you know how to open it.
2. **magic-byte inspection** — read the header in hex. *Scenario:* header says `PK` → it's a zip, not the stated type.
3. **fix a broken/missing header** — patch the magic bytes. *Scenario:* a corrupted PNG opens once you restore `89 50 4E 47`.
4. **strings scan** — pull readable text. *Scenario:* the flag is plainly sitting in the bytes.
5. **binwalk signature scan** — find embedded files. *Scenario:* an image has a zip hidden inside it.
6. **binwalk extract** — carve the embedded files out. *Scenario:* `binwalk -e` drops the hidden archive to disk.
7. **entropy analysis** — spot encrypted/compressed regions. *Scenario:* a high-entropy blob marks where encrypted data begins.
8. **metadata read (exiftool)** — comments, GPS, author. *Scenario:* the flag is in the "Comment" EXIF field.
9. **hexdump pattern spotting** — eyeball structure. *Scenario:* repeating 16-byte blocks hint at ECB or a table.
10. **nested-extraction loop** — extract, then triage the result. *Scenario:* zip→image→zip; repeat until plaintext.

## B. Carving & recovery

11. **header/footer carving (foremost)** — rebuild files from raw data. *Scenario:* carve a JPEG out of a disk blob with no filesystem.
12. **configurable carving (scalpel)** — custom signatures. *Scenario:* carve an unusual file type foremost misses.
13. **photorec recovery** — recover many formats. *Scenario:* pull deleted photos off an image dump.
14. **deleted-file recovery (fls/icat)** — Sleuth Kit inode recovery. *Scenario:* a deleted file still has an inode; `icat` dumps its contents.
15. **orphaned inode recovery (fsck)** — ext2/3/4 leftovers. *Scenario:* recover an orphaned inode the directory no longer lists.
16. **slack-space inspection** — leftover data past a file's end. *Scenario:* the flag hides in the slack after a small file.
17. **anti-carving awareness** — null-byte interleaving fools carvers. *Scenario:* a file split by nulls needs manual reassembly.
18. **bulk_extractor sweep** — pull emails/URLs/cards from any blob. *Scenario:* scan a huge dump for anything flag-shaped fast.

## C. Image forensics / stego

19. **LSB extraction** — read least-significant bits. *Scenario:* zsteg finds the flag in the blue channel LSB.
20. **zsteg all-methods** — try every bit order/channel. *Scenario:* `zsteg -a` surfaces a hidden zip in a PNG.
21. **bit-plane viewing (Stegsolve)** — inspect each plane. *Scenario:* plane 0 of red reveals hidden text as an image.
22. **channel splitting** — isolate R/G/B/A. *Scenario:* the alpha channel hides a QR code.
23. **steghide extract** — keyed JPEG/BMP/WAV extraction. *Scenario:* empty passphrase pops the payload out.
24. **steghide passphrase crack (stegseek)** — brute with a wordlist. *Scenario:* rockyou finds the passphrase in seconds.
25. **error-level analysis (ELA)** — spot edited regions. *Scenario:* a pasted-in area glows differently, marking hidden content.
26. **image diff/compare** — two near-identical images. *Scenario:* pixel diff of before/after reveals the hidden message.
27. **metadata/thumbnail mismatch** — embedded thumbnail differs from image. *Scenario:* the original (uncensored) picture lives in the EXIF thumbnail.
28. **PNG chunk analysis** — extra/after-IEND data. *Scenario:* data appended past IEND holds the flag.
29. **palette/transparency tricks** — hidden via color index/alpha. *Scenario:* text written in a near-invisible palette color.
30. **appended-archive detection** — image + zip concatenation. *Scenario:* `unzip image.png` extracts a hidden file.
31. **polyglot handling** — one file, two valid formats. *Scenario:* the PNG is also a valid PDF with the flag.
32. **auto-everything (AperiSolve)** — run all image tools at once. *Scenario:* upload once, get zsteg/steghide/strings results together.

## D. Audio forensics

33. **spectrogram viewing** — see images/text in frequencies. *Scenario:* the flag is "drawn" in the spectrogram.
34. **waveform inspection** — visual anomalies. *Scenario:* a burst of noise is actually encoded data.
35. **WAV LSB extraction** — bits in samples. *Scenario:* read sample LSBs to recover hidden bytes.
36. **channel isolation** — left vs right differ. *Scenario:* the flag is only in the right channel.
37. **DTMF decoding (multimon-ng)** — phone tones. *Scenario:* beeps decode to a phone-keypad string.
38. **FSK/modem decode (minimodem)** — data-over-audio. *Scenario:* a modem screech decodes to text.
39. **SSTV decoding** — slow-scan TV image in audio. *Scenario:* ham-radio audio renders a picture with the flag.
40. **MP3/audio stego extract** — mp3stego/AudioStego. *Scenario:* hidden payload pulled from an MP3.
41. **tempo/pitch shift** — hidden content at other speeds. *Scenario:* slowing playback reveals spoken letters.

## E. Network (pcap) forensics

42. **protocol hierarchy review** — what's in the capture. *Scenario:* Wireshark stats show unexpected FTP traffic to follow.
43. **follow TCP/UDP stream** — reassemble a conversation. *Scenario:* following the stream shows a cleartext login + flag.
44. **export objects (HTTP/SMB/TFTP)** — pull transferred files. *Scenario:* a transferred zip extracted straight from the pcap.
45. **credential hunting** — grep for user/pass. *Scenario:* a Basic-Auth header base64-decodes to creds.
46. **HTTP file_data extraction** — dump payloads via tshark. *Scenario:* reconstruct an uploaded image from request bodies.
47. **DNS exfiltration decode** — data in subdomains/queries. *Scenario:* base32 chunks across DNS names reassemble the flag.
48. **ICMP exfiltration** — data in ping payloads. *Scenario:* each echo packet carries one flag byte.
49. **TCP-flag covert channel** — bits in flag fields. *Scenario:* SYN/ACK patterns encode a binary message.
50. **packet-timing covert channel** — data in inter-arrival gaps. *Scenario:* long/short delays encode 1s and 0s.
51. **USB HID keyboard decode** — reconstruct typed keys. *Scenario:* `usb.capdata` → your usb_hid_decode.py prints what was typed.
52. **USB mass-storage carving** — files over USB. *Scenario:* reassemble a file copied to a USB stick in the capture.
53. **TLS with given key** — decrypt HTTPS. *Scenario:* the provided key/SSLKEYLOG lets Wireshark show plaintext.
54. **WiFi crack (aircrack-ng)** — WEP/WPA handshake. *Scenario:* crack the PSK, then decrypt the captured traffic.
55. **layered pcap decode** — XOR+zip inside packets. *Scenario:* payload is XOR'd then zipped; undo both.
56. **pcap repair (pcapfix)** — fix a broken capture. *Scenario:* a truncated file opens after repair.
57. **anomaly spotting** — odd ports/sizes/hosts. *Scenario:* one host beaconing on a weird port is the C2 with the flag.

## F. Memory forensics

58. **image profiling (windows.info)** — identify the OS build. *Scenario:* confirm the dump's version so plugins work.
59. **process listing (pslist/pstree)** — what was running. *Scenario:* a suspicious `nc.exe` process points to the action.
60. **command-line recovery (cmdline)** — how processes started. *Scenario:* a command line contains the flag as an argument.
61. **file scan + dump (filescan/dumpfiles)** — recover files from RAM. *Scenario:* dump a document that was only ever open in memory.
62. **network connections (netscan)** — sockets at capture time. *Scenario:* an active connection reveals an exfil server.
63. **credential dumping (hashdump/lsadump)** — creds in memory. *Scenario:* recover a password hash to crack.
64. **registry in memory (hivelist/printkey)** — config/secrets. *Scenario:* a run key or stored value holds the flag.
65. **bash history (linux.bash)** — commands run on Linux. *Scenario:* shell history literally shows `echo flag{...}`.
66. **process memory dump (memmap/vaddump)** — a process's pages. *Scenario:* carve the plaintext flag from a browser's memory.
67. **raw-dump string carving** — strings over the whole dump. *Scenario:* grep the dump for `flag{` as a last resort.
68. **GIMP raw-image view** — see a dump as pixels. *Scenario:* a framebuffer in memory renders the on-screen flag.
69. **clipboard/console recovery** — plugins for clip/consoles. *Scenario:* the copied flag sits in the clipboard artifact.
70. **malware extraction from memory** — dump an injected payload. *Scenario:* pull the unpacked malware that only exists in RAM.
71. **MemProcFS mounting** — browse memory as a filesystem. *Scenario:* explore processes/files via a mounted folder instead of commands.

## G. Disk / filesystem forensics

72. **partition layout (mmls)** — find volumes + offsets. *Scenario:* locate the hidden partition's start offset.
73. **filesystem stats (fsstat)** — FS type/details. *Scenario:* confirm it's NTFS so you parse the right structures.
74. **file listing (fls)** — enumerate files incl. deleted. *Scenario:* spot a deleted file flagged with `*`.
75. **content extraction (icat)** — dump a file by inode. *Scenario:* recover the deleted note's contents.
76. **Autopsy triage** — GUI timeline/keyword/artifacts. *Scenario:* keyword-search the whole image for "flag" at once.
77. **mount an image** — loop-mount to browse. *Scenario:* mount read-only and copy files out normally.
78. **E01/VM image mounting** — ewfmount/guestmount. *Scenario:* browse a forensic E01 or a VM's vmdk.
79. **hidden/alternate data streams** — NTFS ADS. *Scenario:* `file.txt:secret` stream holds the flag.
80. **boot sector / MBR/GPT review** — code/data in boot area. *Scenario:* the flag is stashed in unused boot-sector bytes.
81. **free-space carving** — recover from unallocated space. *Scenario:* a wiped-but-not-overwritten file is carved back.

## H. OS artifacts (DFIR)

82. **Windows registry parsing (RegRipper)** — persistence/config. *Scenario:* a Run key shows the malware path + flag.
83. **event-log analysis (evtx)** — logon/exec history. *Scenario:* event 4688 logs the process that printed the flag.
84. **prefetch analysis (PECmd)** — what ran and when. *Scenario:* prefetch proves a tool executed at a key time.
85. **$MFT parsing (MFTECmd)** — NTFS file records/timestamps. *Scenario:* the MFT reveals a since-deleted file's name.
86. **shellbags** — folders a user browsed. *Scenario:* shellbags show the attacker opened a specific directory.
87. **browser history/cache** — URLs, downloads, cookies. *Scenario:* history shows the paste-site holding the flag.
88. **jump lists / LNK files** — recently used items. *Scenario:* an LNK points to the exfiltrated document.
89. **scheduled tasks / services** — persistence. *Scenario:* a rogue task reveals the attacker's script.
90. **recycle bin ($I/$R)** — "deleted" files. *Scenario:* the flag file sits in the recycle bin metadata.

## I. Documents / archives / email

91. **Office macro analysis (olevba)** — VBA in docs. *Scenario:* a macro contains the base64 flag/payload.
92. **OLE stream inspection (oledump)** — embedded objects. *Scenario:* an embedded object hides the flag.
93. **PDF object/stream analysis (pdf-parser)** — hidden streams/JS. *Scenario:* a PDF stream decompresses to the flag.
94. **PDF attachment extraction** — detached files. *Scenario:* a PDF has a hidden embedded file.
95. **zip/rar cracking (fcrackzip/john)** — password archives. *Scenario:* crack the zip password, extract the flag.
96. **ZipCrypto known-plaintext (bkcrack)** — one known file breaks the archive. *Scenario:* you know one file inside; bkcrack decrypts the rest.
97. **broken-archive repair** — fix zip headers. *Scenario:* `zip -FF` repairs a truncated archive.
98. **docx/xlsx as zip** — just unzip the Office file. *Scenario:* an image or comment hidden in the Office XML parts.
99. **email header/attachment analysis** — .eml/.msg. *Scenario:* a base64 attachment decodes to the flag.

## J. Strategy

100. **triage-first discipline** — file→exiftool→binwalk→strings on everything, every time. *Scenario:* you never waste an hour on "stego" that was just an appended zip.

## Sources
- ljagiello/ctf-skills (ctf-forensics): memory, network, disk, carving patterns
- IISc forensics webinar; FAU forensics specialization; andreafortuna DFIR list

> Tags: #ctf #forensics #techniques


---

# Forensics Catalog — Tools & Scripts

Separate catalog from search. 100+ entries. Covers file/stego/network/memory/disk. Format: **name** — purpose `source`.

## Triage / file ID

1. **file** — type by magic `file x`
2. **xxd / hexdump** — hex view `xxd x | head`
3. **binwalk** — embedded files + entropy `github.com/ReFirmLabs/binwalk`
4. **binwalk -e** — auto extract
5. **foremost** — header/footer carving `apt install foremost`
6. **scalpel** — configurable carving
7. **photorec** — recover many file types `apt install testdisk`
8. **bulk_extractor** — pull emails/urls/cards from blobs
9. **strings** — printable text
10. **Detect It Easy (DIE)** — file/packer ID
11. **TrID** — file type by signature
12. **hachoir** — parse binary structures `pip install hachoir`
13. **exiftool** — metadata `apt install libimage-exiftool-perl`

## Image stego

14. **zsteg** — PNG/BMP LSB `gem install zsteg`
15. **zsteg -a** — all methods
16. **steghide** — JPEG/BMP/WAV/AU stego `apt install steghide`
17. **stegseek** — fast steghide cracker `github.com/RickdeJager/stegseek`
18. **stegsolve** — bit-plane viewer (Java)
19. **AperiSolve** — web multi-tool `github.com/Zeecka/AperiSolve`
20. **outguess** — JPEG stego `apt install outguess`
21. **jsteg / slink** — JPEG LSB
22. **stegpy** — LSB in images/audio `pip install stegpy`
23. **LSBSteg (stego-lsb)** — `pip install stego-lsb`
24. **openstego** — stego + watermark
25. **exiv2** — metadata alt
26. **pngcheck** — PNG chunk integrity `apt install pngcheck`
27. **pngcsum / optipng** — fix PNG CRC/size
28. **TweakPNG** — edit PNG chunks (GUI)
29. **jhead** — JPEG header tool
30. **jpeginfo / jpegtran** — JPEG repair
31. **identify (ImageMagick)** — image info
32. **convert (ImageMagick)** — manipulate/compare
33. **compare (ImageMagick)** — diff images
34. **steghide bruteforce scripts** — wordlist loop
35. **stegoVeritas** — auto image stego `pip install stegoveritas`
36. **cloacked-pixel** — AES LSB tool
37. **F5 steganography** — JPEG F5
38. **zbarimg** — read QR/barcodes `apt install zbar-tools`
39. **qrtools / pyzbar** — QR decode in python

## Audio stego

40. **Audacity** — spectrogram + waveform (GUI)
41. **Sonic Visualiser** — spectrogram analysis
42. **sox** — audio convert/spectrogram `sox in.wav -n spectrogram`
43. **spectrology** — text->spectrogram (and reverse study)
44. **DTMF decoders (multimon-ng)** — tones `apt install multimon-ng`
45. **minimodem** — decode FSK/modem audio
46. **wav LSB scripts** — custom
47. **deepsound** — audio stego (Windows)
48. **morse audio decoder** — online / scripts

## Archives / passwords

49. **7z / p7zip** — extract many formats
50. **unzip / zipinfo** — zip listing
51. **fcrackzip** — zip brute `apt install fcrackzip`
52. **zip2john + john** — zip hash crack
53. **bkcrack** — ZipCrypto known-plaintext `github.com/kimci86/bkcrack`
54. **pkcrack** — PKZIP known-plaintext
55. **rar2john / unrar** — rar crack
56. **pdfcrack** — PDF password
57. **hashcat / john** — general cracking
58. **zip -FF** — repair broken zip

## PDF / documents

59. **pdf-parser.py** — Didier Stevens `blog.didierstevens.com`
60. **pdfid.py** — PDF object summary
61. **peepdf** — interactive PDF analysis
62. **qpdf** — decrypt/linearize PDFs
63. **pdfimages / pdftotext (poppler)** — extract
64. **origami** — Ruby PDF framework
65. **oletools (olevba, oleid)** — Office macros `pip install oletools`
66. **oledump.py** — OLE streams
67. **docx/xlsx = zip** — just `unzip` it
68. **mraptor** — macro malware detector

## Network (pcap)

69. **Wireshark** — GUI packet analysis
70. **tshark** — CLI wireshark
71. **tcpdump** — capture/read
72. **tcpflow** — reassemble TCP streams
73. **NetworkMiner** — passive host/file extraction
74. **Scapy** — craft/parse packets `pip install scapy`
75. **pyshark** — python wrapper over tshark
76. **Zeek (Bro)** — network analysis framework
77. **Suricata** — IDS, pcap replay
78. **ngrep** — grep over packets
79. **foremost on pcap** — carve transferred files
80. **USB HID extraction** — `tshark -e usb.capdata`
81. **your usb_hid_decode.py** — [[Scripts Index#Forensics — USB HID keyboard decode]]
82. **usbrip** — USB event history
83. **ciscodump / exports** — follow-stream helpers
84. **wireshark "Export Objects"** — HTTP/SMB/TFTP files
85. **krackattacks / wifi: aircrack-ng** — WEP/WPA `apt install aircrack-ng`
86. **tshark -z follow** — stream dump

## Memory forensics

87. **Volatility 3** — RAM analysis `pip install volatility3`
88. **Volatility 2** — older profiles (still needed)
89. **vol windows.pslist/pstree/cmdline** — processes
90. **vol windows.filescan + dumpfiles** — recover files
91. **vol windows.hashdump / lsadump** — creds
92. **vol windows.netscan** — connections
93. **vol linux.bash** — shell history
94. **MemProcFS** — mount memory as filesystem `github.com/ufrisk/MemProcFS`
95. **Rekall** — memory framework (legacy)
96. **bulk_extractor on memdump** — strings/artifacts
97. **avml** — acquire Linux memory
98. **windows.dumpfiles / procdump** — extract exe

## Disk / filesystem

99. **The Sleuth Kit (mmls/fls/icat/fsstat)** — disk analysis `sleuthkit.org`
100. **Autopsy** — GUI over Sleuth Kit
101. **FTK Imager** — imaging + preview (free)
102. **testdisk** — partition recovery
103. **ewfmount / xmount** — mount E01 images
104. **guestmount / libguestfs** — mount VM disks
105. **plaso / log2timeline** — super timeline
106. **mactime** — timeline from TSK body
107. **RegRipper** — Windows registry parsing
108. **registry-explorer / shellbags** — Eric Zimmerman tools
109. **evtx_dump / python-evtx** — Windows event logs
110. **prefetch parser (PECmd)** — EZ tools
111. **MFTECmd** — parse $MFT
112. **chainsaw / hayabusa** — event-log hunting

## Misc / encoding-in-forensics

113. **CyberChef** — decode whatever you carve
114. **base64/hex pipelines** — shell
115. **magic byte reference** — [[Forensics#Magic bytes (fix wrong/missing header)]]
116. **your flaghunt.sh** — triage wrapper

## Install quick ref (Debian/Kali/WSL)

```bash
sudo apt install -y binwalk foremost scalpel testdisk bulk-extractor \
  steghide outguess pngcheck imagemagick sox audacity zbar-tools \
  fcrackzip pdf-parser poppler-utils sleuthkit autopsy wireshark tshark \
  aircrack-ng multimon-ng libimage-exiftool-perl
pip install volatility3 scapy pyshark oletools hachoir stegoveritas stego-lsb
gem install zsteg
# stegseek, bkcrack, MemProcFS: install from their GitHub releases
```

## Sources

- wovari/CTF-Tools; GohEeEn/CTF-and-Computer-Security-Tools
- ljagiello/ctf-skills (ctf-forensics); awesome-ctf; andreafortuna DFIR list

> Tags: #ctf #forensics #catalog


---

# Forensics Scripts (100)

Snippets for forensics + stego. 🟢 easy / 🟡 medium / 🔴 hard. Shell + Python. Pairs with [[Forensics]], [[Forensics Techniques]], [[Forensics Catalog]], [[Stego Catalog]].

---

## Triage (run first on anything)

🟢 1. file type
```bash
file mystery.bin
```

🟢 2. magic bytes
```bash
xxd mystery.bin | head
```

🟢 3. metadata
```bash
exiftool mystery.*
```

🟢 4. embedded files
```bash
binwalk mystery.bin
```

🟢 5. extract embedded files
```bash
binwalk -e mystery.bin
```

🟢 6. strings + flag grep
```bash
strings -n 6 mystery.bin | grep -iE 'flag|ctf|\{'
```

🟢 7. raw flag grep (binary)
```bash
grep -aoiE '(flag|ctf|dctf)\{[^}]+\}' mystery.bin
```

🟢 8. full triage wrapper
```bash
bash 99-Scripts/flaghunt.sh mystery.bin
```

🟡 9. identify by signature (TrID/DIE)
```bash
trid mystery.bin ; die mystery.bin
```

🟡 10. check for appended data past EOF
```python
d=open("img.png","rb").read(); i=d.rfind(b"IEND")+8; print(d[i:i+64])
```

🟡 11. entropy map
```bash
binwalk -E mystery.bin
```

🟡 12. hex around an offset
```bash
xxd -s 0x400 -l 128 mystery.bin
```

---

## Carving / recovery

🟢 13. fix PNG header
```python
d=bytearray(open("broken.png","rb").read()); d[0:8]=bytes([0x89,0x50,0x4e,0x47,0x0d,0x0a,0x1a,0x0a]); open("fixed.png","wb").write(d)
```

🟢 14. fix JPEG header
```python
d=bytearray(open("b.jpg","rb").read()); d[0:3]=bytes([0xff,0xd8,0xff]); open("f.jpg","wb").write(d)
```

🟢 15. foremost carve
```bash
foremost -i blob.bin -o out/
```

🟢 16. scalpel carve
```bash
scalpel -c /etc/scalpel/scalpel.conf -o out/ blob.bin
```

🟢 17. photorec recover
```bash
photorec disk.img
```

🟡 18. bulk_extractor sweep
```bash
bulk_extractor -o be_out/ disk.img
```

🟡 19. carve by magic in python
```python
d=open("blob.bin","rb").read(); i=d.find(b"\x89PNG"); j=d.find(b"IEND",i)+8; open("c.png","wb").write(d[i:j])
```

🟡 20. list PNG chunks
```bash
pngcheck -v img.png
```

🟡 21. fix PNG CRCs / dimensions
```bash
pngcsum img.png fixed.png     # or edit IHDR width/height with a hex editor
```

🟡 22. gzip/zlib decompress a blob
```python
import zlib; open("out","wb").write(zlib.decompress(open("blob","rb").read()))
```

---

## Images / stego

🟢 23. zsteg (PNG/BMP LSB)
```bash
zsteg img.png
```

🟢 24. zsteg all methods
```bash
zsteg -a img.png
```

🟢 25. steghide info/extract
```bash
steghide info img.jpg ; steghide extract -sf img.jpg
```

🟢 26. stegseek (brute steghide pass)
```bash
stegseek img.jpg rockyou.txt
```

🟡 27. stegoveritas (run all checks)
```bash
stegoveritas img.png
```

🟡 28. AperiSolve (web/all-in-one)
```bash
# upload at aperisolve.com (zsteg+steghide+outguess+exif+binwalk+foremost+strings)
```

🟡 29. extract LSB plane in python (PIL)
```python
from PIL import Image
px=Image.open("img.png").convert("RGB").getdata()
bits="".join(str(px[i][0]&1) for i in range(len(px)))
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:64])
```

🟡 30. LSB per-channel, choose channel
```python
from PIL import Image
im=Image.open("img.png").convert("RGB"); w,h=im.size; p=im.load()
for ch in range(3):
    bits="".join(str(p[x,y][ch]&1) for y in range(h) for x in range(w))
    out=bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))
    if b"flag" in out: print(ch,out[:80])
```

🟡 31. dump all 8 bit planes as images
```python
from PIL import Image
im=Image.open("img.png").convert("L")
import numpy as np; a=np.array(im)
for b in range(8): Image.fromarray(((a>>b)&1)*255).convert("1").save(f"plane{b}.png")
```

🟡 32. XOR two images
```python
from PIL import Image, ImageChops
Image.open("a.png"); ImageChops.difference(Image.open("a.png"),Image.open("b.png")).save("diff.png")
```

🟡 33. error-level analysis (ELA) quick
```python
from PIL import Image, ImageChops
im=Image.open("x.jpg").convert("RGB"); im.save("t.jpg",quality=90)
ImageChops.difference(im,Image.open("t.jpg")).save("ela.png")
```

🟡 34. outguess extract
```bash
outguess -r img.jpg out.txt
```

🟡 35. extract embedded zip from image
```bash
unzip img.png      # many "images" are also zips
```

🟡 36. read QR / barcode
```bash
zbarimg img.png
```

🟡 37. decode QR in python
```python
from pyzbar.pyzbar import decode; from PIL import Image
print(decode(Image.open("qr.png")))
```

🟡 38. Stegsolve (bit planes, GUI)
```bash
java -jar stegsolve.jar
```

🟡 39. GIF frame split
```bash
convert anim.gif frames/%03d.png     # ImageMagick
```

🟡 40. compare metadata thumbnail to image
```bash
exiftool -b -ThumbnailImage x.jpg > thumb.jpg
```

🟡 41. zero-width unicode reveal
```python
t=open("msg.txt",encoding="utf-8").read()
print("".join("1" if c=="​" else "0" if c=="‌" else "" for c in t))
```

🟡 42. whitespace (snow) extract
```bash
stegsnow -C file.txt
```

---

## Audio

🟢 43. spectrogram (sox)
```bash
sox audio.wav -n spectrogram -o spec.png
```

🟢 44. open for spectrogram/waveform (GUI)
```bash
audacity audio.wav     # view → spectrogram; also Sonic Visualiser
```

🟡 45. WAV LSB extract
```python
import wave; w=wave.open("a.wav","rb"); f=w.readframes(w.getnframes())
bits="".join(str(b&1) for b in f); print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:80])
```

🟡 46. split stereo channels
```bash
sox a.wav left.wav remix 1 ; sox a.wav right.wav remix 2
```

🟡 47. DTMF decode
```bash
multimon-ng -a DTMF -t wav a.wav
```

🟡 48. FSK / modem decode
```bash
minimodem --rx 1200 < a.wav
```

🟡 49. slow playback (reveal speech)
```bash
sox a.wav slow.wav speed 0.5
```

🟡 50. extract audio from video
```bash
ffmpeg -i vid.mp4 -vn audio.wav
```

---

## Network (pcap)

🟢 51. protocol overview (tshark)
```bash
tshark -r cap.pcap -q -z io,phs
```

🟢 52. HTTP requests
```bash
tshark -r cap.pcap -Y http.request -T fields -e http.host -e http.request.uri
```

🟢 53. export HTTP objects (files)
```bash
tshark -r cap.pcap --export-objects http,out/
```

🟡 54. follow a TCP stream
```bash
tshark -r cap.pcap -q -z follow,tcp,ascii,0
```

🟡 55. dump HTTP bodies
```bash
tshark -r cap.pcap -Y http -T fields -e http.file_data | xxd -r -p > body.bin
```

🟡 56. grep credentials
```bash
tshark -r cap.pcap -Y 'http.authorization' -T fields -e http.authorization
```

🟡 57. extract DNS queries (exfil)
```bash
tshark -r cap.pcap -Y dns.qry.name -T fields -e dns.qry.name | sort -u
```

🟡 58. reassemble DNS-exfil payload
```python
import base64
names=open("dns.txt").read().split()
print(base64.b32decode("".join(n.split(".")[0] for n in names).upper()+"==="))
```

🟡 59. ICMP payload data
```bash
tshark -r cap.pcap -Y icmp -T fields -e data
```

🟡 60. USB HID keyboard capdata
```bash
tshark -r cap.pcap -Y usb.capdata -T fields -e usb.capdata > hid.txt
```

🟡 61. decode USB HID (your script)
```bash
python3 99-Scripts/usb_hid_decode.py hid.txt
```

🟡 62. USB mouse movements → drawing
```python
# parse signed dx,dy from usb.capdata, plot cumulative x,y with matplotlib
```

🟡 63. extract files with tcpflow
```bash
tcpflow -r cap.pcap -o flows/
```

🟡 64. decrypt TLS with keylog
```bash
# Wireshark → Prefs → TLS → (Pre)-Master-Secret log = sslkeys.log
```

🟡 65. WiFi crack handshake
```bash
aircrack-ng -w rockyou.txt cap.pcap
```

🟡 66. repair a broken pcap
```bash
pcapfix cap.pcap
```

---

## Memory (Volatility 3)

🟢 67. identify image
```bash
vol -f mem.raw windows.info
```

🟢 68. process list
```bash
vol -f mem.raw windows.pslist ; vol -f mem.raw windows.pstree
```

🟢 69. command lines
```bash
vol -f mem.raw windows.cmdline
```

🟡 70. scan for files
```bash
vol -f mem.raw windows.filescan
```

🟡 71. dump a file by offset
```bash
vol -f mem.raw windows.dumpfiles --virtaddr 0xADDR
```

🟡 72. network connections
```bash
vol -f mem.raw windows.netscan
```

🟡 73. credential hashes
```bash
vol -f mem.raw windows.hashdump
```

🟡 74. registry key value
```bash
vol -f mem.raw windows.registry.printkey --key "Software\\Microsoft\\Windows\\CurrentVersion\\Run"
```

🟡 75. dump a process memory
```bash
vol -f mem.raw windows.memmap --pid 1234 --dump
```

🟡 76. Linux bash history
```bash
vol -f mem.raw linux.bash
```

🟡 77. strings on whole dump
```bash
strings -n 8 mem.raw | grep -i flag
```

🟡 78. view raw dump as image (GIMP)
```bash
# open mem.raw in GIMP as "Raw image data", scrub width to find framebuffer/flag
```

🟡 79. MemProcFS mount
```bash
memprocfs -device mem.raw -mount M:
```

🟡 80. clipboard / console plugins
```bash
vol -f mem.raw windows.consoles ; vol -f mem.raw windows.clipboard
```

---

## Disk / filesystem

🟢 81. partition layout
```bash
mmls disk.img
```

🟢 82. filesystem stats
```bash
fsstat -o 2048 disk.img
```

🟡 83. list files incl. deleted
```bash
fls -r -o 2048 disk.img
```

🟡 84. extract a file by inode
```bash
icat -o 2048 disk.img 1234 > recovered.bin
```

🟡 85. mount image read-only
```bash
sudo mount -o ro,loop,offset=$((2048*512)) disk.img /mnt
```

🟡 86. mount E01 / vmdk
```bash
ewfmount disk.E01 /mnt/ewf ; guestmount -a disk.vmdk -i /mnt/vm
```

🟡 87. NTFS alternate data stream
```bash
# fls shows name:stream ; icat it. On windows: dir /r ; more < file:stream
```

🟡 88. timeline (plaso)
```bash
log2timeline.py out.plaso disk.img ; psort.py -o l2tcsv out.plaso > timeline.csv
```

🟡 89. registry parse (RegRipper)
```bash
rip.pl -r NTUSER.DAT -f ntuser
```

🟡 90. event logs
```bash
evtx_dump.py Security.evtx | grep -i 4688
```

---

## Documents / archives

🟢 91. crack zip
```bash
fcrackzip -D -u -p rockyou.txt secret.zip
```

🟢 92. zip → john
```bash
zip2john secret.zip > h ; john --wordlist=rockyou.txt h
```

🟡 93. ZipCrypto known-plaintext (bkcrack)
```bash
bkcrack -C secret.zip -c known.txt -p plain.bin
```

🟡 94. repair broken zip
```bash
zip -FF broken.zip --out fixed.zip
```

🟡 95. PDF objects / streams
```bash
pdf-parser.py doc.pdf ; pdfdetach -list doc.pdf
```

🟡 96. extract PDF text/images
```bash
pdftotext doc.pdf - ; pdfimages -all doc.pdf out/
```

🟡 97. Office macro (olevba)
```bash
olevba doc.docm
```

---

## Automation / glue

🟡 98. recurse-extract until nothing left
```bash
while binwalk -e $(ls -t _*.extracted 2>/dev/null | head -1 || echo start) 2>/dev/null; do :; done
```

🟡 99. grep flag across extracted tree
```bash
grep -rniaE 'flag\{|ctf\{|dctf\{' . 2>/dev/null
```

🟢 100. the universal forensics flow
```bash
# file → exiftool → binwalk → strings ; then branch by type:
#   image→zsteg/steghide ; audio→spectrogram ; pcap→follow streams ;
#   mem→volatility ; disk→sleuthkit ; archive→crack. Always triage first.
```

---
> Set the `-o` partition offset (sectors) from `mmls` output. Volatility 3 symbol packs may need downloading on first run.
> Tags: #ctf #forensics #stego #scripts


---

## 🗺️ DCTF Roadmap — Forensics

> DCTF forensics is a **Jeopardy quals** category: pull a flag out of a file/image/pcap/dump. It rewards patience and a disciplined triage habit. See [[Roadmap]].

### What to study (in order)
1. **File triage** — magic bytes, `file`/`exiftool`/`binwalk`/`strings`, fixing broken headers. (section A)
2. **Carving/recovery** — foremost/scalpel/photorec, deleted files (fls/icat). (section B)
3. **Stego** — image LSB (zsteg), JPEG (steghide/stegseek), audio spectrogram. (section C–D, and [[Stego-Complete]])
4. **Network** — Wireshark/tshark: follow streams, export objects, DNS/ICMP exfil, USB HID. (section E)
5. **Memory** — Volatility 3: pslist/cmdline/filescan/hashdump/bash. (section F)
6. **Disk + DFIR** — Sleuth Kit, registry, event logs, timelines. (sections G–H)

### How to study
- **Triage-first discipline is the whole game** — the fastest forensics players never skip `file→exiftool→binwalk→strings`. (technique 100)
- Build the branch reflex: once you know the type, you know the 3 tools to try.
- Learn to *read* a pcap (Follow Stream, Export Objects) before scripting it.
- Volatility: learn 8 plugins well, not all 100.

### How to exercise
- **picoCTF** forensics (easy→medium), **HTB** forensics, DFIR-style challenges (Magnet/SANS).
- Make your own: `steghide embed` a flag, then crack it back; capture your own USB-keyboard pcap and decode it.
- Drill: given a mystery file, name its true type + hidden-data location in <10 min.

### How to do things (per-challenge workflow)
1. `file` → `exiftool` → `binwalk` → `strings` on everything. (scripts 1–8)
2. Branch by type:
   - **image** → zsteg / steghide / Stegsolve ([[Stego-Complete]])
   - **audio** → spectrogram → LSB → DTMF
   - **pcap** → follow streams / export objects / USB HID (scripts 51–61)
   - **memory** → Volatility (scripts 67–80)
   - **disk** → Sleuth Kit (scripts 81–90)
   - **archive** → crack / bkcrack / repair
3. Extract, then re-triage the result (nesting is common).
4. `grep -rniaE 'flag\{|ctf\{'` the whole extracted tree.

### In the DCTF A/D final
- Not a finals category directly, but memory/pcap skills help **incident response on your own box** — spot an attacker's traffic in your capture and reproduce their exploit.

> Tags: #ctf #forensics #roadmap #dctf

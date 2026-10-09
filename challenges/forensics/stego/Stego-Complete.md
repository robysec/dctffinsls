# Stego Catalog — Steganography Tools & Scripts


> First pass on ANY suspicious file (do this before reaching for stego tools):
> `file x` → `exiftool x` → `binwalk x` → `strings -n 6 x`. Half of "stego" is just an appended zip or a metadata comment.

## Triage / toolkits

1. ⭐ **stego-toolkit (DominicBreuker)** — Docker image with all the tools + screening scripts `github.com/DominicBreuker/stego-toolkit`
2. ⭐ **AperiSolve** — web + CLI, runs zsteg/steghide/outguess/exiftool/binwalk/foremost/strings at once `github.com/Zeecka/AperiSolve` / aperisolve.com
3. ⭐ **StegOnline** — browser tool for image bit operations `georgeom.net/StegOnline`
4. ⭐ **cyberteach360/Steganography** — tutorial repo + tool list `github.com/cyberteach360/Steganography`
5. **check-all script** — loop file/exiftool/binwalk/strings/zsteg over a file

## Images — analysis / detection

6. ⭐ **zsteg** — PNG/BMP LSB detector (best first tool) `gem install zsteg`
7. **zsteg -a** — try all methods/bit-orders
8. ⭐ **StegoVeritas** — metadata + transformed images + LSB brute `pip install stegoveritas`
9. ⭐ **Stegsolve** — bit-plane viewer, channel filters, image math (Java)
10. ⭐ **Stegextract** — bash: pull hidden files/strings from PNG/JPG/GIF `github.com/evyatarmeged/Stegextract`
11. **pngcheck** — validate/ dump PNG chunks `apt install pngcheck`
12. **TweakPNG** — edit/inspect PNG chunks (GUI)
13. **pngcsum** — fix PNG CRCs
14. **exiftool** — metadata/comments `apt install libimage-exiftool-perl`
15. **exiv2** — metadata alt
16. **identify (ImageMagick)** — image properties
17. **compare (ImageMagick)** — pixel-diff two near-identical images
18. **convert -separate** — split channels for inspection
19. **GIMP** — manual bit-plane / color-curve inspection (also raw memory dumps)
20. **jsteg / slink** — detect/extract JPEG LSB `github.com/lukechampine/jsteg`
21. **jphide/jphs seek** — JP Hide&Seek detection
22. **stegdetect** — statistical JPEG stego detector
23. **StegSpy** — detect common stego signatures
24. **forensically (web)** — online error-level analysis (ELA)
25. **FotoForensics (web)** — ELA + metadata
26. **zbarimg / pyzbar** — decode QR/barcodes hidden in images

## Images — hide / extract (keyed)

27. ⭐ **steghide** — hide/extract in JPEG/BMP/WAV/AU (passphrase) `apt install steghide`
28. **steghide info** — show if a file has embedded data
29. ⭐ **stegseek** — ultra-fast steghide passphrase cracker `github.com/RickdeJager/stegseek`
30. **outguess** — JPEG stego hide/extract `apt install outguess`
31. **openstego** — LSB hide + watermarking (GUI)
32. **stegpy** — LSB in PNG/BMP/GIF/WebP/WAV `pip install stegpy`
33. **stego-lsb (LSBSteg)** — LSB encode/decode `pip install stego-lsb`
34. **cloacked-pixel** — AES-encrypted LSB `github.com/livz/cloacked-pixel`
35. **F5 steganography** — F5 JPEG algorithm tool
36. **outguess-rebirth** — maintained outguess variant
37. **hstego** — modern cost-based image/audio stego
38. **invisible-watermark** — robust watermark embed/extract

## Audio

39. ⭐ **Sonic Visualiser** — spectrogram analysis (find images/text in frequencies)
40. **Audacity** — waveform + spectrogram view, channel split (GUI)
41. **sox** — CLI spectrogram `sox in.wav -n spectrogram -o out.png`
42. **spectrology** — encode/decode text as spectrogram image
43. ⭐ **mp3stego** — hide/extract in MP3 (needs WAV input) `github.com` (fabienpe/MP3Stego)
44. ⭐ **AudioStego (hideme)** — MP3/WAV hide + recover `github.com/danielcardeenas/AudioStego`
45. **DeepSound** — audio stego (Windows GUI)
46. **multimon-ng** — decode DTMF/POCSAG/FSK tones `apt install multimon-ng`
47. **minimodem** — decode FSK/modem audio `apt install minimodem`
48. **wav LSB scripts** — custom per-sample LSB extract
49. **morse audio decoder (web/scripts)** — tones → text
50. **QSSTV / SSTV decoders** — slow-scan TV images in audio

## Video / GIF

51. **ffmpeg frame extract** — `ffmpeg -i v.mp4 frames/%04d.png` then image-stego each
52. **ffmpeg audio extract** — pull the audio track for spectrogram
53. **gifsicle / identify** — split GIF frames for inspection
54. **exiftool on video** — metadata/comments in containers
55. **binwalk on video** — appended/embedded payloads

## Text / whitespace / unicode

56. **snow** — whitespace steganography (trailing spaces/tabs) `darkside.com.au/snow`
57. **stegsnow** — snow variant `apt install stegsnow`
58. **zero-width unicode decoder** — reveal ZWSP/ZWNJ hidden text (330k.github.io / scripts)
59. **unicode-steganography (web)** — zero-width encode/decode
60. **whitespace language interpreter** — the esolang itself
61. **cat -A / show whitespace** — spot trailing-space payloads
62. **homoglyph detector** — spot look-alike-char substitution
63. **base-N in text** — decode blobs embedded in prose

## Files / containers / filesystem-level

64. **binwalk** — find embedded files `github.com/ReFirmLabs/binwalk`
65. **binwalk -e / --dd** — extract embedded data
66. **foremost** — header/footer carving `apt install foremost`
67. **scalpel** — configurable carving
68. **photorec** — recover many file types `apt install testdisk`
69. **steghide in polyglots** — files valid as two types at once
70. **mitra** — build/analyze polyglot files `github.com/corkami/mitra`
71. **PoC||GTFO polyglot refs** — polyglot technique corpus
72. **7z/unzip on images** — many "images" are also archives
73. **truepolyglot** — craft polyglots
74. **pdfdetach / pdf-parser** — hidden attachments/streams in PDFs

## Scripts (yours + glue)

75. **flaghunt.sh** — triage wrapper `99-Scripts/flaghunt.sh`
76. **LSB extract one-liner (PIL)** — pull LSBs per channel in Python
77. **channel-XOR script** — XOR two images / two channels
78. **bit-plane dumper** — save each of 8 bit planes as an image
79. **spectrogram batch** — sox over a folder of audio
80. **steghide wordlist loop** — try passphrases (or just use stegseek)

## Quick install (Debian/Kali/WSL)

```bash
sudo apt install -y steghide outguess pngcheck imagemagick sox audacity \
  zbar-tools binwalk foremost testdisk stegsnow multimon-ng minimodem \
  libimage-exiftool-perl
pip install stegoveritas stegpy stego-lsb
gem install zsteg
# stegseek, Stegsolve, mp3stego, AudioStego, stego-toolkit: from their GitHub releases
# fast path: just use aperisolve.com in a browser
```

## Workflow cheat (what to try, in order)

1. `file` + `exiftool` + `binwalk` + `strings` — the free wins.
2. PNG/BMP → **zsteg -a**; anything → **AperiSolve**.
3. JPEG → **steghide** (empty pass first), then **stegseek** + rockyou.
4. Image looks "normal" → **Stegsolve** bit planes / **StegoVeritas**.
5. Audio → **spectrogram** (sox/Audacity/Sonic Visualiser), then LSB, then DTMF.
6. Still nothing → look for **polyglot** / appended archive / zero-width text.

## Sources
- DominicBreuker/stego-toolkit; Zeecka/AperiSolve; cyberteach360/Steganography
- myctf.digitalpress.blog/steganography; 0xrick stego notes; linuxlinks AperiSolve

> Tags: #ctf #stego #forensics #catalog


---

# Stego Scripts (100)

Copy-paste snippets for steganography. 🟢 easy / 🟡 medium / 🔴 hard. Shell + Python. Needs: `pip install pillow numpy scipy pyzbar` and apt tools (zsteg via gem). Pairs with [[Stego Catalog]], [[Forensics]], [[Forensics Techniques]].

> Golden rule: ALWAYS do triage first (1–10). Most "stego" is just an appended zip or a metadata comment.

---

## Triage (do first, every time)

🟢 1. file type
```bash
file secret.*
```

🟢 2. metadata
```bash
exiftool secret.png
```

🟢 3. embedded files
```bash
binwalk secret.png
```

🟢 4. extract embedded
```bash
binwalk -e secret.png
```

🟢 5. strings + flag grep
```bash
strings -n 6 secret.png | grep -iE 'flag|ctf|\{'
```

🟢 6. carve everything (foremost)
```bash
foremost -i secret.png -o out/
```

🟢 7. all-in-one web (AperiSolve)
```bash
# upload at aperisolve.com → runs zsteg/steghide/outguess/exif/binwalk/foremost/strings
```

🟢 8. triage wrapper
```bash
bash 99-Scripts/flaghunt.sh secret.png
```

🟢 9. image properties
```bash
identify -verbose secret.png | head -40
```

🟡 10. data appended past PNG end
```python
d=open("secret.png","rb").read(); i=d.rfind(b"IEND")+8; print(len(d)-i,"extra bytes"); print(d[i:i+80])
```

---

## Image — LSB / bit planes

🟢 11. zsteg (PNG/BMP)
```bash
zsteg secret.png
```

🟢 12. zsteg all methods
```bash
zsteg -a secret.png
```

🟡 13. zsteg extract a specific channel/order
```bash
zsteg -E 'b1,rgb,lsb,xy' secret.png > out.bin
```

🟢 14. LSB of red channel (PIL)
```python
from PIL import Image
px=list(Image.open("secret.png").convert("RGB").getdata())
bits="".join(str(r&1) for r,g,b in px)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 15. LSB all channels, pick the one with "flag"
```python
from PIL import Image
im=Image.open("secret.png").convert("RGB"); w,h=im.size; p=im.load()
for ch in range(3):
    bits="".join(str(p[x,y][ch]&1) for y in range(h) for x in range(w))
    out=bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))
    if b"flag" in out or b"CTF" in out: print("channel",ch,out[:100])
```

🟡 16. LSB column-major order
```python
from PIL import Image
im=Image.open("secret.png").convert("RGB"); w,h=im.size; p=im.load()
bits="".join(str(p[x,y][0]&1) for x in range(w) for y in range(h))   # x outer
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 17. LSB RGB-interleaved (r,g,b,r,g,b…)
```python
from PIL import Image
px=list(Image.open("secret.png").convert("RGB").getdata())
bits="".join(str(v&1) for pix in px for v in pix)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 18. alpha-channel LSB
```python
from PIL import Image
px=list(Image.open("secret.png").convert("RGBA").getdata())
bits="".join(str(a&1) for r,g,b,a in px)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 19. LSB with numpy (fast)
```python
import numpy as np; from PIL import Image
a=np.array(Image.open("secret.png").convert("RGB"))
bits=(a[:,:,0]&1).flatten(); bs=np.packbits(bits).tobytes(); print(bs[:120])
```

🟡 20. 2 least-significant bits
```python
from PIL import Image
px=list(Image.open("secret.png").convert("RGB").getdata())
bits="".join(f"{r&3:02b}" for r,g,b in px)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 21. MSB extraction
```python
from PIL import Image
px=list(Image.open("secret.png").convert("RGB").getdata())
bits="".join(str((r>>7)&1) for r,g,b in px)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 22. reversed bit order per byte
```python
bits="...."  # from any LSB pull
rev="".join(bits[i:i+8][::-1] for i in range(0,len(bits),8))
print(bytes(int(rev[i:i+8],2) for i in range(0,len(rev)//8*8,8))[:120])
```

🟡 23. dump all 8 bit planes as images
```python
import numpy as np; from PIL import Image
a=np.array(Image.open("secret.png").convert("L"))
for b in range(8): Image.fromarray((((a>>b)&1)*255).astype('uint8')).save(f"plane{b}.png")
```

🟡 24. per-channel bit planes
```python
import numpy as np; from PIL import Image
a=np.array(Image.open("secret.png").convert("RGB"))
for c in range(3):
    for b in range(8): Image.fromarray((((a[:,:,c]>>b)&1)*255).astype('uint8')).save(f"c{c}_b{b}.png")
```

🟡 25. amplify LSB so it's visible
```python
import numpy as np; from PIL import Image
a=np.array(Image.open("secret.png").convert("RGB"))
Image.fromarray(((a&1)*255).astype('uint8')).save("amplified.png")
```

🟡 26. run stegoveritas (brute everything)
```bash
stegoveritas secret.png
```

🟡 27. LSB starting after a header offset
```python
from PIL import Image
px=list(Image.open("secret.png").convert("RGB").getdata())[100:]   # skip first 100 px
bits="".join(str(r&1) for r,g,b in px); print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 28. LSB until null terminator
```python
from PIL import Image
px=list(Image.open("secret.png").convert("RGB").getdata())
bits="".join(str(v&1) for pix in px for v in pix); out=bytearray()
for i in range(0,len(bits)//8*8,8):
    c=int(bits[i:i+8],2)
    if c==0: break
    out.append(c)
print(out)
```

🟡 29. BMP LSB (zsteg + manual)
```bash
zsteg -a secret.bmp
```

🔴 30. brute channel+bit+order combos
```python
from PIL import Image
im=Image.open("secret.png").convert("RGB"); w,h=im.size; p=im.load()
for ch in range(3):
  for order in ("xy","yx"):
    coords=[(x,y) for y in range(h) for x in range(w)] if order=="xy" else [(x,y) for x in range(w) for y in range(h)]
    bits="".join(str(p[x,y][ch]&1) for x,y in coords)
    out=bytes(int(bits[i:i+8],2) for i in range(0,min(len(bits),4000)//8*8,8))
    if b"flag" in out: print(ch,order,out[:80])
```

---

## Image — keyed extractors

🟢 31. steghide info
```bash
steghide info secret.jpg
```

🟢 32. steghide extract (empty pass)
```bash
steghide extract -sf secret.jpg -p ""
```

🟢 33. steghide extract with passphrase
```bash
steghide extract -sf secret.jpg -p "hunter2"
```

🟢 34. stegseek (fast brute)
```bash
stegseek secret.jpg rockyou.txt
```

🟡 35. stegseek seed/empty-pass check
```bash
stegseek --crack secret.jpg /usr/share/wordlists/rockyou.txt out.txt
```

🟢 36. outguess extract
```bash
outguess -r secret.jpg out.txt
```

🟡 37. openstego extract
```bash
openstego extract -sf secret.png -xf out.bin
```

🟡 38. jsteg reveal (JPEG)
```bash
jsteg reveal secret.jpg out.txt
```

🟡 39. stegpy decode
```bash
stegpy -x secret.png
```

🟡 40. stego-lsb decode
```bash
steglsb steglsb -r -i secret.png -o out.txt -n 2
```

🟡 41. cloacked-pixel (AES LSB)
```bash
python lsb.py extract secret.png out.txt "password"
```

🟡 42. steghide passphrase loop (if no stegseek)
```bash
while read p; do steghide extract -sf s.jpg -p "$p" -xf out 2>/dev/null && echo "PASS:$p" && break; done < rockyou.txt
```

---

## Image — visual / transforms

🟡 43. Stegsolve (GUI bit planes/filters)
```bash
java -jar stegsolve.jar
```

🟡 44. error-level analysis (ELA)
```python
from PIL import Image, ImageChops, ImageEnhance
im=Image.open("x.jpg").convert("RGB"); im.save("_t.jpg",quality=90)
ela=ImageChops.difference(im,Image.open("_t.jpg"))
ImageEnhance.Brightness(ela).enhance(20).save("ela.png")
```

🟡 45. XOR two near-identical images
```python
from PIL import Image, ImageChops
ImageChops.difference(Image.open("a.png").convert("RGB"),Image.open("b.png").convert("RGB")).save("diff.png")
```

🟡 46. pixel diff (ImageMagick)
```bash
compare a.png b.png -compose src diff.png
```

🟡 47. brightness/contrast stretch (reveal faint)
```bash
convert secret.png -auto-level -contrast-stretch 0 out.png
```

🟡 48. invert colors
```bash
convert secret.png -negate inv.png
```

🟡 49. separate channels (ImageMagick)
```bash
convert secret.png -channel R -separate r.png
```

🟡 50. dump palette (indexed PNG)
```bash
convert secret.png -unique-colors txt:- | head
```

🟡 51. threshold to black/white
```bash
convert secret.png -threshold 50% bw.png
```

🟡 52. extract embedded thumbnail
```bash
exiftool -b -ThumbnailImage secret.jpg > thumb.jpg
```

🟡 53. GIF frame split
```bash
convert anim.gif -coalesce frame_%03d.png
```

🟡 54. diff consecutive GIF frames
```python
from PIL import Image, ImageChops, ImageSequence
fr=list(ImageSequence.Iterator(Image.open("anim.gif")))
for i in range(1,len(fr)): ImageChops.difference(fr[i-1].convert("RGB"),fr[i].convert("RGB")).save(f"d{i}.png")
```

🟡 55. stack all frames (XOR/average)
```python
import numpy as np; from PIL import Image, ImageSequence
acc=None
for f in ImageSequence.Iterator(Image.open("anim.gif")):
    a=np.array(f.convert("L")).astype(int); acc=a if acc is None else acc^a
Image.fromarray(acc.astype('uint8')).save("xor_all.png")
```

---

## Metadata / file structure

🟢 56. all metadata
```bash
exiftool -a -u -g1 secret.jpg
```

🟢 57. specific comment field
```bash
exiftool -Comment -UserComment secret.jpg
```

🟢 58. GPS coordinates
```bash
exiftool -gps:all secret.jpg
```

🟡 59. PNG chunk listing
```bash
pngcheck -v secret.png
```

🟡 60. extract tEXt/zTXt chunks
```python
from PIL import Image; im=Image.open("secret.png"); print(im.text)   # dict of text chunks
```

🟡 61. JPEG comment (COM) segment
```bash
exiftool -Comment secret.jpg ; strings secret.jpg | grep -i flag
```

🟡 62. dump a named binary tag
```bash
exiftool -b -ICC_Profile secret.jpg > icc.bin
```

🟡 63. find all exif that looks like flag
```bash
exiftool secret.jpg | grep -iE 'flag|ctf|\{'
```

🟡 64. raw APPn segment carve
```python
d=open("secret.jpg","rb").read(); i=d.find(b"\xff\xe1"); print(d[i:i+80])
```

🟡 65. strip vs compare (removed-metadata diff)
```bash
exiftool -all= -o clean.jpg secret.jpg ; ls -l secret.jpg clean.jpg
```

---

## Polyglots / appended data

🟡 66. image is also a zip
```bash
unzip secret.png
```

🟡 67. binwalk extract with dd offset
```bash
binwalk --dd='.*' secret.png
```

🟡 68. find zip signature + carve
```python
d=open("secret.png","rb").read(); i=d.find(b"PK\x03\x04"); open("hidden.zip","wb").write(d[i:])
```

🟡 69. find any trailing archive (gzip/rar/7z)
```python
d=open("f","rb").read()
for sig,ext in [(b"PK\x03\x04","zip"),(b"Rar!","rar"),(b"7z\xbc\xaf","7z"),(b"\x1f\x8b","gz")]:
    i=d.find(sig)
    if i>0: print(ext,"at",i)
```

🟡 70. split concatenated files
```bash
binwalk -e f ; ls -la _f.extracted/
```

🟡 71. build/analyze polyglot (mitra)
```bash
python mitra.py a.png b.zip      # github.com/corkami/mitra
```

🟡 72. check multiple magic at once
```bash
for off in 0 100 512; do xxd -s $off -l 4 secret.png; done
```

---

## Audio

🟢 73. spectrogram (sox)
```bash
sox audio.wav -n spectrogram -o spec.png
```

🟡 74. spectrogram with full range
```bash
sox audio.wav -n spectrogram -x 2000 -y 1025 -o spec.png
```

🟡 75. spectrogram in python (scipy)
```python
from scipy.io import wavfile; import numpy as np, matplotlib.pyplot as plt
r,d=wavfile.read("audio.wav"); d=d[:,0] if d.ndim>1 else d
plt.specgram(d,Fs=r); plt.savefig("spec.png")
```

🟡 76. WAV LSB extract
```python
import wave; w=wave.open("a.wav","rb"); f=w.readframes(w.getnframes())
bits="".join(str(b&1) for b in f)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8))[:120])
```

🟡 77. split stereo channels
```bash
sox a.wav L.wav remix 1 ; sox a.wav R.wav remix 2
```

🟡 78. channel difference (hidden in one side)
```python
from scipy.io import wavfile; import numpy as np
r,d=wavfile.read("a.wav"); diff=d[:,0].astype(int)-d[:,1].astype(int); print(diff[:50])
```

🟡 79. DTMF decode
```bash
multimon-ng -a DTMF -t wav a.wav
```

🟡 80. FSK / modem decode
```bash
minimodem --rx 1200 < a.wav
```

🟡 81. slow playback (reveal speech)
```bash
sox a.wav slow.wav speed 0.5
```

🟡 82. reverse audio
```bash
sox a.wav rev.wav reverse
```

🟡 83. mp3stego extract
```bash
Decode.exe -X -P pass secret.mp3     # Windows; needs WAV path
```

🟡 84. AudioStego (hideme) recover
```bash
hideme secret.mp3 -f           # github.com/danielcardeenas/AudioStego
```

🟡 85. SSTV decode (image in audio)
```bash
# play audio into QSSTV, or use sstv-decoder; choose mode (Robot/Martin/Scottie)
```

---

## Text / whitespace / unicode

🟡 86. snow (whitespace) extract
```bash
stegsnow -C secret.txt
```

🟡 87. zero-width unicode → bits
```python
t=open("msg.txt",encoding="utf-8").read()
bits="".join("1" if c=="​" else "0" if c=="‌" else "" for c in t)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8)))
```

🟡 88. reveal trailing whitespace
```bash
cat -A secret.txt | grep -E ' +\$'
```

🟡 89. unicode tag-char (U+E00xx) decode
```python
t=open("msg.txt",encoding="utf-8").read()
print("".join(chr(ord(c)-0xE0000) for c in t if 0xE0000<=ord(c)<=0xE007F))
```

🟡 90. homoglyph / non-ascii spotter
```python
t=open("msg.txt",encoding="utf-8").read()
print([(i,c,hex(ord(c))) for i,c in enumerate(t) if ord(c)>127])
```

🟡 91. whitespace-lang (tabs/spaces program)
```bash
# interpret with a Whitespace interpreter if file is only space/tab/newline
```

🟡 92. base-N blob hidden in prose
```python
import re,base64; m=re.search(r'[A-Za-z0-9+/=]{20,}', open("t.txt").read()); print(base64.b64decode(m.group()))
```

🟡 93. first-letter / acrostic
```python
print("".join(line[0] for line in open("poem.txt") if line.strip()))
```

🟡 94. bit-per-capital-letter (Bacon-ish)
```python
t=open("t.txt").read(); bits="".join("1" if c.isupper() else "0" for c in t if c.isalpha())
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8)))
```

---

## QR / barcode

🟡 95. zbarimg decode
```bash
zbarimg qr.png
```

🟡 96. pyzbar decode
```python
from pyzbar.pyzbar import decode; from PIL import Image; print(decode(Image.open("qr.png")))
```

🔴 97. rebuild QR from extracted bits
```python
# write bits to a black/white grid image, upscale, then zbarimg it
import numpy as np; from PIL import Image
g=np.array([[0,1],[1,0]])  # your recovered module grid
Image.fromarray((1-g).astype('uint8')*255).resize((300,300),Image.NEAREST).save("rebuilt.png")
```

---

## Automation / flow

🟡 98. batch zsteg over a folder
```bash
for f in *.png *.bmp; do echo "== $f"; zsteg -a "$f" 2>/dev/null | grep -iE 'flag|text|zlib'; done
```

🟡 99. grep flag across extracted tree
```bash
grep -rniaE 'flag\{|ctf\{|dctf\{' . 2>/dev/null
```

🟢 100. the universal stego flow
```bash
# 1 file/exiftool/binwalk/strings  2 PNG/BMP→zsteg -a ; anything→AperiSolve
# 3 JPEG→steghide(empty pass)→stegseek  4 looks normal→Stegsolve/stegoveritas
# 5 audio→spectrogram→LSB→DTMF  6 still nothing→polyglot/appended/zero-width text
```

---
> Set wordlist paths to your own copy of rockyou.txt. Some tools (mp3stego/AudioStego/Stegsolve) are Windows/Java — install from their repos. When in doubt, AperiSolve (web) runs most of these at once.
> Tags: #ctf #stego #scripts


---

## 🗺️ DCTF Roadmap — Steganography

> Stego is a **Jeopardy quals** sub-category of forensics. It punishes guessing and rewards a fixed checklist. See [[Forensics-Complete]] and [[Roadmap]].

### What to study (in order)
1. **Triage** — `file`/`exiftool`/`binwalk`/`strings`; appended data past EOF. (section triage above)
2. **Image LSB** — zsteg (PNG/BMP), channel/bit/order variants, bit-plane viewing. (scripts 11–30)
3. **Keyed image** — steghide (empty pass first), stegseek + rockyou, outguess. (scripts 31–42)
4. **Visual** — Stegsolve bit planes, ELA, image diff/XOR, GIF frame stacking. (scripts 43–55)
5. **Audio** — spectrogram, WAV LSB, DTMF/FSK/SSTV. (scripts 73–85)
6. **Text + polyglots** — zero-width unicode, whitespace, appended zip, polyglot files. (scripts 66–72, 86–94)

### How to study
- **Memorize the order of operations** (below) — stego is lost by trying things randomly.
- Create your own stego of each type, then recover it — you learn the artifacts to look for.
- When a tool finds nothing, that's information: it rules out a whole class; move down the checklist.

### How to exercise
- **picoCTF** + **aperisolve.com** on practice images; **stego-toolkit** (Docker) to have everything ready.
- Drill: for a given image, run the full checklist in <10 min and state where (if anywhere) data hides.

### How to do things (the fixed checklist)
1. `file` + `exiftool` + `binwalk` + `strings` — the free wins. (scripts 1–8)
2. PNG/BMP → **zsteg -a**; anything → **AperiSolve** (runs most tools at once). (scripts 11–12)
3. JPEG → **steghide** (empty passphrase first) → **stegseek** + rockyou. (scripts 32–34)
4. Looks "normal" → **Stegsolve** bit planes / **StegoVeritas** / per-channel LSB in Python. (scripts 14–30, 43)
5. Audio → **spectrogram** → LSB → DTMF/FSK. (scripts 73–80)
6. Still nothing → **polyglot / appended archive / zero-width text**. (scripts 66–72, 87–89)
7. `grep` the extracted output for the flag.



> Tags: #ctf #stego #roadmap #dctf

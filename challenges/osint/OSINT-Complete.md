# OSINT — Complete (Techniques + Catalog + Scripts + Roadmap)

All-in-one for **CTF OSINT**: finding planted clues, geolocation puzzles, infrastructure recon, metadata, and public records. Pairs with [[Home]].


> Mental model: OSINT CTF = a scavenger hunt. The flag is reachable from public breadcrumbs — a username, a photo's background, a DNS record, an old cached page. Pivot from one breadcrumb to the next.

---

# TECHNIQUES (100, explained + scenarios)

## A. Scoping & methodology

1. **read the prompt for a pivot** — the given handle/photo/domain is the seed. *Scenario:* a challenge gives a username; everything starts there.
2. **pick the entity type** — person-handle / domain / image / IP. *Scenario:* knowing it's an "image geolocation" tells you which toolset to open.
3. **pivot chaining** — each find feeds the next lookup. *Scenario:* username → profile → bio link → domain → WHOIS email → flag.
4. **timeline the target** — order events/posts. *Scenario:* the flag is "what city were they in on date X" — build the timeline.
5. **keep a link graph** — map breadcrumbs as you go. *Scenario:* a Maltego-style graph shows the one connection you missed.
6. **note everything verbatim** — exact strings, IDs, timestamps. *Scenario:* a post ID later becomes the lookup key.
7. **stop at scope** — public data, challenge target only. *Scenario:* a lead points at a real private person → that's not the intended path; re-read the prompt.
8. **verify, don't assume** — confirm a find from a second source. *Scenario:* two independent sources agree the photo is in Lisbon.

## B. Search engines & dorking

9. **exact-phrase search** — quotes force literal. *Scenario:* `"flag{"` or a unique bio phrase finds the one page.
10. **site: operator** — restrict to a domain. *Scenario:* `site:pastebin.com target` surfaces a leak.
11. **filetype: operator** — find documents. *Scenario:* `filetype:pdf site:target.com` finds a metadata-rich doc.
12. **inurl/intitle** — match URL/title. *Scenario:* `intitle:"index of"` exposes an open directory.
13. **Google Hacking DB (GHDB)** — prebuilt dorks. *Scenario:* an exposed-config dork finds the planted file.
14. **cache: / Wayback** — old versions. *Scenario:* the flag was removed; the cached page still has it.
15. **multi-engine** — Bing/DuckDuckGo/Yandex differ. *Scenario:* Yandex indexes a page Google dropped.
16. **boolean & minus** — include/exclude terms. *Scenario:* `target -site:twitter.com` cuts noise.
17. **date-range search** — scope by time. *Scenario:* limit to the challenge's year to find the right post.
18. **reverse-link / mentions** — who references the target. *Scenario:* a forum mention links the handle to a real domain.

## C. Username / account discovery (public)

19. **username enumeration** — same handle across sites. *Scenario:* sherlock/maigret finds the handle on 10 platforms; one bio has the flag.
20. **handle variations** — dots/underscores/numbers. *Scenario:* `john.doe` vs `johndoe_` is the real account.
21. **profile-pivot** — bio links, pinned posts. *Scenario:* a linktree in the bio leads to a personal site.
22. **cross-reference avatars** — same picture reused. *Scenario:* reverse-image the avatar to find other accounts.
23. **public post history** — archived timelines. *Scenario:* an old post names the hometown the flag asks for.
24. **follower/following patterns (public)** — obvious links. *Scenario:* a public "friends" list names an alt account.
25. **platform IDs** — numeric user IDs persist. *Scenario:* a renamed account is still findable by its numeric ID.
26. **join date / first post** — account age. *Scenario:* the challenge asks for the account's creation year.
27. **public gists/repos by user** — code footprint. *Scenario:* a user's gist contains the flag in a comment.
28. **forum/Q&A footprint** — StackOverflow/Reddit. *Scenario:* a question they posted leaks infra details.

## D. Email & breach (public, hashed where possible)

29. **email format guessing** — `first.last@domain`. *Scenario:* the org's pattern yields the challenge mailbox.
30. **email → accounts (holehe)** — where an email is registered (no login). *Scenario:* confirms the email ties to a specific platform.
31. **breach-presence check (HIBP)** — is it in known breaches. *Scenario:* the challenge asks which breach exposed the address.
32. **gravatar lookup** — avatar from email hash. *Scenario:* the MD5-of-email Gravatar reveals a profile photo.
33. **MX / mail provider** — who hosts the mail. *Scenario:* identifies the provider the flag references.
34. **public mailing-list archives** — posts by address. *Scenario:* a dev-list post leaks a server name.
35. **verify via headers** — email header metadata (given a sample). *Scenario:* `Received:` chain reveals the origin IP.

## E. Domain / DNS / WHOIS / certificates

36. **WHOIS lookup** — registrant/dates/registrar. *Scenario:* a registrant email is the pivot to a person.
37. **historical WHOIS** — pre-privacy records. *Scenario:* an old WHOIS still shows the real owner.
38. **DNS records (A/MX/TXT/NS)** — infra map. *Scenario:* a TXT record literally contains the flag.
39. **subdomain enumeration** — hidden hosts. *Scenario:* `dev.target.com` serves the staging app.
40. **certificate transparency (crt.sh)** — certs reveal subdomains. *Scenario:* CT logs list a subdomain not in DNS brute lists.
41. **passive DNS** — historical resolutions. *Scenario:* a domain used to point at an IP that still hosts the clue.
42. **reverse DNS / PTR** — IP→name. *Scenario:* PTR reveals the hostname pattern.
43. **zone transfer (AXFR)** — misconfigured NS. *Scenario:* an open AXFR dumps every record at once.
44. **DNS brute (wordlist)** — guess names. *Scenario:* `vpn.target.com` appears via brute force.
45. **registrar / nameserver pivot** — shared infra. *Scenario:* same NS links two seemingly separate domains.
46. **favicon hash pivot** — identify same app elsewhere. *Scenario:* a favicon hash on Shodan finds the same site on another IP.
47. **ASN / netblock** — who owns the IP range. *Scenario:* the ASN ties the host to an organization.
48. **domain age / history** — Wayback first-seen. *Scenario:* the challenge asks the site's launch year.

## F. IP / infrastructure

49. **Shodan host lookup** — open ports/banners. *Scenario:* a banner on the target IP contains the flag.
50. **Censys lookup** — certs/services. *Scenario:* a cert's SAN lists a hidden hostname.
51. **geo-IP** — approximate location. *Scenario:* the challenge wants the hosting country.
52. **banner/service fingerprint** — what runs there. *Scenario:* a server header reveals the stack.
53. **favicon / title search (Shodan)** — find clones. *Scenario:* `http.title:"Secret Portal"` finds the box.
54. **TLS cert pivot** — shared cert across hosts. *Scenario:* the same cert fingerprint appears on the real server.
55. **open-directory / exposed service** — misconfig. *Scenario:* an open S3-style listing holds the flag file.
56. **threat-intel lookups** — VirusTotal/urlscan relations. *Scenario:* urlscan shows the domain's resources + a hidden endpoint.
57. **IP history** — what the IP hosted before. *Scenario:* passive data shows the prior domain with the clue.

## G. Image OSINT

58. **EXIF/metadata read** — camera, time, GPS. *Scenario:* the photo's GPS tag is literally the answer.
59. **GPS → place** — map the coordinates. *Scenario:* lat/long drops you on the exact building.
60. **reverse image search** — find the source. *Scenario:* the stock photo's origin page names the location.
61. **multi-engine reverse search** — Yandex excels at faces/places. *Scenario:* Yandex matches the landmark Google missed.
62. **crop & re-search** — isolate a detail. *Scenario:* cropping to a sign gets a better match.
63. **thumbnail/embedded preview** — original inside metadata. *Scenario:* the EXIF thumbnail is the uncropped image.
64. **error-level analysis** — edited regions. *Scenario:* ELA shows a pasted-in element that's the clue.
65. **image hash / dedupe** — find reuse. *Scenario:* perceptual hash links the avatar to another account.
66. **software/device fingerprint** — editor/camera tags. *Scenario:* "Adobe" + device model narrows the source.

## H. Geolocation (visual clues)

67. **landmark identification** — recognizable structures. *Scenario:* a tower in the background pins the city.
68. **signage / language** — text + script. *Scenario:* Portuguese signage → Portugal/Brazil; narrow further.
69. **license-plate region** — plate format/colors. *Scenario:* an EU yellow plate narrows the country.
70. **road markings / driving side** — left vs right. *Scenario:* left-side driving rules out many countries.
71. **vegetation / climate** — flora hints. *Scenario:* palm species suggests a Mediterranean coast.
72. **sun position / shadows** — direction + time. *Scenario:* shadow angle + timestamp gives rough latitude/bearing.
73. **architecture style** — regional building types. *Scenario:* Nordic facades point to Scandinavia.
74. **utility/bollard/sign standards** — country-specific gear. *Scenario:* a specific bollard style = a specific country (GeoGuessr lore).
75. **street-view matching** — walk the area. *Scenario:* match the storefront in Street View to confirm the address.
76. **map feature search (Overpass)** — query OSM for features. *Scenario:* find "the only church next to a lake" in a region.
77. **what3words / coordinates formats** — convert given hints. *Scenario:* a what3words address resolves to the spot.
78. **business/POI lookup** — a named shop in-frame. *Scenario:* the café's name geocodes to one address.

## I. Social media (public)

79. **public post search** — keyword across a platform. *Scenario:* snscrape finds the post holding the flag.
80. **geotagged posts** — location-tagged content. *Scenario:* a tagged photo gives the venue.
81. **public media download** — profile images/videos. *Scenario:* a video's background is the geolocation target.
82. **bio / link analysis** — external links. *Scenario:* the bio URL is the next hop.
83. **hashtag/event correlation** — tie posts to an event. *Scenario:* an event hashtag dates and places the photo.
84. **archived/deleted posts** — caches/archives. *Scenario:* a deleted tweet survives on archive.today.
85. **replies/mentions graph** — who they talk to. *Scenario:* a public reply names the alt account.

## J. Documents & metadata

86. **document metadata (exiftool)** — author/software/dates. *Scenario:* a PDF's Author field is a real username.
87. **Office/PDF producer tags** — tool + sometimes paths. *Scenario:* a template path leaks an internal username.
88. **metadata harvesting (metagoofil/FOCA)** — bulk doc metadata. *Scenario:* harvesting a site's PDFs reveals usernames/emails.
89. **revision/track-changes** — hidden edits. *Scenario:* an unaccepted change in a DOCX holds the flag.
90. **embedded objects** — fonts/images with their own metadata. *Scenario:* an embedded image's EXIF is the clue.

## K. Archives / history / caches

91. **Wayback Machine** — historical snapshots. *Scenario:* the flag was on the homepage in 2019; Wayback has it.
92. **archive.today** — on-demand snapshots. *Scenario:* a deleted page was archived here.
93. **cached search results** — engine caches. *Scenario:* the live page 404s; the cache still renders.
94. **CommonCrawl / web archives** — bulk corpora. *Scenario:* a URL only survives in a crawl dump.
95. **paste-site history** — Pastebin & mirrors. *Scenario:* a paste linked from an old post holds credentials.

## L. Code / repo leaks (public)

96. **GitHub search / dorking** — code/secrets. *Scenario:* a committed `.env` in a public repo has the flag.
97. **git history mining** — removed-but-committed. *Scenario:* `git log -p` shows a deleted secret still in history.
98. **secret scanning (gitleaks/trufflehog)** — keys in repos. *Scenario:* a scanner finds the planted token.
99. **public CI/issue/wiki leaks** — build logs, issues. *Scenario:* a CI log prints an env var with the flag.

## M. Flow
100. **the OSINT flow** — seed → pivot → verify → pivot → flag; log every step. *Scenario:* disciplined pivoting beats random googling every time.

---

# CATALOG (100 tools)

## Frameworks / aggregators
1. **Maltego** — link-analysis graphs `maltego.com`
2. **SpiderFoot** — automated OSINT engine `github.com/smicallef/spiderfoot`
3. **recon-ng** — modular recon framework
4. **theHarvester** — emails/subdomains/hosts `github.com/laramies/theHarvester`
5. **OSINT Framework** — directory of tools `osintframework.com`
6. **IntelTechniques tools** — Michael Bazzell's toolset
7. **Hunchly** — capture/organize web OSINT
8. **Creepy** — geolocation aggregation (public)
9. **Mitaka** — browser extension for IOC pivots
10. **Datasploit / FinalRecon** — recon automation

## Search / dorking
11. **Google dorks (GHDB)** — `exploit-db.com/google-hacking-database`
12. **DuckDuckGo / Bing / Yandex** — multi-engine
13. **Photon** — fast web crawler `github.com/s0md3v/Photon`
14. **grep.app** — search public code
15. **publicwww** — search page source/markup
16. **Intelligence X** — search leaks/darkweb indexes (public)
17. **Startpage / Searx** — privacy meta-search
18. **Google Programmable Search** — scoped custom engine

## Username / people (public)
19. **Sherlock** — username across sites `github.com/sherlock-project/sherlock`
20. **Maigret** — username + profile parsing `github.com/soxoj/maigret`
21. **WhatsMyName** — username enumeration project
22. **social-analyzer** — profile discovery `github.com/qeeqbox/social-analyzer`
23. **Blackbird** — fast username search
24. **snscrape** — scrape public social posts `pip install snscrape`
25. **Instaloader** — public Instagram media `pip install instaloader`
26. **gallery-dl** — bulk public media download
27. **twint (legacy)** — twitter scraping (often broken)
28. **nitter instances** — alt twitter frontends

## Email / breach
29. **holehe** — email → registered sites (no login) `github.com/megadose/holehe`
30. **h8mail** — email breach lookup `github.com/khast3x/h8mail`
31. **Have I Been Pwned** — breach presence `haveibeenpwned.com`
32. **Hunter.io** — company email patterns
33. **Epieos** — email/phone → accounts (public)
34. **GHunt** — public Google account info `github.com/mxrch/GHunt`
35. **Gravatar** — avatar from email hash
36. **phoneinfoga** — phone number recon `github.com/sundowndev/phoneinfoga`

## Domain / DNS / cert
37. **whois** — registration data `apt install whois`
38. **dig / nslookup / host** — DNS `apt install dnsutils`
39. **Amass** — subdomain enum `github.com/owasp-amass/amass`
40. **subfinder** — passive subdomains `github.com/projectdiscovery/subfinder`
41. **assetfinder** — find domains/subs
42. **Sublist3r** — subdomain enum
43. **dnsrecon / dnsenum** — DNS recon
44. **dnsdumpster** — passive DNS (web)
45. **crt.sh** — certificate transparency `crt.sh`
46. **censys (certs)** — cert/service search
47. **SecurityTrails** — DNS/WHOIS history
48. **ViewDNS.info** — WHOIS/DNS tools
49. **whoisxmlapi / whoisfreaks** — WHOIS history APIs
50. **dnstwist** — typosquat/variant domains

## IP / infrastructure
51. **Shodan** — internet device search `shodan.io`
52. **Censys** — host/cert search `censys.io`
53. **Netlas** — asset discovery
54. **ZoomEye / FOFA / Quake** — Shodan-like engines
55. **urlscan.io** — scan + resources of a URL
56. **VirusTotal** — file/URL/domain relations
57. **GreyNoise** — internet background noise context
58. **AbuseIPDB** — IP reputation
59. **BGP.he.net / bgpview** — ASN/netblock lookup
60. **ipinfo.io / ip-api** — geo/ASN of an IP

## Image / geolocation
61. **ExifTool** — metadata read `apt install libimage-exiftool-perl`
62. **Jimpl / exif.tools** — web EXIF viewers
63. **Google Images** — reverse image search
64. **Yandex Images** — best for faces/places
65. **TinEye** — reverse image (oldest indexes)
66. **Bing Visual Search** — reverse image
67. **GeoSpy / Picarta** — AI image geolocation (verify results)
68. **Google Earth / Maps / Street View** — ground truth
69. **OpenStreetMap** — open map data
70. **Overpass Turbo** — query OSM features `overpass-turbo.eu`
71. **Nominatim** — OSM geocoding
72. **SunCalc** — sun position by time/place `suncalc.org`
73. **what3words** — 3-word → coordinates
74. **Mapillary / KartaView** — crowd street imagery
75. **GeoHints / GeoGuessr meta refs** — regional clue guides

## Transport / signals (public trackers)
76. **FlightRadar24 / ADS-B Exchange** — flights
77. **MarineTraffic / VesselFinder** — ships
78. **Wigle.net** — wifi SSID geolocation
79. **OpenSky Network** — flight data API
80. **RailMapOnline** — rail lines

## Documents / metadata
81. **metagoofil** — harvest doc metadata
82. **FOCA** — document metadata (Windows)
83. **pdfinfo / exiftool on PDF** — PDF metadata
84. **oletools** — Office metadata/macros
85. **strings / binwalk** — leftover metadata in files

## Archives / caches
86. **Wayback Machine** — `web.archive.org`
87. **archive.today** — on-demand snapshots
88. **CachedView** — multi-cache viewer
89. **CommonCrawl** — bulk web corpus
90. **Pastebin + psbdmp** — paste search/history

## Code / leaks
91. **GitHub code search** — `github.com/search`
92. **gitleaks** — secrets in repos `github.com/gitleaks/gitleaks`
93. **trufflehog** — secret scanning `github.com/trufflesecurity/trufflehog`
94. **git-dumper** — reconstruct exposed .git `github.com/arthaud/git-dumper`
95. **GitHound / shhgit** — GitHub secret hunting

## Phone / misc / verify
96. **phoneinfoga** — phone recon (dup, key tool)
97. **Numverify / libphonenumber** — number validation
98. **Mitaka / IOC pivots** — quick pivots
99. **Checkmate / OSINT bookmarklets** — browser helpers
100. **your flaghunt.sh + exiftool** — local triage of any downloaded artifact

## Install quick ref
```bash
pip install sherlock-project maigret holehe h8mail snscrape instaloader \
  phoneinfoga theHarvester spiderfoot shodan
sudo apt install -y whois dnsutils libimage-exiftool-perl amass
# subfinder/assetfinder/gitleaks/trufflehog: install from their GitHub releases (Go)
```

---

# SCRIPTS (100 snippets)

Public-data only; respect ToS/law. 🟢 easy / 🟡 medium / 🔴 hard. Many are tool invocations; python ones hit public APIs.

## Search / dorking
🟢 1. exact phrase + site dork
```
"unique phrase" site:github.com
```
🟢 2. exposed files dork
```
site:target.com filetype:pdf
```
🟢 3. open directory dork
```
intitle:"index of" "backup"
```
🟢 4. find config leaks
```
site:target.com ext:env OR ext:cfg OR ext:ini
```
🟢 5. paste-site dork
```
site:pastebin.com target_keyword
```
🟡 6. multi-engine quick open (shell)
```bash
for e in "google.com/search?q=" "bing.com/search?q=" "yandex.com/search/?text="; do echo "https://$e\"$1\""; done
```

## Username / accounts (public)
🟢 7. sherlock
```bash
sherlock targetuser
```
🟢 8. maigret (with report)
```bash
maigret targetuser --html
```
🟡 9. check a handle on one site (python)
```python
import requests
u="targetuser"
r=requests.get(f"https://api.github.com/users/{u}")
print(r.status_code, r.json().get("blog"), r.json().get("bio"))
```
🟡 10. GitHub user's public gists
```bash
curl -s "https://api.github.com/users/targetuser/gists" | grep -oE '"raw_url":[^,]+'
```
🟡 11. snscrape public posts
```bash
snscrape --max-results 50 twitter-user targetuser > posts.txt
```
🟡 12. instaloader public profile (metadata only)
```bash
instaloader --no-pictures --no-videos profile targetuser
```

## Email / breach (public)
🟢 13. holehe (email → sites)
```bash
holehe target@example.com
```
🟢 14. h8mail breach check
```bash
h8mail -t target@example.com
```
🟡 15. gravatar from email
```python
import hashlib
e="target@example.com".strip().lower()
print("https://gravatar.com/avatar/"+hashlib.md5(e.encode()).hexdigest()+"?d=404")
```
🟡 16. HIBP (needs API key)
```bash
curl -s -H "hibp-api-key: $KEY" "https://haveibeenpwned.com/api/v3/breachedaccount/target@example.com"
```
🟡 17. email format from name
```python
n=("john","doe"); d="example.com"
print([f"{n[0]}.{n[1]}@{d}", f"{n[0]}{n[1]}@{d}", f"{n[0][0]}{n[1]}@{d}"])
```

## Domain / DNS / WHOIS / certs
🟢 18. whois
```bash
whois example.com
```
🟢 19. all common DNS records
```bash
for t in A AAAA MX NS TXT SOA CNAME; do echo "== $t"; dig +short $t example.com; done
```
🟢 20. subfinder
```bash
subfinder -d example.com -silent
```
🟢 21. amass passive
```bash
amass enum -passive -d example.com
```
🟡 22. crt.sh subdomains (python)
```python
import requests
r=requests.get("https://crt.sh/?q=%25.example.com&output=json").json()
print(sorted({e["name_value"] for e in r}))
```
🟡 23. crt.sh one-liner
```bash
curl -s "https://crt.sh/?q=%25.example.com&output=json" | tr ',' '\n' | grep name_value | cut -d'"' -f4 | sort -u
```
🟡 24. zone transfer attempt
```bash
dig AXFR example.com @ns1.example.com
```
🟡 25. DNS brute (wordlist)
```bash
for s in www dev staging vpn admin api mail; do dig +short $s.example.com; done
```
🟡 26. reverse DNS (PTR)
```bash
dig +short -x 93.184.216.34
```
🟡 27. favicon hash (Shodan-style, python)
```python
import requests,codecs,mmh3
d=requests.get("https://example.com/favicon.ico").content
print(mmh3.hash(codecs.encode(d,"base64")))
```
🟡 28. passive DNS / history (SecurityTrails API)
```bash
curl -s -H "APIKEY: $KEY" "https://api.securitytrails.com/v1/domain/example.com/subdomains"
```

## IP / infra
🟢 29. geo + ASN of an IP
```bash
curl -s "https://ipinfo.io/93.184.216.34/json"
```
🟡 30. Shodan host (CLI)
```bash
shodan host 93.184.216.34
```
🟡 31. Shodan search by title
```bash
shodan search 'http.title:"Secret Portal"' --fields ip_str,port
```
🟡 32. Shodan API (python)
```python
import shodan; api=shodan.Shodan("KEY"); print(api.host("93.184.216.34")["data"][0]["data"][:200])
```
🟡 33. urlscan submit + result
```bash
curl -s -X POST https://urlscan.io/api/v1/scan/ -H "API-Key: $KEY" -d '{"url":"https://example.com"}'
```
🟡 34. VirusTotal domain relations
```bash
curl -s -H "x-apikey: $KEY" "https://www.virustotal.com/api/v3/domains/example.com"
```
🟡 35. ASN → prefixes
```bash
curl -s "https://api.bgpview.io/asn/15169/prefixes" | python3 -m json.tool | grep prefix | head
```

## Image / metadata
🟢 36. EXIF read
```bash
exiftool photo.jpg
```
🟢 37. GPS only
```bash
exiftool -gps:all -c "%.6f" photo.jpg
```
🟡 38. GPS → Google Maps link
```python
import subprocess,re
o=subprocess.check_output(["exiftool","-c","%.6f","-GPSLatitude","-GPSLongitude","photo.jpg"]).decode()
print(o); print("https://maps.google.com/?q=<lat>,<lon>")
```
🟡 39. extract embedded thumbnail
```bash
exiftool -b -ThumbnailImage photo.jpg > thumb.jpg
```
🟡 40. strip-less metadata dump (all tags)
```bash
exiftool -a -u -g1 photo.jpg
```
🟡 41. perceptual hash (dedupe)
```python
from PIL import Image; import imagehash
print(imagehash.phash(Image.open("photo.jpg")))
```
🟡 42. reverse image (open engines)
```python
print("Yandex: https://yandex.com/images/search?rpt=imageview&url=IMG_URL")
print("Google: https://lens.google.com/uploadbyurl?url=IMG_URL")
```
🟡 43. error-level analysis (ELA)
```python
from PIL import Image, ImageChops, ImageEnhance
im=Image.open("p.jpg").convert("RGB"); im.save("_t.jpg",quality=90)
ImageEnhance.Brightness(ImageChops.difference(im,Image.open("_t.jpg"))).enhance(20).save("ela.png")
```

## Geolocation helpers
🟡 44. sun position for lat/long/time
```python
# pip install astral
from astral.sun import sun; from astral import LocationInfo
import datetime
li=LocationInfo(latitude=38.72, longitude=-9.14)
print(sun(li.observer, date=datetime.date(2026,6,21)))
```
🟡 45. reverse geocode (Nominatim)
```bash
curl -s "https://nominatim.openstreetmap.org/reverse?lat=38.72&lon=-9.14&format=json" -H "User-Agent: ctf"
```
🟡 46. forward geocode a place
```bash
curl -s "https://nominatim.openstreetmap.org/search?q=Belem+Tower&format=json" -H "User-Agent: ctf" | head
```
🟡 47. Overpass: find a feature near a point
```
[out:json];(node["amenity"="place_of_worship"](around:500,38.69,-9.21););out;
```
🟡 48. what3words → coords (API)
```bash
curl -s "https://api.what3words.com/v3/convert-to-coordinates?words=filled.count.soap&key=$KEY"
```
🟡 49. decimal ↔ DMS coordinates
```python
def dms(d): deg=int(d); m=int((abs(d)-abs(deg))*60); s=(abs(d)-abs(deg)-m/60)*3600; return deg,m,round(s,2)
print(dms(38.6916))
```

## Social / public media
🟡 50. snscrape with keyword
```bash
snscrape --max-results 100 twitter-search "from:targetuser flag" 
```
🟡 51. download public media (gallery-dl)
```bash
gallery-dl "https://www.instagram.com/p/POSTID/"
```
🟡 52. nitter RSS (alt frontend)
```bash
curl -s "https://nitter.net/targetuser/rss" | grep -oE '<title>[^<]+'
```
🟡 53. extract geotag from a downloaded video frame
```bash
ffmpeg -i vid.mp4 -vframes 1 frame.jpg && exiftool frame.jpg
```

## Documents
🟡 54. harvest site PDFs' metadata (metagoofil)
```bash
metagoofil -d example.com -t pdf,docx -o out/ -l 50
```
🟡 55. PDF metadata
```bash
exiftool report.pdf ; pdfinfo report.pdf
```
🟡 56. Office doc metadata + macros
```bash
exiftool doc.docx ; olevba doc.docm
```
🟡 57. pull author across many docs
```bash
exiftool -Author -Creator -LastModifiedBy *.pdf *.docx
```

## Archives / history
🟡 58. Wayback snapshots list (CDX)
```bash
curl -s "http://web.archive.org/cdx/search/cdx?url=example.com*&output=text&limit=50"
```
🟡 59. fetch a specific snapshot
```bash
curl -s "http://web.archive.org/web/2019id_/https://example.com/"
```
🟡 60. Wayback "oldest" of a URL
```bash
curl -s "http://archive.org/wayback/available?url=example.com&timestamp=2015"
```
🟡 61. CommonCrawl index lookup
```bash
curl -s "https://index.commoncrawl.org/CC-MAIN-2024-10-index?url=example.com*&output=json" | head
```
🟡 62. Pastebin dump search (psbdmp)
```bash
curl -s "https://psbdmp.ws/api/search/example.com"
```

## Code / leaks (public)
🟡 63. GitHub code search (API)
```bash
curl -s -H "Authorization: token $GH" "https://api.github.com/search/code?q=example.com+password" | grep -oE '"html_url":[^,]+' | head
```
🟡 64. gitleaks on a cloned repo
```bash
gitleaks detect -s ./repo -v
```
🟡 65. trufflehog a repo
```bash
trufflehog git https://github.com/org/repo
```
🟡 66. reconstruct exposed .git
```bash
git-dumper https://target/.git/ out/
```
🟡 67. search removed secret in git history
```bash
git -C repo log -p -S 'API_KEY' | grep -A2 API_KEY
```

## Framework runs
🟡 68. theHarvester
```bash
theHarvester -d example.com -b bing,crtsh,duckduckgo
```
🟡 69. spiderfoot headless scan
```bash
sf.py -s example.com -m sfp_dnsresolve,sfp_crt,sfp_whois
```
🟡 70. recon-ng workspace
```
workspaces create ctf ; modules load recon/domains-hosts/hackertarget ; run
```
🟡 71. phoneinfoga scan
```bash
phoneinfoga scan -n "+15551234567"
```
🟡 72. GHunt email (public Google data)
```bash
ghunt email target@gmail.com
```

## Pivot / verify helpers
🟡 73. extract all links from a page
```python
import requests,re
print(set(re.findall(r'https?://[^\s"\'<>]+', requests.get("https://example.com").text)))
```
🟡 74. pull a bio field (generic)
```python
import requests; print(requests.get("https://api.github.com/users/targetuser").json().get("blog"))
```
🟡 75. find email in a page
```bash
curl -s https://example.com | grep -oiE '[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}'
```
🟡 76. extract social handles from a page
```bash
curl -s https://example.com | grep -oiE '(twitter|instagram|github|linkedin)\.com/[A-Za-z0-9_./-]+' | sort -u
```
🟡 77. resolve shortlink / final URL
```bash
curl -sIL https://bit.ly/xyz | grep -i ^location
```
🟡 78. screenshot a page (headless)
```bash
chromium --headless --screenshot=shot.png --window-size=1280,2000 https://example.com
```
🟡 79. archive a page yourself (archive.today)
```bash
curl -s "https://archive.ph/submit/?url=https://example.com"
```
🟡 80. whois history (whoisxml API)
```bash
curl -s "https://whois-history.whoisxmlapi.com/api/v1?apiKey=$KEY&domainName=example.com"
```

## Transport trackers (public)
🟡 81. flights over an area (OpenSky)
```bash
curl -s "https://opensky-network.org/api/states/all?lamin=38&lomin=-10&lamax=39&lomax=-9" | head -c 400
```
🟡 82. vessel lookup (MarineTraffic page)
```
https://www.marinetraffic.com/en/ais/details/ships/mmsi:<MMSI>
```
🟡 83. wifi SSID → location (WiGLE page)
```
https://wigle.net/search?ssid=<SSID>
```

## Misc / glue
🟢 84. normalize a username set
```python
u="John.Doe"; print([u.lower(), u.replace(".",""), u.replace(".","_")])
```
🟡 85. batch reverse-image URLs
```python
img="https://site/pic.jpg"
for e in ["https://yandex.com/images/search?rpt=imageview&url=","https://www.bing.com/images/search?q=imgurl:"]:
    print(e+img)
```
🟡 86. geolocate IP of a domain
```bash
curl -s "https://ipinfo.io/$(dig +short example.com | head -1)/json"
```
🟡 87. extract EXIF from every image in a folder
```bash
exiftool -gps:all -filename -r ./images/ | grep -B1 Position
```
🟡 88. pull coordinates from many photos to CSV
```bash
exiftool -csv -gpslatitude -gpslongitude -c "%.6f" *.jpg > coords.csv
```
🟡 89. map a CSV of coords (folium)
```python
import csv,folium
m=folium.Map()
for r in csv.DictReader(open("coords.csv")):
    try: folium.Marker([float(r['GPSLatitude']),float(r['GPSLongitude'])]).add_to(m)
    except: pass
m.save("map.html")
```
🟡 90. decode a what3words-looking hint
```
# three dotted words → api.what3words convert-to-coordinates (script 48)
```
🟡 91. username → gravatar profile JSON
```bash
curl -s "https://en.gravatar.com/$(python3 -c "import hashlib;print(hashlib.md5(b'target@example.com').hexdigest())").json"
```
🟡 92. extract metadata from a downloaded doc set
```bash
metagoofil -d example.com -t pdf -o docs/ -l 20 && exiftool -Author docs/*
```
🟡 93. find subdomains (multi-tool merge)
```bash
{ subfinder -silent -d example.com; assetfinder --subs-only example.com; } | sort -u
```
🟡 94. probe which subdomains are live
```bash
cat subs.txt | httpx -silent -title -status-code
```
🟡 95. screenshot many hosts
```bash
cat live.txt | gowitness file -f -   # or: eyewitness
```
🟡 96. reverse WHOIS by email (viewdns)
```
https://viewdns.info/reversewhois/?q=registrant@example.com
```
🟡 97. certificate SANs for pivoting
```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"
```
🟡 98. find the one page a term lives on (site crawl)
```bash
curl -s https://example.com/sitemap.xml | grep -oE '<loc>[^<]+' | sed 's/<loc>//' | while read u; do curl -s "$u" | grep -l flag && echo "$u"; done
```
🟡 99. OSINT notes template (markdown)
```bash
printf '# Target\n## Seeds\n## Pivots\n- [ ]\n## Findings\n## Flag\n' > osint_notes.md
```
🟢 100. the flow
```
# seed → (search/dork, username, domain/cert, image EXIF, archive) → verify → pivot → flag
# log every hop in osint_notes.md; stay in scope (public + challenge target only)
```

---

## 🗺️ DCTF Roadmap — OSINT

> OSINT in DCTF is a **Jeopardy quals** category: the organizers planted the answer in public data. It's a scavenger hunt, not surveillance. See [[Roadmap]]. Stay in scope (ethics banner at top).

### What to study (in order)
1. **Methodology** — seed → pivot → verify → log. (techniques A, M)
2. **Search & dorking** — operators, GHDB, multi-engine, Wayback. (techniques B, K)
3. **Username/email pivots** — Sherlock/Maigret/holehe (public). (techniques C–D)
4. **Domain/cert/DNS** — WHOIS, crt.sh, subdomains. (techniques E)
5. **Image + geolocation** — EXIF, reverse search, visual clues, maps. (techniques G–H)
6. **Infra + archives + code leaks** — Shodan/Censys, Wayback, GitHub. (techniques F, K, L)

### How to study
- **Pivoting is the skill, not the tools.** Practice turning one breadcrumb into the next.
- Keep a notes file every time (template in script 99) — OSINT is lost by forgetting a lead.
- Learn **geolocation meta** (driving side, signage, bollards, vegetation) — it wins image challenges.
- Verify each find from a second source before you commit.

### How to exercise
- **GeoGuessr** (free daily) for image geolocation reflexes.
- **Sourcing/OSINT challenge sites** (TraceLabs CTFs are missing-persons focused and real — only via sanctioned events), **picoCTF**/general CTF OSINT categories, @quiztime geolocation puzzles.
- Drill: given one username or one photo, reach a verified location/identity-of-the-planted-target in <15 min, logging each pivot.

### How to do things (per-challenge workflow)
1. Identify the seed type (handle / domain / image / IP). (technique 2)
2. Run the matching first tool: Sherlock (handle), crt.sh+whois (domain), ExifTool+reverse search (image), Shodan (IP). (scripts 7, 18–23, 36–42, 30)
3. Pivot on every field you get (bio link → domain → WHOIS email → …). (technique 3)
4. Check archives/caches if the live page is scrubbed. (scripts 58–62)
5. Verify, then submit; log the full chain in your notes.

### In the DCTF A/D final
- Not a finals category. But recon habits (cert transparency, subdomain enum, Shodan) help you **understand the game infra** and your own exposed services quickly.

> Tags: #ctf #osint #techniques #catalog #scripts #roadmap #dctf

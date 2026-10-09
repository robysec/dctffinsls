# Networking — Complete (Techniques + Catalog + Scripts + Roadmap)

All-in-one for CTF networking: packet captures, protocols, extraction, covert channels, wireless, USB, and lab scanning/crafting. Pairs with [[Forensics-Complete]] (pcap overlaps) and [[Home]].

> Mental model: a network challenge hands you a capture (or a live service in a lab) and hides the flag in *what was sent*. Your job: find the right conversation, reassemble it, and decode what's inside. Only scan/attack hosts you're authorized to (CTF scope or your own lab).

---

# TECHNIQUES (100, explained + scenarios)

## A. Capture & triage

1. **capture to file** — record traffic for later. *Scenario:* `tcpdump -i any -w out.pcap` saves everything so you can grep it offline.
2. **open a pcap** — load it in Wireshark/tshark. *Scenario:* you're handed `traffic.pcap`; first look is the packet list.
3. **protocol hierarchy** — see the mix of protocols. *Scenario:* Stats show 3% FTP in a sea of HTTP — that FTP is the clue.
4. **conversations view** — who talked to whom, how much. *Scenario:* one host pair moved 2 MB; that's your target stream.
5. **endpoints view** — list hosts + volume. *Scenario:* a single external IP stands out as the exfil server.
6. **display vs capture filters** — filter shown vs recorded. *Scenario:* `http.request` narrows 100k packets to the 12 that matter.
7. **follow TCP stream** — reassemble a conversation. *Scenario:* the stream shows a plaintext login and the flag in a response.
8. **follow UDP/HTTP2 stream** — same for non-TCP. *Scenario:* a UDP game protocol carries the flag in one datagram.
9. **export objects** — pull transferred files. *Scenario:* File→Export Objects→HTTP drops a hidden zip to disk.
10. **IO/throughput graph** — spot bursts/beacons. *Scenario:* a regular blip every 30s reveals C2 beaconing.
11. **time filtering** — focus a window. *Scenario:* the incident happened at 14:03; filter to that minute.
12. **repair a broken pcap** — fix truncation. *Scenario:* `pcapfix` makes a cut-off capture openable.

## B. Protocol analysis

13. **HTTP analysis** — requests/responses, headers, bodies. *Scenario:* a `X-Flag` header or a form POST holds the token.
14. **TLS/HTTPS (with key)** — decrypt given keylog/key. *Scenario:* an `SSLKEYLOGFILE` provided → Wireshark shows plaintext.
15. **DNS analysis** — queries/answers. *Scenario:* odd long subdomains hint at DNS exfil.
16. **FTP** — control + data channels. *Scenario:* USER/PASS in cleartext; the data channel transfers the flag file.
17. **SMTP/IMAP/POP** — email traffic. *Scenario:* a base64 MIME attachment decodes to the flag.
18. **Telnet** — cleartext shell. *Scenario:* follow the stream to watch commands + output including the flag.
19. **SMB/CIFS** — file shares. *Scenario:* a file read over SMB can be carved from the capture.
20. **SNMP** — community strings + OIDs. *Scenario:* `public` community leaks device info with the flag.
21. **DHCP** — host/option fields. *Scenario:* a vendor option or hostname carries a clue.
22. **ARP** — L2 mapping / spoofing. *Scenario:* gratuitous ARP shows a MITM in the capture.
23. **ICMP** — echo payloads. *Scenario:* ping data fields spell the flag byte by byte.
24. **NTP/Kerberos/LDAP/RDP** — service-specific fields. *Scenario:* a Kerberos AS-REP yields a crackable hash.
25. **MQTT / industrial (modbus)** — IoT/ICS protocols. *Scenario:* an MQTT publish payload is the flag.
26. **HTTP/2 & gRPC** — binary framing. *Scenario:* decode HPACK headers / protobuf body for the data.
27. **WebSocket frames** — decode WS messages. *Scenario:* the flag streams over a websocket after upgrade.
28. **TFTP** — simple file transfer. *Scenario:* a TFTP read reconstructs a config with secrets.
29. **chunked / gzip HTTP** — reassemble + decompress. *Scenario:* the body is gzipped; decompress to read it.
30. **fragmented reassembly** — merge IP/TCP fragments. *Scenario:* a payload split across packets needs reassembly first.

## C. Extraction

31. **cleartext credential hunt** — grep user/pass. *Scenario:* Basic-Auth header base64-decodes to creds.
32. **file carving from HTTP** — export or carve bodies. *Scenario:* a PNG transferred over HTTP holds the flag.
33. **http.file_data dump** — tshark field extract. *Scenario:* rebuild an uploaded file from request bodies.
34. **image reassembly** — pull images from traffic. *Scenario:* an avatar upload is the stego carrier.
35. **cookie/token extraction** — session values. *Scenario:* a JWT in a Cookie header decodes to admin claims.
36. **email attachment extract** — MIME parts. *Scenario:* decode the base64 attachment to a document.
37. **SMB file carve** — reconstruct a shared file. *Scenario:* carve the copied document from SMB reads.
38. **FTP data-channel file** — follow the data stream. *Scenario:* the retrieved file is the flag.
39. **base64/hex in bodies** — decode embedded blobs. *Scenario:* a response body is a base64 flag.
40. **user-agent / header clues** — oddities. *Scenario:* a custom UA string is the flag itself.
41. **certificate fields (TLS)** — CN/SAN/serial. *Scenario:* the flag is stuffed in a cert's Common Name.
42. **DNS answer data** — TXT/CNAME payloads. *Scenario:* a TXT record spells the flag.
43. **multi-stream correlation** — stitch related flows. *Scenario:* a key in one stream decrypts data in another.
44. **follow then decode chain** — layered decode. *Scenario:* stream → base64 → gzip → flag.
45. **tshark field export** — scriptable extraction. *Scenario:* pull one field across thousands of packets to a file.

## D. Covert channels / exfiltration

46. **DNS subdomain exfil** — data in query names. *Scenario:* base32 chunks across `a1b2.exfil.com` reassemble the flag.
47. **DNS TXT exfil** — data in TXT answers. *Scenario:* each response carries a slice of the secret.
48. **ICMP exfil** — data in echo payload. *Scenario:* each ping packet holds one flag byte.
49. **TCP flag/seq covert channel** — bits in header fields. *Scenario:* SYN/ACK patterns or seq LSBs encode a message.
50. **timing covert channel** — inter-arrival gaps. *Scenario:* long/short delays encode 1s and 0s.
51. **TTL / IP-ID channel** — data in rarely-checked fields. *Scenario:* the IP ID field carries bytes across packets.
52. **port-knock sequence** — ordered ports as a code. *Scenario:* the knock order spells/unlocks the flag.
53. **HTTP header smuggling** — data in custom headers. *Scenario:* a sequence of `X-` headers reassembles the flag.
54. **packet-size encoding** — lengths as data. *Scenario:* payload sizes map to characters.
55. **unused/reserved fields** — hidden bits. *Scenario:* reserved TCP bits or padding carry the payload.
56. **beacon detection** — periodic callbacks. *Scenario:* regular intervals expose the C2 channel to follow.
57. **tunneling (dns2tcp/iodine)** — a shell inside DNS. *Scenario:* reconstruct the tunneled session to read commands.
58. **protocol mismatch** — wrong data on a port. *Scenario:* "HTTP" on 53 or raw bytes on 80 flags the covert channel.

## E. Wireless

59. **WPA handshake capture** — the 4-way handshake. *Scenario:* capture EAPOL frames to crack the PSK.
60. **WPA/WPA2 PSK crack** — dictionary on the handshake. *Scenario:* `aircrack-ng -w rockyou` recovers the passphrase.
61. **PMKID attack** — crack without a client. *Scenario:* hcxtools pulls a PMKID; hashcat cracks it.
62. **WEP crack** — IV weakness. *Scenario:* enough IVs → instant key recovery.
63. **deauth spotting** — forced disconnects in capture. *Scenario:* deauth frames explain why a handshake appears.
64. **probe-request analysis** — device's known networks. *Scenario:* probed SSIDs reveal a location clue.
65. **hidden SSID reveal** — from association frames. *Scenario:* a "hidden" network's name appears in a probe response.
66. **decrypt WiFi traffic** — with the cracked key. *Scenario:* after the PSK, Wireshark shows the plaintext HTTP + flag.
67. **Bluetooth capture** — BLE/BR-EDR frames. *Scenario:* a BLE characteristic write carries the flag.
68. **Zigbee/802.15.4** — IoT mesh frames. *Scenario:* a sensor payload is the clue.

## F. USB / peripheral captures

69. **USB HID keyboard decode** — typed keys from capdata. *Scenario:* `usb.capdata` → decode to the typed flag.
70. **USB mouse path** — reconstruct movement/drawing. *Scenario:* plotting dx/dy draws the flag.
71. **USB mass-storage carve** — files over USB. *Scenario:* reassemble a file copied to a stick.
72. **capdata parsing** — generic HID reports. *Scenario:* map report bytes to the device's spec.
73. **serial/UART capture** — console over serial. *Scenario:* the UART log prints the flag at boot.
74. **logic-analyzer decode (SPI/I2C)** — decode bus signals. *Scenario:* sigrok decodes I2C to reveal an EEPROM's contents.

## G. Scanning / active (lab or CTF scope only)

75. **host discovery** — who's alive. *Scenario:* `nmap -sn` finds the one live box on the subnet.
76. **port scan** — open ports. *Scenario:* an unusual high port hosts the challenge service.
77. **service/version detect** — what's running. *Scenario:* `-sV` fingerprints a vulnerable version.
78. **NSE scripts** — nmap's script engine. *Scenario:* `--script` enumerates SMB shares exposing the flag.
79. **UDP scan** — UDP services. *Scenario:* an open SNMP/TFTP port is the way in.
80. **banner grab** — read service banners. *Scenario:* `nc host port` returns a banner containing the flag.
81. **ping sweep / traceroute** — map the path. *Scenario:* traceroute reveals an intermediate host of interest.
82. **OS fingerprint** — guess the OS. *Scenario:* `-O` hints which exploit path fits.
83. **masscan (fast)** — huge ranges quickly. *Scenario:* scan a /16 in seconds to find the service.
84. **enumerate shares/services** — smbclient/enum4linux/snmpwalk. *Scenario:* a readable share holds the flag file.
85. **vhost/DNS brute** — hidden names. *Scenario:* a vhost `admin.target` serves the real app.

## H. Crafting / interaction

86. **craft a packet (scapy)** — build any frame. *Scenario:* send a custom packet a service expects to reply to.
87. **ARP interaction** — who-has/reply. *Scenario:* answer an ARP to receive traffic in a lab.
88. **custom ICMP** — set payload/type. *Scenario:* send the exact ICMP a challenge waits for.
89. **DNS query craft** — specific record types. *Scenario:* request a TXT that returns the flag.
90. **replay traffic (tcpreplay)** — resend a capture. *Scenario:* replay a recorded auth to trigger a response.
91. **spoof source** — forge src IP (lab). *Scenario:* a service trusts a src IP; spoof it in your lab.
92. **protocol fuzzing** — malformed inputs. *Scenario:* a bad field crashes/reveals a service bug.
93. **write a parser** — decode a custom protocol. *Scenario:* reverse the binary framing to read messages.
94. **interactive client** — speak the protocol. *Scenario:* scripted netcat/scapy negotiates and grabs the flag.

## I. Decode / crack in-network

95. **TLS decrypt w/ keylog** — plaintext from HTTPS. *Scenario:* given keys, read the encrypted flag transfer.
96. **Kerberos hash extract** — AS-REP/TGS to crack. *Scenario:* pull the hash from the capture, crack offline.
97. **NTLM from capture** — NTLMv2 challenge/response. *Scenario:* extract and crack to recover a password.
98. **SNMP community/creds** — from SNMP packets. *Scenario:* community string unlocks device data.
99. **binary protocol decode** — protobuf/MessagePack. *Scenario:* decode the structured body to fields.
100. **the networking flow** — triage → find the stream → reassemble → decode → flag. *Scenario:* every net challenge reduces to this loop.

---

# CATALOG (100 tools)

## Capture & analysis
1. **Wireshark** — GUI packet analysis `wireshark.org`
2. **tshark** — CLI Wireshark
3. **tcpdump** — capture/read `apt install tcpdump`
4. **tcpflow** — reassemble TCP streams
5. **NetworkMiner** — passive host/file extraction
6. **Scapy** — craft/parse packets `pip install scapy`
7. **pyshark** — python wrapper over tshark `pip install pyshark`
8. **dpkt** — fast pcap parsing lib `pip install dpkt`
9. **Zeek (Bro)** — network analysis framework `zeek.org`
10. **Suricata** — IDS, pcap replay `suricata.io`
11. **ngrep** — grep over packets `apt install ngrep`
12. **Arkime (Moloch)** — large-scale capture index
13. **Brim / Zui** — Zeek+pcap desktop analyzer
14. **Security Onion** — monitoring distro
15. **Chaosreader** — extract sessions from pcap
16. **Wireshark Lua** — custom dissectors
17. **capinfos** — pcap summary (Wireshark suite)
18. **editcap** — slice/convert pcaps
19. **mergecap** — merge captures
20. **pcapfix** — repair captures `github.com/Rc0r/pcapfix`

## Extraction / carving
21. **foremost** — carve files from pcap/blob
22. **tcpxtract** — extract files by signature
23. **NetworkMiner (files tab)** — auto file extraction
24. **Wireshark Export Objects** — HTTP/SMB/TFTP/IMF files
25. **tshark -z follow** — scripted stream dump
26. **tshark --export-objects** — CLI object export
27. **PcapXray** — visualize a pcap graphically
28. **python-evtx / scapy sessions** — programmatic extraction
29. **CyberChef** — decode carved blobs
30. **binwalk** — embedded files in carved blobs

## DNS / recon
31. **dig / nslookup / host** — DNS queries `apt install dnsutils`
32. **dnsrecon** — DNS enumeration
33. **dnsenum** — DNS brute/zone
34. **fierce** — DNS recon
35. **dnsdumpster** — passive DNS (web)
36. **dns2tcp / iodine** — DNS tunneling (decode/setup)
37. **tcpdump 'udp port 53'** — isolate DNS
38. **passivedns** — log DNS from pcap

## Scanning (lab/CTF scope)
39. **nmap** — the scanner `nmap.org`
40. **nmap NSE** — scripted enumeration
41. **masscan** — internet-scale fast scan
42. **RustScan** — fast port scan → nmap
43. **netcat (nc/ncat)** — connect/listen/banner
44. **socat** — advanced sockets/relays
45. **hping3** — custom packet probes
46. **netdiscover** — ARP host discovery
47. **nbtscan** — NetBIOS scan
48. **arp-scan** — L2 host discovery
49. **traceroute / mtr** — path mapping
50. **unicornscan** — async scanner

## Service enumeration
51. **smbclient** — SMB shares
52. **enum4linux / enum4linux-ng** — SMB/Samba enum
53. **snmpwalk / snmpget** — SNMP queries
54. **onesixtyone** — SNMP community brute
55. **ike-scan** — IPSec/IKE enum
56. **rpcclient** — MSRPC enum
57. **ldapsearch** — LDAP queries
58. **showmount** — NFS exports
59. **Impacket suite** — SMB/Kerberos/MSRPC tooling `github.com/fortra/impacket`
60. **CrackMapExec / NetExec** — network auth sweep

## Wireless
61. **aircrack-ng suite** — airodump/aireplay/aircrack `aircrack-ng.org`
62. **airmon-ng** — monitor mode
63. **hcxdumptool / hcxtools** — PMKID capture/convert
64. **hashcat** — crack WPA/PMKID (`-m 22000`)
65. **Wifite** — automated wireless auditing
66. **Kismet** — wireless sniffer/detector
67. **Reaver / Bully** — WPS attacks
68. **bettercap** — WiFi/BLE/HID MITM framework
69. **Ubertooth / BtleJack** — Bluetooth LE
70. **gr-gsm / GNURadio** — SDR protocol work

## MITM / interception (lab)
71. **mitmproxy** — HTTP(S) intercept/modify `mitmproxy.org`
72. **Ettercap** — LAN MITM
73. **Bettercap** — modern MITM framework
74. **Responder** — LLMNR/NBT-NS poisoning (lab) `github.com/lgandx/Responder`
75. **arpspoof / dsniff** — ARP MITM + sniff
76. **sslstrip / SSLsplit** — TLS downgrade/intercept (lab)
77. **tcpreplay** — replay captures `tcpreplay.appneta.com`
78. **proxychains** — route tools through proxies
79. **websocat** — websocket client
80. **grpcurl** — gRPC client

## USB / hardware / SDR
81. **USBPcap / usbmon** — capture USB
82. **usbrip** — USB event history
83. **sigrok / PulseView** — logic analyzer decode
84. **rtl_433** — decode ISM-band devices (SDR)
85. **gqrx / SDR#** — SDR receivers
86. **GNURadio Companion** — signal flowgraphs
87. **Saleae Logic** — logic capture (HW)
88. **HID report parsers** — decode usb.capdata

## Decode / crack helpers
89. **JA3/JA3S** — TLS client/server fingerprint
90. **ja3er / tls-fingerprint libs** — identify clients
91. **krbrelayx / Impacket getTGT** — Kerberos
92. **NetNTLM crackers (hashcat -m 5600)** — NTLMv2
93. **protobuf-inspector** — decode unknown protobuf
94. **blackbox-protobuf** — decode/edit protobuf
95. **asn1crypto / pyasn1** — decode ASN.1 (TLS/Kerberos)
96. **mqtt-explorer / mosquitto_sub** — MQTT
97. **modbus-cli / pymodbus** — ICS/modbus
98. **iodine (decode)** — reconstruct DNS tunnels
99. **your usb_hid_decode.py** — `99-Scripts/usb_hid_decode.py`
100. **PayloadsAllTheThings (Network)** — technique refs `github.com/swisskyrepo/PayloadsAllTheThings`

## Install quick ref
```bash
sudo apt install -y wireshark tshark tcpdump tcpflow ngrep nmap masscan \
  ncat socat hping3 dnsutils dnsrecon smbclient snmp onesixtyone \
  aircrack-ng kismet ettercap-text-only mitmproxy tcpreplay
pip install scapy pyshark dpkt impacket
```

---

# SCRIPTS (100 snippets)

`pip install scapy pyshark dpkt` and apt tools above. 🟢 easy / 🟡 medium / 🔴 hard.

## Capture / read / triage
🟢 1. capture to file
```bash
sudo tcpdump -i any -w out.pcap
```
🟢 2. capture with a filter
```bash
sudo tcpdump -i eth0 'tcp port 80 or port 443' -w web.pcap
```
🟢 3. read a pcap (CLI)
```bash
tshark -r cap.pcap | head
```
🟢 4. protocol hierarchy
```bash
tshark -r cap.pcap -q -z io,phs
```
🟢 5. conversations (top talkers)
```bash
tshark -r cap.pcap -q -z conv,tcp
```
🟢 6. endpoints list
```bash
tshark -r cap.pcap -q -z endpoints,ip
```
🟡 7. follow a TCP stream (by index)
```bash
tshark -r cap.pcap -q -z follow,tcp,ascii,0
```
🟡 8. list all HTTP requests
```bash
tshark -r cap.pcap -Y http.request -T fields -e http.host -e http.request.uri
```
🟢 9. export HTTP objects (files)
```bash
tshark -r cap.pcap --export-objects http,out/
```
🟡 10. capinfos summary
```bash
capinfos cap.pcap
```
🟡 11. slice a time window
```bash
editcap -A "2026-01-01 14:03:00" -B "2026-01-01 14:04:00" cap.pcap window.pcap
```
🟡 12. repair a broken pcap
```bash
pcapfix cap.pcap
```

## Extraction
🟡 13. dump HTTP bodies
```bash
tshark -r cap.pcap -Y http -T fields -e http.file_data | tr -d '\n' | xxd -r -p > body.bin
```
🟡 14. grep Basic-Auth creds
```bash
tshark -r cap.pcap -Y http.authorization -T fields -e http.authorization
```
🟡 15. decode Basic-Auth value
```python
import base64; print(base64.b64decode("YWRtaW46cGFzcw=="))
```
🟡 16. extract all URLs visited
```bash
tshark -r cap.pcap -Y http.request -T fields -e http.host -e http.request.uri | sort -u
```
🟡 17. pull DNS queries
```bash
tshark -r cap.pcap -Y dns.flags.response==0 -T fields -e dns.qry.name | sort -u
```
🟡 18. pull DNS TXT answers
```bash
tshark -r cap.pcap -Y dns.txt -T fields -e dns.txt
```
🟡 19. follow + save a stream to file
```bash
tshark -r cap.pcap -q -z follow,tcp,raw,3 | tr -d '\n' | xxd -r -p > stream3.bin
```
🟡 20. extract FTP commands
```bash
tshark -r cap.pcap -Y ftp -T fields -e ftp.request.command -e ftp.request.arg
```
🟡 21. extract Telnet data
```bash
tshark -r cap.pcap -Y telnet -T fields -e telnet.data | tr -d '\n'
```
🟡 22. SMTP/IMF email bodies
```bash
tshark -r cap.pcap --export-objects imf,mail/
```
🟡 23. cookies / tokens
```bash
tshark -r cap.pcap -Y http.cookie -T fields -e http.cookie
```
🟡 24. decompress a gzip HTTP body (python)
```python
import gzip; open("out","wb").write(gzip.decompress(open("body.bin","rb").read()))
```
🟡 25. pyshark iterate packets
```python
import pyshark
for p in pyshark.FileCapture("cap.pcap", display_filter="http"):
    print(p.http.get_field_value("request_full_uri"))
```

## Covert channels / exfil
🟡 26. reassemble DNS-subdomain exfil
```python
import base64
names=[l.split(".")[0] for l in open("dns.txt")]
print(base64.b32decode("".join(names).upper()+"==="))
```
🟡 27. ICMP payload bytes
```bash
tshark -r cap.pcap -Y icmp -T fields -e data | tr -d '\n' | xxd -r -p
```
🟡 28. ICMP exfil decode (scapy)
```python
from scapy.all import rdpcap, ICMP, Raw
print(b"".join(bytes(p[Raw].load) for p in rdpcap("cap.pcap") if p.haslayer(ICMP) and p.haslayer(Raw)))
```
🟡 29. TCP seq LSB channel
```python
from scapy.all import rdpcap, TCP
bits="".join(str(p[TCP].seq & 1) for p in rdpcap("cap.pcap") if p.haslayer(TCP))
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8)))
```
🟡 30. IP ID field channel
```python
from scapy.all import rdpcap, IP
print(bytes(p[IP].id & 0xff for p in rdpcap("cap.pcap") if p.haslayer(IP))[:120])
```
🔴 31. packet-timing channel (gaps → bits)
```python
from scapy.all import rdpcap
ts=[float(p.time) for p in rdpcap("cap.pcap")]
gaps=[ts[i+1]-ts[i] for i in range(len(ts)-1)]
bits="".join("1" if g>0.5 else "0" for g in gaps)
print(bytes(int(bits[i:i+8],2) for i in range(0,len(bits)//8*8,8)))
```
🟡 32. port-knock sequence
```bash
tshark -r cap.pcap -Y 'tcp.flags.syn==1 && tcp.flags.ack==0' -T fields -e tcp.dstport | tr '\n' ' '
```
🟡 33. packet sizes → chars
```python
from scapy.all import rdpcap
print("".join(chr(len(p)) for p in rdpcap("cap.pcap") if 32<=len(p)<127))
```

## USB
🟡 34. pull USB HID capdata
```bash
tshark -r cap.pcap -Y usb.capdata -T fields -e usb.capdata > hid.txt
```
🟡 35. decode USB keyboard (your script)
```bash
python3 99-Scripts/usb_hid_decode.py hid.txt
```
🔴 36. USB mouse path → image
```python
import matplotlib.pyplot as plt
x=y=0; xs=[]; ys=[]
for line in open("hid.txt"):
    b=bytes.fromhex(line.strip());
    dx=b[1]-256 if b[1]>127 else b[1]; dy=b[2]-256 if b[2]>127 else b[2]
    x+=dx; y+=dy; xs.append(x); ys.append(-y)
plt.scatter(xs,ys,s=1); plt.savefig("mouse.png")
```

## Protocol decode
🟡 37. TLS cert CN/SAN
```bash
tshark -r cap.pcap -Y tls.handshake.type==11 -T fields -e x509sat.printableString
```
🟡 38. decrypt TLS with keylog (Wireshark)
```bash
# Wireshark → Prefs → Protocols → TLS → (Pre)-Master-Secret log filename = keys.log
tshark -r cap.pcap -o tls.keylog_file:keys.log -Y http2 -T fields -e http2.data.data
```
🟡 39. websocket frames
```bash
tshark -r cap.pcap -Y websocket -T fields -e websocket.payload.text
```
🟡 40. MQTT publish payloads
```bash
tshark -r cap.pcap -Y mqtt.msgtype==3 -T fields -e mqtt.msg
```
🟡 41. extract Kerberos AS-REP hash
```bash
tshark -r cap.pcap -Y 'kerberos.msg_type==11' -T fields -e kerberos.cipher
```
🔴 42. decode unknown protobuf body
```bash
echo "<hex>" | xxd -r -p | protoc --decode_raw
```
🟡 43. SNMP community strings
```bash
tshark -r cap.pcap -Y snmp -T fields -e snmp.community | sort -u
```
🟡 44. DHCP hostnames/options
```bash
tshark -r cap.pcap -Y bootp -T fields -e dhcp.option.hostname
```
🟡 45. ARP table from capture
```bash
tshark -r cap.pcap -Y arp -T fields -e arp.src.proto_ipv4 -e arp.src.hw_mac | sort -u
```

## Wireless
🟡 46. isolate EAPOL (WPA handshake)
```bash
tshark -r wifi.pcap -Y eapol
```
🟡 47. convert for hashcat (PMKID/handshake)
```bash
hcxpcapngtool -o hash.22000 wifi.pcapng
```
🟡 48. crack WPA with hashcat
```bash
hashcat -m 22000 hash.22000 rockyou.txt
```
🟡 49. crack WPA with aircrack
```bash
aircrack-ng -w rockyou.txt wifi.pcap
```
🟡 50. list SSIDs / probes
```bash
tshark -r wifi.pcap -Y 'wlan.fc.type_subtype==4' -T fields -e wlan.ssid | sort -u
```
🟡 51. decrypt WiFi after key (Wireshark)
```bash
# Prefs → IEEE 802.11 → Decryption keys → wpa-pwd:<pass>:<ssid>
```

## Scanning / active (lab/CTF scope)
🟢 52. nmap host discovery
```bash
nmap -sn 10.0.0.0/24
```
🟢 53. nmap fast top ports
```bash
nmap -F 10.0.0.5
```
🟢 54. nmap full + version + scripts
```bash
nmap -sC -sV -p- 10.0.0.5
```
🟡 55. nmap UDP top ports
```bash
sudo nmap -sU --top-ports 50 10.0.0.5
```
🟡 56. nmap NSE smb enum
```bash
nmap --script smb-enum-shares,smb-enum-users -p445 10.0.0.5
```
🟢 57. banner grab (netcat)
```bash
nc -nv 10.0.0.5 21
```
🟢 58. masscan fast range
```bash
sudo masscan -p1-65535 10.0.0.0/24 --rate 10000
```
🟡 59. enum4linux
```bash
enum4linux-ng 10.0.0.5
```
🟡 60. snmpwalk
```bash
snmpwalk -v2c -c public 10.0.0.5
```
🟡 61. smbclient list shares
```bash
smbclient -L //10.0.0.5 -N
```
🟡 62. ldapsearch anonymous
```bash
ldapsearch -x -H ldap://10.0.0.5 -b "dc=target,dc=local"
```

## Crafting (scapy)
🟡 63. send an ICMP with payload
```python
from scapy.all import IP, ICMP, send
send(IP(dst="10.0.0.5")/ICMP()/b"ping-data")
```
🟡 64. craft + read a DNS query
```python
from scapy.all import IP, UDP, DNS, DNSQR, sr1
print(sr1(IP(dst="8.8.8.8")/UDP(dport=53)/DNS(rd=1,qd=DNSQR(qname="example.com",qtype="TXT")),timeout=2).an.rdata)
```
🟡 65. TCP SYN scan one port
```python
from scapy.all import IP, TCP, sr1
r=sr1(IP(dst="10.0.0.5")/TCP(dport=80,flags="S"),timeout=1); print(r.sprintf("%TCP.flags%"))
```
🟡 66. sniff live (scapy)
```python
from scapy.all import sniff
sniff(filter="tcp port 80", count=10, prn=lambda p:p.summary())
```
🟡 67. ARP who-has
```python
from scapy.all import ARP, Ether, srp
ans,_=srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="10.0.0.0/24"),timeout=2)
for _,r in ans: print(r.psrc, r.hwsrc)
```
🟡 68. replay a capture
```bash
sudo tcpreplay -i eth0 cap.pcap
```
🟡 69. custom protocol client (netcat scripted)
```bash
printf 'HELLO\nGETFLAG\n' | nc 10.0.0.5 1337
```
🔴 70. speak a binary protocol (python socket)
```python
import socket, struct
s=socket.create_connection(("10.0.0.5",1337)); s.send(struct.pack(">I",1)+b"x"); print(s.recv(1024))
```

## Decode / crack
🟡 71. extract NTLMv2 for hashcat
```bash
# use Wireshark 'ntlmssp' + pcredz, then: hashcat -m 5600 ntlm.txt rockyou.txt
```
🟡 72. decode base64 across a stream
```python
import re,base64
d=open("stream.bin","rb").read()
for m in re.findall(rb'[A-Za-z0-9+/]{16,}={0,2}', d):
    try:
        o=base64.b64decode(m)
        if b"flag" in o: print(o)
    except: pass
```
🟡 73. carve files from pcap (foremost)
```bash
tshark -r cap.pcap --export-objects http,out/ ; foremost -i cap.pcap -o carve/
```
🟡 74. extract all strings from payloads
```bash
tshark -r cap.pcap -T fields -e data | xxd -r -p | strings | grep -i flag
```
🟡 75. flag grep across tshark fields
```bash
tshark -r cap.pcap -T fields -e data -e http.file_data | xxd -r -p 2>/dev/null | grep -aoiE 'flag\{[^}]+\}'
```

## dpkt / python pcap parsing
🟡 76. iterate packets with dpkt
```python
import dpkt
for ts,buf in dpkt.pcap.Reader(open("cap.pcap","rb")):
    eth=dpkt.ethernet.Ethernet(buf); print(ts, type(eth.data).__name__)
```
🟡 77. pull HTTP requests with dpkt
```python
import dpkt
for ts,buf in dpkt.pcap.Reader(open("cap.pcap","rb")):
    ip=dpkt.ethernet.Ethernet(buf).data
    if isinstance(ip.data,dpkt.tcp.TCP) and b"HTTP" in ip.data.data:
        print(ip.data.data[:80])
```
🟡 78. reassemble a TCP stream (scapy)
```python
from scapy.all import rdpcap, TCP, Raw
data=b"".join(bytes(p[Raw].load) for p in rdpcap("cap.pcap") if p.haslayer(Raw) and p[TCP].dport==1337)
print(data[:200])
```
🟡 79. per-conversation byte counts
```python
from collections import Counter; from scapy.all import rdpcap, IP
c=Counter()
for p in rdpcap("cap.pcap"):
    if p.haslayer(IP): c[(p[IP].src,p[IP].dst)]+=len(p)
print(c.most_common(5))
```
🟡 80. extract a field everywhere (tshark)
```bash
tshark -r cap.pcap -T fields -e frame.number -e ip.src -e ip.dst -e tcp.dstport
```

## Misc / glue
🟢 81. convert pcapng → pcap
```bash
editcap -F libpcap in.pcapng out.pcap
```
🟢 82. merge captures
```bash
mergecap -w all.pcap a.pcap b.pcap
```
🟡 83. split a huge pcap
```bash
editcap -c 100000 big.pcap chunk.pcap
```
🟡 84. live grep with ngrep
```bash
sudo ngrep -q -W byline 'flag' tcp
```
🟡 85. mitmproxy record HTTPS (lab)
```bash
mitmdump -w flows.mitm
```
🟡 86. replay mitm flows
```bash
mitmdump -nr flows.mitm
```
🟡 87. socat relay/listener
```bash
socat -v TCP-LISTEN:8080,fork TCP:10.0.0.5:80
```
🟡 88. websocat connect
```bash
websocat ws://10.0.0.5:8000/ws
```
🟡 89. curl with raw output
```bash
curl -sv http://10.0.0.5/ 2>&1 | grep -i flag
```
🟡 90. hping3 custom probe
```bash
sudo hping3 -S -p 80 -c 3 10.0.0.5
```
🟡 91. decode protobuf (blackbox)
```bash
echo "<hex>" | xxd -r -p | python3 -m blackboxprotobuf decode
```
🟡 92. extract TLS JA3 (python)
```python
# use pyja3: python3 -m pyja3 --pcap cap.pcap
```
🟡 93. snmp community brute
```bash
onesixtyone -c communities.txt 10.0.0.5
```
🟡 94. tcp stream count
```bash
tshark -r cap.pcap -q -z conv,tcp | tail -n +6 | wc -l
```
🟡 95. filter by IP pair
```bash
tshark -r cap.pcap -Y "ip.addr==10.0.0.5 && ip.addr==10.0.0.9"
```
🟡 96. extract credentials (pcredz)
```bash
pcredz -f cap.pcap
```
🟡 97. zeek to logs
```bash
zeek -r cap.pcap ; cat http.log | zeek-cut host uri
```
🟡 98. decode chunked HTTP (python)
```python
import re; b=open("body.bin","rb").read()
# strip chunk sizes: split on CRLF, drop hex-length lines
print(b)
```
🟡 99. find the one odd packet (big/rare)
```bash
tshark -r cap.pcap -T fields -e frame.len | sort -n | tail
```
🟢 100. the flow (one-liner triage)
```bash
capinfos cap.pcap; tshark -r cap.pcap -q -z io,phs; tshark -r cap.pcap --export-objects http,out/
```

---

## 🗺️ DCTF Roadmap — Networking

> Networking in DCTF appears in the **Jeopardy quals** (pcap/protocol puzzles) and underpins the **A/D final** (reading traffic to/from your box). See [[DCTF Finals Playbook]] and [[Roadmap]].

### What to study (in order)
1. **Wireshark fluency** — filters, Follow Stream, Export Objects, Statistics. (techniques A)
2. **Core protocols** — HTTP, DNS, FTP/Telnet, TLS (with keys), SMB. (techniques B)
3. **Extraction** — files, creds, tokens, embedded blobs. (techniques C)
4. **Covert channels** — DNS/ICMP/timing exfil, USB HID. (techniques D, F)
5. **Wireless** — WPA handshake → crack → decrypt. (techniques E)
6. **Active (lab)** — nmap/netcat/scapy for crafting + enumeration. (techniques G–H)

### How to study
- **Live in Wireshark.** 80% of net challenges fall to Follow Stream + Export Objects.
- Learn display filters cold: `http.request`, `dns`, `tcp.stream==N`, `ip.addr==`.
- Script the repetitive stuff with tshark/scapy (the Scripts section is your kit).
- Make your own captures (curl over HTTP, a DNS exfil loop) and decode them back.

### How to exercise
- **picoCTF / HTB** networking + forensics (pcap) challenges.
- **malware-traffic-analysis.net** for realistic pcaps.
- Drill: given a pcap, name the protocol of interest and extract the flag in <10 min.

### How to do things (per-challenge workflow)
1. `capinfos` + protocol hierarchy → what's in it. (scripts 4, 100)
2. Conversations/endpoints → find the interesting stream. (scripts 5–6)
3. Follow the stream / export objects. (scripts 7, 9)
4. If nothing obvious → check covert channels (DNS/ICMP/timing/USB). (scripts 26–36)
5. Decode layers (base64/gzip/protobuf), grep for the flag. (scripts 72–75)

### In the DCTF A/D final
- Capture all traffic to your services from minute one (`tcpdump -w`). Feed it to Tulip.
- When a team pops you, their exploit is **in your capture** — follow the stream, reproduce the request, fire it back at everyone.
- Watch DNS/ICMP from your box for exfil (sign of a planted backdoor).

> Tags: #ctf #networking #techniques #catalog #scripts #roadmap #dctf

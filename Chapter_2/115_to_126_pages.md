# Chapter 2 Notes — DNS (2.4.2–2.4.3) & Video Streaming/CDN (2.5)

---

# PART A: DNS — How It Works, Records, Messages, Security

## 1. Simple meaning

DNS (Domain Name System) is the internet's phonebook. Humans like typing names like `google.com`, but computers only understand IP addresses like `142.250.183.14`. DNS converts the name you type into the IP address the computer needs to actually connect to a server.

## 2. Why it matters

- Every app you build that talks to the internet (API calls, downloading images, chat apps) relies on DNS silently working in the background.
- If DNS is slow or down, your app "can't connect" even if the server is perfectly fine — a very common debugging trap.
- Backend engineers configure DNS records (A, CNAME, MX) when deploying servers, domains, or email.
- Security engineers must understand DNS because it's a common attack surface (spoofing, cache poisoning).
- On Android, understanding DNS helps you debug `UnknownHostException` and connectivity issues.

## 3. Core idea (step by step)

1. DNS is **not one giant server** — it's a **distributed, hierarchical database** spread across millions of servers worldwide. No single server holds all mappings.
2. It's hierarchical like a tree: **Root → Top-Level Domain (TLD) → Authoritative servers**.
3. When you look up `www.example.com`, your query walks down this tree: root tells you where `.com` servers are, `.com` tells you where `example.com`'s servers are, and `example.com`'s server gives the final IP.
4. To avoid doing this full walk every time (which is slow), DNS uses **caching** at multiple levels.
5. DNS also carries more than just name→IP mapping — it stores different **record types** (A, MX, CNAME, NS) for different purposes.
6. DNS communicates using simple **query/response messages**, mostly over UDP (port 53).
7. Because DNS was designed in the early internet without strong security, it is vulnerable to several attacks — this is why DNSSEC and other protections exist.

---

## 4. Key terms

| Term | Meaning |
|---|---|
| DNS | System that translates domain names to IP addresses |
| Root DNS servers | Top of the DNS hierarchy; know where all TLD servers are |
| TLD servers | Handle domains like `.com`, `.org`, `.net`, country codes |
| Authoritative DNS server | Holds the actual, real records for a specific domain |
| Local DNS server (resolver) | Your ISP/network's server that does lookups on your behalf |
| Recursive query | Resolver does all the work and returns final answer to client |
| Iterative query | Each server gives the "next server to ask" instead of the final answer |
| DNS caching | Temporarily storing recent lookup results to speed up future queries |
| TTL (Time To Live) | How long a DNS record can be cached before it must be refreshed |
| A record | Maps a hostname to an IPv4 address |
| AAAA record | Maps a hostname to an IPv6 address |
| CNAME record | Alias — maps one hostname to another hostname |
| NS record | Specifies the authoritative name server for a domain |
| MX record | Specifies the mail server for a domain |
| DNS message | The actual packet format used for queries and replies |
| DNS spoofing/cache poisoning | Attacker inserts fake DNS records to redirect users |
| DNSSEC | Security extension that adds authentication to DNS records |

---

## 5. How it works (the lookup flow)

**Step-by-step resolution of `www.example.com`:**

1. Your device sends the query to its **local DNS server** (usually given by your ISP or Wi-Fi network).
2. If the local server has the answer cached, it replies immediately (fast path).
3. If not cached, the local server acts as a **resolver** and starts an iterative search:
   - Step A: Asks a **root server** → "Who handles `.com`?" → gets TLD server address.
   - Step B: Asks the **`.com` TLD server** → "Who handles `example.com`?" → gets the authoritative server address.
   - Step C: Asks the **authoritative server for `example.com`** → "What's the IP for `www.example.com`?" → gets the final IP.
4. The local DNS server returns this IP to your device.
5. Your device now connects directly to that IP (e.g., opens a TCP connection for HTTP).
6. The local server **caches** the result (respecting the TTL) so the next request for the same name is instant.

**DNS Caching details:**
- Caching happens at multiple levels: browser, OS, local resolver, even intermediate routers.
- Records come with a TTL (e.g., 300 seconds); after it expires, the cache is discarded and a fresh lookup happens.
- Caching massively reduces load on root and TLD servers — without it, the internet would be far slower.
- Downside: if a record changes (e.g., server IP updated), cached entries can serve stale data until TTL expires.

**DNS Messages (query & reply format, simplified):**
- Sent mostly over **UDP port 53** (fast, connectionless — fits DNS's small message size).
- A message has:
  - **Header**: identification number, flags (query/reply, recursive desired, etc.), counts of each section.
  - **Question section**: the name and record type being asked about.
  - **Answer section**: the actual record(s) returned.
  - **Authority section**: which servers are authoritative.
  - **Additional section**: extra helpful records (e.g., IP of a mail server mentioned in an MX record).

**Inserting records into the DNS database:**
- To make your domain reachable, you register it with a **registrar** (e.g., GoDaddy, Namecheap).
- You provide the registrar with the IP addresses of your **primary and secondary authoritative DNS servers**.
- The registrar inserts two key records into the parent TLD's database:
  - An **NS record** pointing to your authoritative server.
  - A **glue A record** (the IP of that name server, so it can be found).
- You then configure your own authoritative server with the actual A, CNAME, MX records for your domain.

---

## 6. Real example

When you open an app that calls `api.example.com/users`:
1. The phone's OS resolver checks cache — if empty, queries the DNS chain (root → `.com` → `example.com`'s server).
2. Gets back an A record, e.g., `93.184.216.34`.
3. Your HTTP client (like Retrofit/OkHttp on Android) opens a TCP connection to that IP on port 443 and sends the HTTPS request.
4. If the company later moves servers and updates the A record, users automatically get the new IP once their cached TTL expires — no app update needed.

## Focus on Security — DNS Vulnerabilities

- **DNS Cache Poisoning / Spoofing**: An attacker sends fake DNS responses to a resolver, tricking it into caching a wrong IP (e.g., redirecting `bank.com` to a phishing server). Because DNS traditionally has no authentication, resolvers can't easily tell fake replies from real ones.
- **DDoS attacks on DNS infrastructure**: Attackers flood root/TLD/authoritative servers with traffic to make lookups fail internet-wide (e.g., the 2016 Dyn attack disrupted major sites).
- **DNS amplification attacks**: Attackers exploit DNS's UDP nature — small spoofed queries trigger large responses sent to a victim, overwhelming them.
- **Mitigations**: DNSSEC (cryptographically signs records to prevent spoofing), using trusted resolvers (like 1.1.1.1, 8.8.8.8), rate limiting, and redundant authoritative servers.

---

## 7. Mini implementation idea (Termux/Python/Kotlin)

**Termux (quick, no coding):**
```bash
nslookup google.com
dig google.com
dig +trace google.com   # shows the full root -> TLD -> authoritative walk
```
`dig +trace` is the best way to *see* the hierarchical lookup happen live.

**Python (Termux):**
```python
import socket
import time

domains = ["google.com", "python.org", "github.com"]
for d in domains:
    start = time.time()
    ip = socket.gethostbyname(d)
    print(f"{d} -> {ip} ({(time.time()-start)*1000:.1f} ms)")
```
Run it twice — the second run will likely be faster due to OS-level DNS caching.

**Kotlin/Java (simple resolver):**
```kotlin
import java.net.InetAddress

fun main() {
    val address = InetAddress.getByName("github.com")
    println("IP: ${address.hostAddress}")
}
```

---

## 8. Common mistakes

- Thinking DNS is one central server — it's actually thousands of distributed servers.
- Confusing **recursive** query (resolver does all the work) with **iterative** query (each server just points to the next one) — the client-to-local-resolver step is usually recursive, while local-resolver-to-others is usually iterative.
- Forgetting that DNS mostly uses **UDP**, not TCP (TCP is used only for large responses like zone transfers).
- Assuming DNS changes are instant — they're delayed by **TTL caching**.
- Confusing CNAME (alias to another name) with A record (direct IP mapping).

## 9. Memory hook

Think of DNS as **asking for directions in stages**: you ask a city guide (root) which district to go to (TLD), the district guide tells you which street (authoritative server), and the street sign gives you the exact house number (IP). Once you know the way, you remember it for a while (caching) instead of asking again.

## 10. Ultra-short revision

DNS translates domain names into IP addresses using a distributed, hierarchical system of root, TLD, and authoritative servers. Local resolvers do the lookup, then cache results using a TTL to speed up future requests. DNS records (A, AAAA, CNAME, NS, MX) store different types of mapping information. DNS messages travel mostly over UDP port 53 with header, question, answer, authority, and additional sections. Domains are registered with a registrar, which inserts NS/glue records into the parent zone. Because DNS lacks built-in authentication, it's vulnerable to spoofing, cache poisoning, and DDoS — DNSSEC helps fix this.

## 11. Interview / exam questions

1. **Q: Why is DNS described as a distributed, hierarchical database?**
   A: Because no single server has all records — lookups are split across root, TLD, and authoritative servers, each responsible for part of the namespace.

2. **Q: What's the difference between a recursive and an iterative DNS query?**
   A: In recursive, the server does the full lookup and returns the final answer; in iterative, the server only returns a referral to the next server to ask.

3. **Q: What is DNS caching and why does TTL matter?**
   A: Caching stores recent lookup results to avoid repeating slow lookups; TTL controls how long that cached data stays valid before requiring a fresh query.

4. **Q: What transport protocol does DNS mainly use and why?**
   A: UDP, because DNS messages are small and connectionless, making UDP faster; TCP is used only for large transfers like zone transfers.

5. **Q: What is DNS cache poisoning?**
   A: An attack where a fake DNS response is injected into a resolver's cache, redirecting users to malicious servers instead of the real one.

---

# PART B: Video Streaming & Content Distribution Networks (2.5)

## 1. Simple meaning

This topic explains how services like YouTube or Netflix send video to millions of users smoothly. Instead of sending one giant video file and hoping your connection can handle it, video is broken into small chunks at different quality levels, and delivered from servers close to you — so it plays without long buffering.

## 2. Why it matters

- Video is the largest consumer of internet bandwidth today — understanding this shows how real production systems scale.
- Directly relevant if you ever build a video/streaming feature, or work with media APIs, or optimize app load speed.
- CDNs aren't just for video — they speed up almost every large app and website (images, JS bundles, API responses).
- Understanding adaptive streaming helps you debug "why does video quality drop" issues in apps.

## 3. Core idea (step by step)

1. Raw video files are huge, and network conditions (speed, congestion) vary constantly — a fixed-quality video would buffer badly on slow connections.
2. **Internet video** solves part of this by pre-encoding the same video at multiple quality/bitrate levels.
3. **HTTP Streaming and DASH** solves the delivery problem: the video is split into short segments, and the player intelligently requests whichever quality level fits the current network speed — switching up or down as conditions change.
4. Even with smart streaming, sending video from one central server to millions of global users would be slow and would overload that server.
5. **CDNs (Content Distribution Networks)** solve this by storing copies of the video/content on servers spread across the world, so users are served from a nearby, fast server instead of one far-away origin server.

---

## 4. Key terms

| Term | Meaning |
|---|---|
| Bitrate | Amount of video data sent per second; higher bitrate = better quality, more bandwidth needed |
| Encoding | Compressing raw video into a smaller, transmittable format (e.g., H.264, H.265) |
| Adaptive bitrate streaming | Technique where the player switches video quality based on current network speed |
| DASH (Dynamic Adaptive Streaming over HTTP) | A standard protocol for adaptive streaming using plain HTTP |
| Manifest file | A file listing all available quality versions and segment URLs for a video |
| Segment/chunk | A few-second piece of video, encoded at a specific quality level |
| CDN (Content Distribution Network) | Network of geographically distributed servers that cache and serve content close to users |
| Origin server | The original source server that holds the master copy of the content |
| Edge server | A CDN server located close to end users that serves cached content |
| Enter deep vs. bring home CDN | Two CDN deployment strategies: placing servers deep inside ISP networks vs. fewer large clusters at network hubs |
| Latency | Delay before content starts playing or responding |

---

## 5. How it works

### 5.1 Internet Video
- Video is compressed (encoded) to reduce size while keeping acceptable quality — trade-off between quality and bandwidth.
- The same video is often encoded at **multiple bitrates** (e.g., 240p, 480p, 720p, 1080p) ahead of time.
- Video streaming today is mostly **on-demand**, played through progressive download or adaptive streaming, not old-style live broadcast pushing.

### 5.2 HTTP Streaming and DASH
- The video file is split into many small **segments** (typically 2–10 seconds each), each available at several bitrate versions.
- A **manifest file** (like an index) lists all available versions and their segment URLs.
- The client (video player):
  1. Downloads the manifest first.
  2. Measures current download speed while fetching segments.
  3. Requests the next segment at whichever bitrate best matches current bandwidth.
  4. Continuously repeats this — so quality can go up or down smoothly during playback, without stopping the video.
- Since it works over regular HTTP, it works through normal web infrastructure (proxies, caches, firewalls) without special server support.

### 5.3 Content Distribution Networks (CDNs)
- A CDN stores **copies (caches)** of content on many **edge servers** distributed across regions/countries.
- When a user requests a video, DNS-based or HTTP-based redirection sends them to the **nearest/best-performing edge server**, not the far-away origin.
- Two general CDN strategies:
  - **Enter deep**: Place many small server clusters deep inside access ISPs, very close to users (lower latency, more complex to manage).
  - **Bring home**: Place fewer, larger clusters at key network exchange points (simpler, still fairly fast).
- Benefits: lower latency, less load on origin server, better reliability, ability to handle huge traffic spikes.

---

## 6. Real example

When you watch a video on YouTube:
1. YouTube's player downloads a manifest listing available resolutions (144p–4K) and segment URLs.
2. If you're on Wi-Fi, it may start at 1080p; if your connection drops (e.g., walking out of Wi-Fi range to mobile data), the player automatically requests the next segments at 480p instead of buffering.
3. Because YouTube uses a global CDN (Google's own edge servers), your video segments are fetched from a server near your city, not from a single data center — that's why playback starts fast almost anywhere in the world.

---

## 7. Mini implementation idea (Termux/Python/Kotlin)

**See adaptive streaming manifests yourself:**
```bash
# Install ffmpeg in Termux
pkg install ffmpeg

# Inspect a video's streams/bitrates
ffprobe -v error -show_entries stream=bit_rate,width,height,codec_name -of json input.mp4
```

**Simulate simple bitrate-adaptive chunk selection in Python:**
```python
import random

qualities = {"1080p": 5000, "720p": 2500, "480p": 1000, "240p": 500}  # kbps required

def pick_quality(available_bandwidth_kbps):
    # Pick highest quality that fits under available bandwidth
    sorted_q = sorted(qualities.items(), key=lambda x: -x[1])
    for name, needed in sorted_q:
        if needed <= available_bandwidth_kbps:
            return name
    return "240p"

# Simulate changing network speed
for _ in range(5):
    bw = random.randint(300, 6000)
    print(f"Bandwidth: {bw} kbps -> Chosen quality: {pick_quality(bw)}")
```
This is a simplified mental model of what a real DASH player does every few seconds.

**CDN concept demo:** Try `dig youtube.com` or `dig netflix.com` from different networks (Wi-Fi vs mobile data) — you'll often get different IPs, showing DNS-based CDN redirection to a nearby edge server.

---

## 8. Common mistakes

- Thinking streaming means "downloading the whole file first" — adaptive streaming downloads small segments continuously, not the full file.
- Confusing CDN with just "a faster server" — it's actually a distributed network of many caching servers, chosen dynamically per user location.
- Assuming video quality is fixed once selected — DASH constantly re-evaluates and can switch mid-playback.
- Believing CDNs only work for video — they cache and speed up all kinds of static/dynamic content.
- Forgetting that DASH works over normal HTTP/HTTPS — no special streaming protocol or port is required.

## 9. Memory hook

Think of adaptive streaming like a **smart delivery driver** who checks the traffic (your bandwidth) before choosing which size package (video quality) to bring next. And think of a CDN like **local warehouses** in every city, so the driver never has to come all the way from one central factory — they grab the package from the nearest warehouse instead.

## 10. Ultra-short revision

Internet video is pre-encoded at multiple bitrates to handle varying network speeds. HTTP streaming with DASH breaks video into small segments described by a manifest file, letting the player pick the best quality segment based on real-time bandwidth — enabling smooth adaptive playback over plain HTTP. Since serving all users from one origin server would be slow and overloaded, CDNs cache content on edge servers spread globally, routing each user to a nearby server. This combination (adaptive streaming + CDN) is why modern video apps load fast and rarely buffer.

## 11. Interview / exam questions

1. **Q: What problem does adaptive bitrate streaming (DASH) solve?**
   A: It handles changing network conditions by letting the player dynamically choose video quality per segment instead of using one fixed quality.

2. **Q: What is a manifest file in DASH?**
   A: A file listing all available video quality versions and the URLs of their segments, which the player downloads first.

3. **Q: Why is DASH built on top of HTTP instead of a custom protocol?**
   A: Because HTTP works seamlessly with existing web infrastructure (firewalls, proxies, caches) without needing special server support.

4. **Q: What is a CDN and why is it needed?**
   A: A network of distributed edge servers that cache content close to users; needed because a single origin server can't efficiently serve a global audience with low latency.

5. **Q: What's the difference between "enter deep" and "bring home" CDN strategies?**
   A: "Enter deep" places many small server clusters inside ISP networks near users for lowest latency; "bring home" uses fewer, larger clusters at major network hubs for simpler management.

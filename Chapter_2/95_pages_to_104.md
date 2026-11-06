# 2.2.4–2.2.7: Cookies, Browser Caching, HTTP/2, HTTP/3 & QUIC

---

## 1. Simple meaning

- **Cookies (2.2.4):** A cookie is a small piece of data that a website asks your browser to store. The browser sends it back on every future request to that site, so the server can "remember" you.
- **Browser Caching (2.2.5):** Your browser saves copies of files (images, CSS, HTML) locally, so it doesn't have to download them again from the server every time.
- **HTTP/2 (2.2.6):** A faster version of HTTP. Instead of sending plain text messages, it breaks them into small binary pieces called **frames**, and can send many requests/responses at once over **one** connection.
- **HTTP/3 & QUIC (2.2.7):** The newest version of HTTP. It throws away TCP (the usual reliable transport) and instead runs on top of a new transport protocol called **QUIC**, which is built on UDP. This fixes delays that TCP was causing.

---

## 2. Why it matters

- **Cookies** power login sessions, shopping carts, "remember me," and ad tracking — almost every real web/Android app that talks to a backend uses cookies or cookie-like tokens.
- **Caching** is why apps and websites feel fast on the second visit — it's a core skill for backend/API design (cache headers) and Android networking (OkHttp cache).
- **HTTP/2 and HTTP/3** are what real production APIs, CDNs (Cloudflare, Google, Facebook), and mobile apps use today. If you build backend or Android apps, your HTTP client (OkHttp, Retrofit, Ktor) already negotiates these protocols — understanding them helps you debug slow network calls and design efficient APIs.
- This whole chapter connects to Chapter 3 (Transport Layer — TCP vs UDP) and Chapter 8 (Security — TLS), since QUIC merges transport + encryption.

---

## 3. Core idea

### A. Cookies
HTTP itself is **stateless** — the server forgets you after every request. Cookies add "state" on top of it.

- Server sends a `Set-Cookie` header in its response with a unique ID.
- Browser stores this ID and attaches it via the `Cookie` header on every future request to that domain.
- Server keeps a backend database mapping cookie ID → user info (login state, cart items, preferences).

### B. Browser Caching
To avoid re-downloading unchanged files:

- Server tells the browser **how long** a file is "fresh" using `Cache-Control` / `Expires` headers.
- While fresh, the browser uses the local copy — **zero network trip**.
- Once stale, the browser asks the server "has this changed?" using conditional requests (`If-Modified-Since` or `If-None-Match`). If unchanged, server replies `304 Not Modified` (no body sent — saves bandwidth).

### C. HTTP/2
Old HTTP/1.1 problem: only one request could be "in flight" per TCP connection at a time (or browsers opened multiple parallel TCP connections as a workaround) — this caused **Head-of-Line (HOL) blocking** at the application level and wasted resources.

HTTP/2 fixes this by:
- Turning every message into **binary frames** instead of plain text.
- **Multiplexing** many request/response streams over a **single TCP connection**.
- Letting the server **prioritize** which responses matter more.
- Letting the server **push** extra resources the client didn't explicitly ask for yet.

**HTTP/2 Framing:** Every HTTP/2 message is split into frames such as `HEADERS` (holds headers, compressed with HPACK) and `DATA` (holds the body). Each frame belongs to a numbered **stream**, so frames from different requests can interleave on the same TCP connection and get reassembled correctly at the other end.

**Prioritization & Server Push:**
- Each stream can be given a **weight** and a **dependency** on another stream, telling the server which responses to send first (e.g., HTML before images).
- **Server Push** lets the server proactively send resources (like CSS/JS) it knows the browser will need, using a `PUSH_PROMISE` frame, before the browser even asks — saving a round trip.
- ⚠️ Note: in practice, Server Push turned out to waste bandwidth (pushing things already cached) and major browsers (Chrome, from 2022) removed support for it. It's still worth knowing conceptually and for exams.

### D. HTTP/3 and QUIC
Even with HTTP/2's multiplexing, one problem remained: it all still runs over **one TCP connection**. If a single TCP packet is lost, **TCP itself blocks all streams** until that packet is retransmitted — this is **HOL blocking at the transport layer**, and HTTP/2 cannot fix it because TCP guarantees strict in-order delivery.

**Why a new transport protocol?**
- TCP is implemented deep inside operating system kernels and network hardware (middleboxes) — changing TCP's behavior across the whole internet is extremely slow.
- TCP + TLS also needs multiple round trips just to set up a secure connection before any data flows.
- So instead of modifying TCP, engineers built **QUIC**, a new protocol on top of **UDP** (which has no built-in ordering/reliability rules), implemented in **user space** (inside apps/libraries, not the OS kernel) — so it can be updated and improved quickly.

**The QUIC Transport Protocol — what it does:**
- Runs over UDP but adds its own reliability, ordering, and congestion control (things TCP normally provides).
- Combines the transport handshake and the TLS 1.3 encryption handshake into essentially **one round trip** (or **zero** extra round trips when reconnecting to a known server) — much faster connection setup than TCP+TLS.
- Supports multiple independent **streams** inside one QUIC connection — data loss on one stream does **not** block the others (fixes the HOL blocking problem for real).
- Supports **connection migration**: if your phone switches from Wi-Fi to mobile data, the QUIC connection survives (identified by a Connection ID, not IP+port like TCP).

**QUIC Services (what it provides to HTTP/3):**
- Reliable, in-order delivery **per stream**.
- Built-in, mandatory **encryption** (security is not optional, unlike TCP where TLS is a separate add-on layer).
- Fast connection establishment (1-RTT or 0-RTT).
- Congestion control and loss recovery, similar in spirit to TCP's.

**HTTP/3: Running the Web over QUIC:**
- HTTP/3 keeps the same HTTP semantics (GET, POST, headers, status codes) as HTTP/2.
- But instead of TCP streams, it maps HTTP requests/responses onto **QUIC streams**.
- Header compression is done with **QPACK** instead of HPACK (HPACK assumed strict order, which QUIC doesn't guarantee across streams).
- Net result: faster page loads, especially on unreliable networks like mobile, because one lost packet no longer stalls everything.

---

## 4. Key terms

| Term | One-line meaning |
|---|---|
| Cookie | Small data token stored by browser, sent back to identify the user on future requests |
| `Set-Cookie` header | Server → browser header that creates/updates a cookie |
| Session cookie | Cookie deleted when browser closes |
| Persistent cookie | Cookie with an expiry date, survives browser restarts |
| Cache-Control | Header that sets how long a response can be reused before re-checking |
| Conditional GET | A request asking "has this changed?" using `If-Modified-Since`/`If-None-Match` |
| 304 Not Modified | Server's reply meaning "your cached copy is still fine" |
| Frame | Smallest unit of HTTP/2 communication (binary chunk) |
| Stream | A virtual, bidirectional flow of frames representing one request/response inside a connection |
| Multiplexing | Sending multiple streams over one connection at the same time |
| HOL blocking | One stalled piece of data blocking everything behind it |
| Server Push | Server sends extra resources before client asks (deprecated in most browsers) |
| QUIC | New transport protocol over UDP combining transport + encryption, used by HTTP/3 |
| 0-RTT / 1-RTT | Zero/one network round trips needed before data can be sent |
| Connection migration | QUIC connection surviving an IP address change (e.g., Wi-Fi → mobile data) |
| QPACK | Header compression method used in HTTP/3 (QUIC-friendly version of HPACK) |

---

## 5. How it works

**Cookie flow:**
1. Browser sends first request to a site (no cookie yet).
2. Server responds and includes `Set-Cookie: sessionId=abc123`.
3. Browser stores it.
4. On every future request to that domain, browser automatically adds `Cookie: sessionId=abc123`.
5. Server looks up `abc123` in its database to know who you are.

**Caching flow:**
1. Browser requests a file.
2. Server responds with the file + `Cache-Control: max-age=3600`.
3. For the next hour, browser uses the local copy — no network request at all.
4. After it expires, browser sends a conditional GET.
5. Server replies `304 Not Modified` (fast, no body) or `200 OK` with the new file if it changed.

**HTTP/2 request flow:**
1. Browser opens one TCP connection to the server.
2. Browser opens multiple streams inside it (e.g., stream 1 for HTML, stream 3 for CSS, stream 5 for an image).
3. Server interleaves `HEADERS`/`DATA` frames from different streams on the same connection.
4. Browser reassembles frames per stream ID.
5. (Optionally) server pushes extra resources via `PUSH_PROMISE`.

**HTTP/3 / QUIC connection flow:**
1. Client and server perform a combined QUIC + TLS 1.3 handshake (1-RTT, or 0-RTT if reconnecting).
2. Client opens independent QUIC streams for each HTTP request.
3. Data flows per stream; if one packet is lost, only that stream pauses — others continue.
4. If the client's network changes (Wi-Fi → mobile), the Connection ID keeps the session alive.

---

## 6. Real example

- **Cookies + Caching together:** When you open Amazon, your login/cart is remembered via a cookie, while product images and the Amazon logo load instantly on repeat visits because they're cached with a long `max-age`.
- **HTTP/2:** Google, YouTube, and most CDNs serve pages over HTTP/2 by default — this is why a webpage with 50 images doesn't need 50 separate slow connections.
- **HTTP/3/QUIC:** YouTube and Google Search already serve over HTTP/3. On a shaky mobile network, HTTP/3 keeps loading smoothly because one dropped packet doesn't freeze the whole page — this is exactly the problem QUIC was designed to fix.

---

## 7. Mini implementation idea

You can explore all of this from your phone in Termux:

- **Cookies:** Use Python + `requests` in Termux:
  ```python
  import requests
  s = requests.Session()
  r = s.get("https://httpbin.org/cookies/set/testcookie/12345")
  print(s.cookies.get_dict())          # see stored cookie
  r2 = s.get("https://httpbin.org/cookies")
  print(r2.json())                      # server confirms it received the cookie
  ```
- **Caching:** Hit `https://httpbin.org/cache` twice with `requests` and inspect `Cache-Control`/`ETag` headers using `r.headers`.
- **HTTP/2 check:** Run `curl -I --http2 -v https://www.google.com` in Termux (install with `pkg install curl`) and look for `HTTP/2 200` in the output.
- **HTTP/3 check:** `curl -I --http3 -v https://cloudflare-quic.com` (needs a curl build with HTTP/3 support) to see the QUIC handshake succeed.
- **Kotlin/Android angle:** In an OkHttp client, log `response.protocol()` — you'll see it print `h2` or `h3` depending on what the server negotiated, which ties this theory directly to real Android networking code.

---

## 8. Common mistakes

- Thinking cookies are "sent by the server automatically forever" — they're actually re-sent by the **browser**, not pushed by the server.
- Confusing **caching** with **cookies** — caching stores files; cookies store identity/state. Different purpose, different headers.
- Thinking HTTP/2 multiplexing completely solves HOL blocking — it only solves it at the **application layer**; the underlying **TCP connection** can still block everything on packet loss. This exact gap is why HTTP/3 exists.
- Believing QUIC is just "HTTP/2 renamed" — QUIC is a **transport layer** change (replaces TCP with UDP-based QUIC), not just an application-layer tweak.
- Forgetting that QUIC's encryption is **mandatory**, unlike TCP where TLS is optional and separate.

---

## 9. Memory hook

- **Cookie** = a name tag the server pins on your browser so it recognizes you next time.
- **Caching** = keeping a photocopy at home so you don't walk to the library every time.
- **HTTP/2** = one wide highway with multiple lanes (streams) instead of separate narrow roads.
- **QUIC/HTTP/3** = each lane gets its own mini-highway (independent stream over UDP), so **one traffic jam (packet loss) doesn't block the other lanes** — unlike HTTP/2, where all lanes still share one TCP road.

---

## 10. Ultra-short revision

Cookies let a stateless HTTP protocol remember users across requests via `Set-Cookie`/`Cookie` headers. Browser caching avoids re-downloading unchanged files using `Cache-Control` and conditional GETs (`304 Not Modified`). HTTP/2 speeds things up by breaking messages into binary frames and multiplexing many streams over one TCP connection, with optional prioritization and (now largely deprecated) server push. But TCP still causes head-of-line blocking when a packet is lost. HTTP/3 solves this by replacing TCP with QUIC — a UDP-based transport with built-in encryption, fast 0/1-RTT handshakes, independent streams, and connection migration across networks.

---

## 11. Interview / exam questions

1. **Q: How does a cookie make HTTP "stateful" when HTTP itself is stateless?**
   A: The server sends a unique ID via `Set-Cookie`; the browser stores it and re-sends it on every future request via the `Cookie` header, letting the server match it to stored user data.

2. **Q: What is a conditional GET request, and why is it useful?**
   A: A request using `If-Modified-Since`/`If-None-Match` to ask if a cached file changed; if not, the server replies `304 Not Modified`, saving bandwidth and time.

3. **Q: What problem does HTTP/2 multiplexing solve, and what problem does it NOT solve?**
   A: It solves application-layer HOL blocking by running multiple streams over one TCP connection. It does NOT solve transport-layer HOL blocking, since TCP still delivers data strictly in order — one lost packet stalls all streams.

4. **Q: Why did engineers build QUIC on top of UDP instead of fixing TCP directly?**
   A: TCP is implemented in OS kernels and network hardware worldwide, making it very slow to change. Building QUIC in user space over UDP allows fast, independent updates without needing OS or middlebox changes.

5. **Q: What is connection migration in QUIC, and why does it matter for mobile devices?**
   A: QUIC identifies a connection by a Connection ID instead of IP+port, so switching networks (e.g., Wi-Fi to mobile data) doesn't break the connection — useful since phones change networks often.

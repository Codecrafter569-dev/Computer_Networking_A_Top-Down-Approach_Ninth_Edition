# The Network Core — Study Notes

## 1. Simple meaning

Computers don't send a whole message in one shot. Data gets broken into small pieces called **packets**, and these travel through a mesh of routers called the **network core**.

There are two ways to move data through this core:
- **Circuit switching** — reserve a dedicated path for your whole connection (old telephone system)
- **Packet switching** — break data into packets, send each independently, no reservation (how the internet works today)

The internet itself isn't one giant network — it's a **network of networks**: thousands of separate networks (ISPs, companies, colleges) all wired together.

## 2. Why it matters

- Every app you use — WhatsApp, YouTube, an API call — sends packets through this core. "Connection timeout," lag, and buffering all trace back to what happens here.
- Backend devs add retry logic, timeouts, and rate limits *because* packet switching gives no guarantees — packets can be delayed or dropped.
- `ping`, `traceroute`, load balancers, and CDNs all exist because of how forwarding and routing work.
- "Packet switching vs circuit switching" is a classic networking interview/exam question.

## 3. Core idea

**A) Packet switching**
Data splits into packets, each carrying a destination address in its header. Packets hop router to router, sharing links with other traffic — no dedicated path.

**B) Store-and-forward transmission**
A router doesn't relay a packet bit-by-bit as it arrives. It waits for the **entire packet**, checks it, then sends it out.
Delay for one link = **L/R** (L = packet size in bits, R = link rate in bits/sec).
For a path of N links (ignoring queuing/propagation) = **N × (L/R)**.

**C) Queuing delay & packet loss**
Each router has an output buffer per link. If packets arrive faster than the link can push them out, they wait — **queuing delay**. If the buffer fills up, new packets get dropped — **packet loss**. The busier the link, the worse both get.

**D) Forwarding tables & routing protocols**
A router uses a **forwarding table** (destination → outgoing link) to decide where each packet goes. This table is built and kept updated automatically by **routing protocols** (e.g., OSPF, BGP), which calculate the best paths across the whole network.

**E) Circuit switching (1.3.2)**
Before data moves, the network reserves resources (bandwidth, buffers) on every link along the path. You get a guaranteed, constant rate. Classic landline calls work this way.

**F) Multiplexing in circuit-switched networks**
A link's capacity is split among calls using:
- **FDM** — frequency range divided into bands, one band per call, for the whole call
- **TDM** — time divided into repeating frames of slots, one slot per call in every frame

**G) Packet switching vs circuit switching**
Circuit switching guarantees performance but wastes capacity — an idle reserved slot can't be borrowed. Packet switching shares links on demand (**statistical multiplexing**) — more efficient, supports more users, but no guarantees, so it needs congestion control to avoid overload.

**H) A network of networks (1.3.3)**
Your device connects to an **access ISP**. Access ISPs connect to bigger **regional** and **tier-1 ISPs** to reach the rest of the world. Networks physically meet and swap traffic at **IXPs** and **PoPs**. Big content providers (Google, Netflix) run their own private networks and connect close to access ISPs to cut delay.

**Connects to other chapters:** packet loss here is why the Transport layer (TCP) needs reliability; forwarding tables lead into Network-layer routing algorithms; this whole section sets up Chapter 1.4 (delay, loss, throughput).

## 4. Key terms

- **Packet** — a chunk of data with a header, sent through the network
- **Packet switching** — sending data as independent packets sharing links
- **Circuit switching** — reserving a dedicated path for the full connection
- **Store-and-forward** — router waits for the whole packet before sending it on
- **Transmission rate (R)** — link speed, in bits per second
- **Queuing delay** — time a packet waits in a router's buffer
- **Packet loss** — packets dropped when a buffer is full
- **Forwarding table** — router's map of destination → outgoing link
- **Routing protocol** — computes routes and fills forwarding tables (e.g., OSPF, BGP)
- **FDM** — splits a link by frequency bands
- **TDM** — splits a link by time slots
- **Access ISP** — connects end users to the internet
- **Tier-1 ISP** — top-level ISP with a global backbone
- **IXP** — Internet Exchange Point, where networks meet to swap traffic
- **PoP** — Point of Presence, routers where an ISP connects to others

## 5. How it works

**Packet switching journey:**
1. App breaks data into packets, each with a destination address in its header
2. First router receives the full packet before forwarding it (store-and-forward)
3. Router checks the header, looks up the destination in its forwarding table
4. Packet queues for the outgoing link if that link is busy
5. Router sends the packet once the link is free
6. Steps 2–5 repeat at every router until it reaches the destination
7. If a queue overflows anywhere, the packet is dropped — TCP detects this and resends it

**Circuit switching setup:**
1. Caller requests a connection
2. Network reserves bandwidth on every link along the path (via FDM/TDM)
3. Data flows at a fixed, guaranteed rate
4. When the call ends, resources are released for others

## 6. Real example

Sending a WhatsApp message or opening a website:
- Your phone breaks the request into packets and sends them to your Wi-Fi/mobile router
- The router forwards each packet through your ISP, possibly through an IXP, into the destination's network
- Different packets can take slightly different routes but still arrive and get reassembled correctly

Contrast: an old landline call used circuit switching — one dedicated line, reserved for that call, start to finish.

## 7. Mini implementation idea

**See real delay (Termux):**
```bash
ping -c 4 google.com
```
Shows round-trip delay to a real server. Want to see the actual hops (routers)?
```bash
pkg install traceroute
traceroute google.com
```
(Some Android setups restrict raw traceroute — if it fails, `ping` is your reliable fallback.)

**Simulate store-and-forward delay (Python):**

```python
def store_forward_delay(L_bits, R_bps, hops):
    return hops * (L_bits / R_bps)

print(store_forward_delay(8_000_000, 1_000_000, 3))  # seconds
```
Change `hops` or `R_bps` and watch the delay shift — builds intuition for why slow links or long paths mean more delay.

## 8. Common mistakes

- Thinking a router forwards bit-by-bit as data arrives — it waits for the full packet first (store-and-forward)
- Mixing up queuing delay (waiting in line) with packet loss (dropped because the line is full)
- Confusing **forwarding** (a router's local per-packet lookup) with **routing** (the network-wide process that builds the forwarding table)
- Assuming circuit switching is obsolete — it's just less efficient for bursty data, not extinct
- Thinking the internet is one single network instead of a network of networks

## 9. Memory hook

**Packets = postcards. Circuits = a private taxi.**
Postcards travel independently, queue up, and can get lost — but many people share the mail system cheaply. A private taxi is reserved just for you — guaranteed, but wasted if empty. And the internet is many separate road networks (ISPs) joined at interchanges (IXPs), not one single road.

## 10. Ultra-short revision

- Circuit switching reserves a dedicated path (guaranteed, wasteful); packet switching shares links on demand (efficient, no guarantee)
- Store-and-forward: router waits for the whole packet before sending it on; delay per link = L/R
- Queuing delay = waiting in a buffer; packet loss = buffer overflow drops the packet
- Forwarding table = local lookup at a router; routing protocol = builds that table network-wide
- FDM splits a circuit by frequency, TDM splits it by time slots
- The internet is a network of networks — access ISPs connect via regional/tier-1 ISPs and IXPs

## 11. Interview / exam questions

1. **What is store-and-forward transmission?**
A router must receive the entire packet before forwarding any of it; delay for one link = L/R.

2. **Difference between queuing delay and packet loss?**
Queuing delay is time waiting in a router's buffer; packet loss happens only when that buffer is full and the packet gets dropped.

3. **Forwarding table vs routing protocol?**
A forwarding table is a router's local lookup (destination → output link); a routing protocol computes routes network-wide and fills in that table.

4. **Circuit switching vs packet switching — core trade-off?**
Circuit switching reserves resources (via FDM/TDM) for guaranteed performance but wastes idle capacity; packet switching shares resources on demand — more efficient, but no delay/loss guarantee.

5. **What does "the internet is a network of networks" mean?**
It isn't one network — access ISPs connect to regional and tier-1 ISPs, and networks exchange traffic at IXPs, forming a huge interconnected mesh rather than a single system.
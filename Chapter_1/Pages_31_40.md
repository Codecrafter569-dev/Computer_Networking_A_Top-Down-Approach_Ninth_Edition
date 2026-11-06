# 1.4 Delay, Loss, and Throughput in Packet-Switched Networks

## 1. Simple meaning
When data travels from your phone to a server (like opening YouTube or sending a WhatsApp message), it doesn't arrive instantly. It takes time — this is called **delay**. Sometimes packets don't arrive at all — this is **loss**. And the network can only carry data at a certain speed — this is **throughput**. This topic explains *why* delay happens, *where* it happens, and how fast data can really move.

## 2. Why it matters
- **Real-time apps** (video calls, gaming, VoIP) break if delay is too high or unpredictable.
- **Backend/API design**: server response time = processing + network delay. Knowing this helps you debug "why is my API slow."
- **Android dev**: network timeouts, retry logic, and UX loaders are all designed around expected delay/throughput.
- **DevOps/networking**: tools like `ping`, `traceroute`, `mtr` are literally built on these concepts — you'll use them in Termux constantly.

## 3. Core idea

When a packet travels from source to destination, it passes through **routers**. At *every* router, the packet experiences **4 types of delay** before moving to the next link. Total delay = sum of these 4, repeated at each hop.

**Step by step at one router (node):**

1. **Processing delay** – router looks at the packet header, checks for bit errors, decides which outgoing link to send it on. Very small (microseconds).
2. **Queuing delay** – packet waits in a buffer/queue if the router is busy sending other packets first. This is the *most variable* delay — depends on network congestion.
3. **Transmission delay** – time to push *all bits* of the packet onto the link. Depends on packet size and link speed: `L / R` (L = packet length in bits, R = link rate in bits/sec).
4. **Propagation delay** – time for a bit to physically travel across the link (through cable/fiber/air) to reach the other end: `d / s` (d = distance, s = propagation speed, ~2×10⁸ m/s in cable).

This repeats at every router until the packet reaches the destination → giving **end-to-end delay**.

## 4. Key terms

| Term | Meaning |
|---|---|
| **Nodal delay** | Total delay at one router = processing + queuing + transmission + propagation |
| **Processing delay (dproc)** | Time to inspect header & route the packet |
| **Queuing delay (dqueue)** | Time packet waits in buffer before transmission |
| **Transmission delay (dtrans)** | Time to push all packet bits onto the link = L/R |
| **Propagation delay (dprop)** | Time for a bit to travel the physical link = d/s |
| **Traffic intensity (La/R)** | Ratio showing how "busy" a link is; a = packet arrival rate, L = packet size, R = link rate |
| **Packet loss** | Packet is dropped because the router's buffer is full |
| **End-to-end delay** | Total delay from source to destination across all hops |
| **Traceroute** | Tool that measures delay to each router along a path |
| **Throughput** | Actual rate (bits/sec) at which data is transferred |
| **Bottleneck link** | The slowest link along the path — it caps overall throughput |

## 5. How it works

**Queuing delay & loss:**
- Each router has a **finite buffer** for each outgoing link.
- If a packet arrives and the buffer is full → it's **dropped** (packet loss). Lost packets may be retransmitted (by TCP) or just lost (UDP).
- Queuing delay depends on **traffic intensity = La/R**:
  - If La/R ≈ 0 → almost no queuing delay.
  - If La/R → 1 → queuing delay grows huge (approaches infinity in theory).
  - If La/R > 1 → traffic arrives faster than it can leave → buffer overflows → guaranteed loss.

**End-to-end delay (for N links, ideal equal case):**
```
d_end-to-end = N × (d_proc + d_trans + d_prop)
```
(Real networks aren't equal at every hop, so it's really a sum of each hop's individual delays.)

**Traceroute — how it works:**
- Sends packets toward destination with increasing **TTL (Time To Live)** values (1, 2, 3…).
- Each router that receives a packet whose TTL expires sends back an error message.
- This reveals the identity and round-trip delay to **each router along the path** — great for diagnosing where delay/loss is happening.

**Throughput:**
- **Instantaneous throughput** = rate at a given moment.
- **Average throughput** = total bits / total time.
- For a path with multiple links of different speeds, throughput = **minimum link rate along the path** (the bottleneck link decides the max possible speed).

## 6. Real example
You download a file from a server. Your Wi-Fi link is 100 Mbps, but the server's uplink is only 10 Mbps. Even though your Wi-Fi is fast, your **download throughput will cap at 10 Mbps** — the bottleneck link. Run `traceroute google.com` and you'll see each router hop with its delay in milliseconds — some hops jump high because of queuing delay at a congested router.

## 7. Mini implementation idea (Termux-friendly)
- **Try it live:** In Termux, run:
  ```
  ping -c 10 google.com      # measure RTT, see delay variation
  traceroute google.com      # or install mtr: mtr google.com
  ```
- **Simulate throughput/download:**
  ```python
  import time, requests
  start = time.time()
  r = requests.get("https://speed.hetzner.de/100MB.bin", stream=True)
  size = sum(len(chunk) for chunk in r.iter_content(1024))
  elapsed = time.time() - start
  print(f"Throughput: {size*8/elapsed/1_000_000:.2f} Mbps")
  ```
- **Simulate a queue (Kotlin/Python):** Model a router buffer as a `Queue`, feed packets in at rate `a`, drain at rate `R`. Track queue length over time as `La/R` approaches 1 — you'll see delay explode. Great small project to *feel* the theory.

## 8. Common mistakes
- ❌ Thinking **transmission delay** and **propagation delay** are the same — transmission is about *pushing bits out* (depends on packet size & link speed); propagation is about *bits traveling distance* (depends on distance & medium).
- ❌ Confusing **bandwidth** with **throughput** — bandwidth is the link's max capacity; throughput is what you *actually* get (limited by bottleneck, congestion, etc.).
- ❌ Assuming queuing delay is constant — it's highly variable and depends on real-time traffic.
- ❌ Forgetting that traffic intensity must stay **below 1**, or queuing delay/loss becomes unmanageable.

## 9. Memory hook
**"PQTP"** = **P**rocess → **Q**ueue → **T**ransmit → **P**ropagate (order at every router).
Think of a **toll booth on a highway**:
- Checking your ticket = processing delay
- Waiting in line at the booth = queuing delay
- Time for your whole car to pass through the booth = transmission delay
- Driving the highway distance to the next booth = propagation delay

## 10. Ultra-short revision
- Every router adds 4 delays: **processing, queuing, transmission, propagation**.
- Queuing delay is the variable one — depends on traffic intensity (La/R); if ≥1, delay/loss explodes.
- Packet loss happens when a router's buffer is full.
- End-to-end delay = sum of nodal delays across all hops.
- Traceroute uses increasing TTL to map delay at each hop.
- Throughput = actual data rate achieved; capped by the **bottleneck link** (minimum rate on the path).

## 11. Interview / exam questions

1. **Q: What are the four types of delay a packet experiences at a router?**
   A: Processing delay, queuing delay, transmission delay, propagation delay.

2. **Q: What's the difference between transmission delay and propagation delay?**
   A: Transmission delay = time to push all bits onto the link (L/R). Propagation delay = time for one bit to travel the physical distance (d/s).

3. **Q: What happens when traffic intensity (La/R) exceeds 1?**
   A: Packets arrive faster than they can be sent out, so the queue grows unboundedly and packet loss occurs.

4. **Q: How does traceroute measure delay to intermediate routers?**
   A: It sends packets with increasing TTL values; each router where TTL expires sends back an error, revealing its identity and round-trip delay.

5. **Q: How is throughput determined on a path with multiple links of different speeds?**
   A: Throughput equals the rate of the slowest link (bottleneck link) along the path.
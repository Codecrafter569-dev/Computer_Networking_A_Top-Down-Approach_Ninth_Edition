# 🌐 1.3.3 Network of Networks + 1.4 Delay, Loss & Throughput

---

## 1. Simple Meaning

**Network of Networks** = the internet isn't one giant network — it's millions of smaller networks (ISPs, companies, universities) all interconnected, forming a huge "network of networks."

**Delay, Loss, and Throughput** = every packet traveling from sender to receiver faces **four types of delay**, may get **dropped (lost)**, and the actual data rate delivered is called **throughput**. Understanding these explains why the internet is sometimes fast, sometimes slow, and sometimes drops your data.

---

## 2. Why It Matters

- Explains real-world app behavior: why API calls are slow, why video calls buffer, why some requests time out.
- Backend engineers need to understand **queuing delay** and **packet loss** to design systems that handle network congestion (retries, timeouts, load balancing).
- ISPs, CDNs (like Cloudflare), and cloud providers are built around this hierarchical "network of networks" structure — important for understanding latency in distributed systems.
- Helps you reason about **why fiber optic backbone + local Wi-Fi** together determine your app's real-world speed.

---

## 3. Core Idea (Step by Step)

### A. Network of Networks

The internet has a **hierarchical structure**:
1. **Access ISPs** (your home/mobile provider) connect end systems to the internet.
2. Access ISPs connect to **regional/tier-1 ISPs** — huge backbone providers with global fiber networks.
3. Tier-1 ISPs interconnect directly with each other (**peering**) so traffic doesn't need a single central authority.
4. **Points of Presence (PoPs)**, **multi-homing**, **peering**, and **Internet Exchange Points (IXPs)** all help data find efficient paths across this mesh.
5. No single entity owns the internet — it's a cooperation of thousands of independently operated networks.

### B. Delay in Packet-Switched Networks

When a packet travels from source to destination, it passes through multiple routers. At **each router**, it experiences **four types of delay**:

1. **Processing Delay** — time for a router to examine the packet's header and decide where to forward it (check for bit errors, determine output link). Usually microseconds.

2. **Queuing Delay** — time the packet waits in the router's output buffer before it can be transmitted, because other packets are ahead of it. Depends on **congestion level** — highly variable.

3. **Transmission Delay** — time to push all the packet's bits onto the link. Formula: 
   **Transmission Delay = Packet Size (L) / Link Bandwidth (R)** = L/R

4. **Propagation Delay** — time for a bit to physically travel from one end of the link to the other. Formula: 
   **Propagation Delay = Distance (d) / Propagation Speed (s)** = d/s

**Total nodal delay** = Processing + Queuing + Transmission + Propagation.

### C. Transmission vs Propagation Delay (Important Distinction)

- **Transmission delay**: depends on **packet size and link speed** — how long to "push" the packet out.
- **Propagation delay**: depends on **physical distance and medium** — how long for the signal to travel through the wire/fiber/air.
- Analogy: transmission delay is like the time to load a truck; propagation delay is the time the truck drives to the destination.

### D. Queuing Delay and Packet Loss

- Queuing delay depends on **traffic intensity = (arrival rate × packet size) / link bandwidth** = La/R.
- If **La/R is close to 0**: little queuing.
- If **La/R approaches 1**: queuing delay grows very large (theoretically infinite).
- If **La/R > 1**: queue keeps growing infinitely — meaning the router receives packets faster than it can send them.
- **Packet Loss** happens when a router's buffer is **full** — arriving packets are simply **dropped**, since there's no room to store them. Lost packets may be retransmitted by the sender (depending on protocol — e.g., TCP retransmits, UDP doesn't).

---

## 4. Key Terms

| Term | Meaning |
|---|---|
| **Access ISP** | Provider connecting end users to the internet |
| **Tier-1 ISP** | Global backbone ISP interconnecting other ISPs |
| **Peering** | Direct interconnection agreement between ISPs |
| **IXP (Internet Exchange Point)** | Physical location where multiple ISPs connect and exchange traffic |
| **Processing Delay** | Time to examine packet header and decide routing |
| **Queuing Delay** | Time packet waits in buffer before transmission |
| **Transmission Delay** | Time to push all packet bits onto the link (L/R) |
| **Propagation Delay** | Time for a bit to travel across the physical link (d/s) |
| **Nodal Delay** | Total delay at one router = sum of all 4 delay types |
| **Traffic Intensity (La/R)** | Ratio measuring how congested a link is |
| **Packet Loss** | Packet dropped due to full buffer at a router |
| **Throughput** | Actual rate of data successfully delivered end-to-end |

---

## 5. How It Works

**Delay flow at each router (per hop):**
1. Packet arrives at router's input link.
2. Router **processes** the packet (check header, decide output link) → processing delay.
3. Packet joins the **output queue** if the link is busy → queuing delay (variable, depends on congestion).
4. Once it's packet's turn, router **transmits** all bits onto the outgoing link → transmission delay = L/R.
5. Bits **propagate** through the physical medium to the next router → propagation delay = d/s.
6. This entire process repeats at every router until the packet reaches its destination.

**Packet loss flow:**
1. Packets arrive at a router faster than they can be transmitted.
2. The output buffer starts filling up.
3. If buffer is full, new arriving packets are **dropped** (lost).
4. Depending on transport protocol: TCP detects loss and retransmits; UDP does not, so lost packets are just gone.

---

## 6. Real Example

- When you load a webpage, your request passes through your home ISP → regional ISP → tier-1 backbone → destination's ISP → server, each hop adding processing + queuing + transmission + propagation delay.
- **Video call lag/freeze**: during high network congestion, queuing delay increases sharply (as La/R approaches 1), and if buffers overflow, packets are **lost**, causing visible frame drops.
- **Streaming (Netflix, YouTube)**: throughput determines the actual video quality you get, which is often **lower than your ISP's advertised speed** due to congestion at intermediate routers.

---

## 7. Mini Implementation Idea (Termux)

Simulate transmission vs propagation delay with Python:

```python
def transmission_delay(packet_size_bits, bandwidth_bps):
    return packet_size_bits / bandwidth_bps

def propagation_delay(distance_m, prop_speed_mps=2e8):  # ~2x10^8 m/s in copper/fiber
    return distance_m / prop_speed_mps

L = 1000 * 8       # 1000 bytes packet in bits
R = 1_000_000       # 1 Mbps link
d = 5000 * 1000     # 5000 km distance in meters

t_trans = transmission_delay(L, R)
t_prop = propagation_delay(d)

print(f"Transmission Delay: {t_trans*1000:.3f} ms")
print(f"Propagation Delay: {t_prop*1000:.3f} ms")
print(f"Total Delay (approx): {(t_trans+t_prop)*1000:.3f} ms")
```

Try changing `R` (bandwidth) and `d` (distance) to see which delay dominates — this builds intuition for the "comparing transmission and propagation delay" concept.

You can also use `ping` and `traceroute` in Termux (`pkg install inetutils traceroute`) to see **real propagation + queuing delay** to different servers.

---

## 8. Common Mistakes

- Thinking transmission delay and propagation delay are the same — they depend on **completely different factors** (link speed vs distance).
- Forgetting that **queuing delay is the most variable** delay — it depends on network traffic, not fixed link properties.
- Assuming packet loss only happens due to bad connections — it mainly happens due to **buffer overflow from congestion**.
- Confusing **throughput** with **bandwidth** — bandwidth is the link's maximum capacity; throughput is the actual achieved rate (often lower).
- Thinking traffic intensity (La/R) can safely be close to or above 1 — this causes delay to blow up or guarantees packet loss.

---

## 9. Memory Hook

🚚 **"Load the truck, then drive it"**:
- **Transmission delay** = time to **load** the truck (packet) with goods (bits).
- **Propagation delay** = time to **drive** the truck to the destination.

🪣 **Queuing delay = bucket filling up** — if water (packets) comes in faster than it drains (transmission), the bucket overflows → **packet loss**.

**4 Delays acronym: "PQTP"** → **P**rocessing, **Q**ueuing, **T**ransmission, **P**ropagation (in order at each hop).

---

## 10. Ultra-Short Revision

- Internet = hierarchical "network of networks" (access ISPs → tier-1 ISPs → peering/IXPs).
- Each router hop adds 4 delays: **Processing + Queuing + Transmission + Propagation**.
- Transmission delay = L/R (packet size ÷ bandwidth); Propagation delay = d/s (distance ÷ speed).
- Queuing delay depends on traffic intensity (La/R) — grows sharply as intensity nears 1.
- Packet loss happens when a router's buffer overflows due to congestion.
- Throughput = actual delivered data rate, often less than the link's max bandwidth.

---

## 11. Interview / Exam Questions

1. **Q: What are the four types of delay a packet experiences at each router?**
   A: Processing delay, queuing delay, transmission delay, and propagation delay.

2. **Q: What is the formula for transmission delay and propagation delay?**
   A: Transmission delay = L/R (packet size ÷ bandwidth); Propagation delay = d/s (distance ÷ propagation speed).

3. **Q: Why does queuing delay vary so much compared to other delays?**
   A: Because it depends on network traffic/congestion at that moment, not on fixed link properties like distance or bandwidth.

4. **Q: What causes packet loss in a network?**
   A: When a router's output buffer is full and cannot store any more incoming packets, they are dropped.

5. **Q: What is traffic intensity, and why is a value close to 1 problematic?**
   A: Traffic intensity (La/R) measures how congested a link is; as it approaches 1, queuing delay grows very large, and above 1 the queue grows unbounded, causing severe delay and loss.
# 📡 Physical Media + 1.3 The Network Core

---

## 1. Simple Meaning

**Physical Media** = the actual physical "roads" that bits travel on — wires, cables, glass fibers, or air (radio waves).

**The Network Core** = the mesh of routers and links inside the internet that connects all end systems together. Data can travel through this core in two ways: **packet switching** (chop data into small packets, send them independently) or **circuit switching** (reserve a fixed path for the whole call, like old telephones).

---

## 2. Why It Matters

- Every app you build (Android, backend, API) ultimately sends bits over some physical medium — knowing the medium's speed/limits helps you understand latency and bandwidth issues.
- Packet switching is **the reason the internet works the way it does** — HTTP, TCP, your REST APIs, all ride on packet-switched networks.
- Understanding queuing delay and packet loss explains why your app sometimes lags, times out, or drops requests under load — directly relevant to backend performance tuning.
- Circuit vs packet switching concepts reappear in VoIP, telecom backend systems, and QoS design.

---

## 3. Core Idea (Step by Step)

### A. Physical Media

**Guided media** (signal travels in solid medium):
1. **Twisted-Pair Copper Wire** — two insulated copper wires twisted together to cancel electrical interference. Cheapest, used in Ethernet (Cat5e/Cat6) and telephone lines.
2. **Coaxial Cable** — one copper core wire surrounded by insulation, metallic shield, then plastic jacket. Used in cable TV/internet. Can act as a **shared broadcast medium**.
3. **Fiber Optics** — thin glass strands carrying light pulses instead of electricity. Immune to electrical noise, extremely high speed, used for long-distance backbones and undersea cables.

**Unguided media** (signal travels through air/space, no physical wire):
4. **Terrestrial Radio Channels** — connect Wi-Fi routers, cellular towers, ground-based devices. Medium range, affected by obstacles and interference.
5. **Satellite Radio Channels** — signal bounces off satellites in orbit. Useful for remote areas, but has **high propagation delay** (signal must travel to space and back).

### B. The Network Core

The network core is not just wires — it's a mesh of **packet switches (routers)** connected by **communication links**.

**Two switching approaches:**

**1. Packet Switching**
- Data is broken into **packets** (small chunks with headers).
- Each packet is sent independently through the network — **store-and-forward**.
- At each router: the packet is fully received, checked, then forwarded to the next hop.
- Routers use **forwarding tables** (built by **routing protocols**) to decide which output link to send a packet to.

**2. Circuit Switching**
- Before data is sent, a **dedicated circuit (path)** is reserved end-to-end for the entire session (like a phone call).
- Resources (bandwidth/time slots) are reserved in advance — **guaranteed performance**, but wasted if idle.
- Achieved via **multiplexing**: FDM (Frequency Division) or TDM (Time Division), splitting a link's capacity into separate chunks per connection.

**Packet Switching vs Circuit Switching** — the biggest architectural debate in networking:
- Packet switching = efficient sharing, but unpredictable delay (best-effort).
- Circuit switching = guaranteed quality, but wastes capacity during idle time.
- The internet chose **packet switching** for its efficiency and scalability.

---

## 4. Key Terms

| Term | Meaning |
|---|---|
| **Twisted-Pair** | Two twisted copper wires; cheap, used in Ethernet/telephone |
| **Coaxial Cable** | Shielded copper cable; used in cable internet/TV |
| **Fiber Optics** | Glass fiber carrying light; fastest, no electrical interference |
| **Terrestrial Radio** | Ground-based wireless signal (Wi-Fi, cellular) |
| **Satellite Radio** | Signal relayed via orbiting satellite; high delay |
| **Packet Switching** | Data split into packets, routed independently |
| **Store-and-Forward** | Router receives entire packet before forwarding it |
| **Queuing Delay** | Time a packet waits in a router's buffer before transmission |
| **Packet Loss** | Packet dropped because router's buffer is full |
| **Forwarding Table** | Table in router mapping destination → outgoing link |
| **Routing Protocol** | Algorithm/protocol that builds forwarding tables (e.g., OSPF, BGP) |
| **Circuit Switching** | Dedicated end-to-end path reserved before data transfer |
| **Multiplexing (FDM/TDM)** | Splitting one link's capacity among multiple connections |
| **Propagation Delay** | Time for signal to physically travel through the medium |

---

## 5. How It Works

**Packet Switching flow:**
1. Sender breaks message into packets, each with a header (destination info).
2. First packet arrives at Router A.
3. Router A **stores the entire packet** in memory (store-and-forward), checks header.
4. Router A looks up its **forwarding table** → decides output link.
5. Packet is forwarded to Router B; process repeats hop-by-hop.
6. If a router's outgoing link is busy, packet waits in a **queue** → causes **queuing delay**.
7. If the queue (buffer) is full, the packet is **dropped** → packet loss.
8. Forwarding tables are updated dynamically by **routing protocols** based on network topology changes.

**Circuit Switching flow:**
1. Before any data flows, the network reserves a path end-to-end (call setup).
2. Bandwidth is divided via **FDM** (separate frequency bands) or **TDM** (separate time slots).
3. Data flows at guaranteed rate for the whole session.
4. Circuit is released when the call/session ends.

---

## 6. Real Example

- **Fiber optics**: Undersea fiber cables carry almost all international internet traffic (used by AWS, Google, Netflix backbones).
- **Packet switching**: When you send a WhatsApp message, it's broken into IP packets, each routed independently through different paths, and reassembled at the destination.
- **Circuit switching**: Traditional landline phone calls — a dedicated line is reserved for your call duration, even during silence.
- **Queuing delay/loss**: Video call lag/freeze during peak network congestion is queuing delay and packet loss in action.

---

## 7. Mini Implementation Idea (Termux)

You can **simulate store-and-forward and queuing delay** conceptually with Python in Termux:

```python
import time, queue, random

router_buffer = queue.Queue(maxsize=3)  # buffer size = 3 packets

def send_packet(packet_id):
    if router_buffer.full():
        print(f"Packet {packet_id} DROPPED (buffer full)")
        return
    router_buffer.put(packet_id)
    print(f"Packet {packet_id} queued")

def forward_packets():
    while not router_buffer.empty():
        pkt = router_buffer.get()
        delay = random.uniform(0.1, 0.5)
        time.sleep(delay)  # simulate transmission delay
        print(f"Packet {pkt} forwarded after {delay:.2f}s")

for i in range(1, 6):
    send_packet(i)
forward_packets()
```

This models buffer limits, queuing, and packet drops — the core intuition behind 1.3.1.

You can also use `traceroute` in Termux (`pkg install traceroute`) to see real hop-by-hop packet switching in action.

---

## 8. Common Mistakes

- Thinking **coaxial cable is obsolete** — it's still widely used in cable broadband.
- Confusing **propagation delay** (medium/distance-based) with **queuing delay** (congestion-based) — they're different delay types (connects to earlier topic 1.4).
- Assuming **packet switching is always better** — circuit switching still wins for guaranteed real-time quality (some VoIP/telecom systems use circuit-like reservation).
- Thinking **store-and-forward** means "processing" — it just means the **entire packet must arrive** before any part is forwarded.
- Mixing up **FDM vs TDM** — FDM = separate frequency slices; TDM = separate time slices, same frequency.

---

## 9. Memory Hook

🖇️ **"Store fully, then forward"** — a router is like a **mail sorting office**: the whole letter (packet) must arrive before it's placed in the next outgoing bag.

📞 **Circuit switching = reserved lane on a highway** (guaranteed but can go empty).
📦 **Packet switching = shared highway, cars merge as needed** (efficient but can jam).

---

## 10. Ultra-Short Revision

- Physical media = wires (twisted-pair, coax, fiber) or wireless (terrestrial/satellite radio) carrying bits.
- Network core = mesh of routers + links connecting all end systems.
- Packet switching: data → packets → store-and-forward → forwarding tables decide next hop.
- Congestion causes **queuing delay**; full buffers cause **packet loss**.
- Circuit switching: dedicated path reserved via FDM/TDM before data flows.
- Internet uses packet switching for efficiency; telephone networks traditionally used circuit switching.

---

## 11. Interview / Exam Questions

1. **Q: What is store-and-forward transmission?**
   A: A router must receive the entire packet before forwarding any part of it to the next link.

2. **Q: Why does packet switching cause queuing delay but circuit switching doesn't?**
   A: Packet switching shares link capacity on demand (packets can wait in queues); circuit switching reserves fixed capacity in advance, so no waiting occurs once the circuit is set up.

3. **Q: What's the difference between FDM and TDM?**
   A: FDM splits a link's bandwidth into separate frequency bands per connection; TDM splits it into separate time slots per connection.

4. **Q: Why did the internet choose packet switching over circuit switching?**
   A: Packet switching allows efficient, flexible sharing of network resources among many users, unlike circuit switching which reserves resources even when idle.

5. **Q: What determines a router's forwarding decision?**
   A: Its forwarding table, which maps destination address ranges to outgoing links, built and updated by routing protocols.
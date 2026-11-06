# Access Networks, Physical Media & Packet Switching

---

## 1. Simple Meaning

Before data can travel across the internet, your device needs a way to **get connected to the internet in the first place**. That connection is called the **access network** — it's the "first mile" of your internet journey, the link between your device and your ISP.

Once connected, data travels through **physical media** — the actual wires, cables, or wireless signals that carry information from one place to another.

And once your data enters the internet's core, it moves using **packet switching** — a technique where data is broken into small chunks called packets, which travel independently through a network of routers until they reach the destination.

---

## 2. Why It Matters

- As an **Android developer**, when your app makes an API call, it starts through an access network (Wi-Fi or 4G/5G). Understanding why some connections are faster or more reliable helps you design apps that handle slow networks gracefully.
- As a **backend developer**, you host servers in data centers that communicate with clients across different access networks. Knowing latency differences between DSL vs fiber vs 5G helps you design timeout values and retry logic.
- **Packet switching** is the fundamental reason the internet scales to billions of users — every backend request/response you've ever made used it.

---

## 3. Core Idea

### 3.1 Access Networks — Getting Connected

An access network is what physically connects an end system (your phone, laptop) to the first router in your internet journey. Different types exist based on location, speed, and available infrastructure.

**Home Access:**

- **DSL (Digital Subscriber Line):** Uses existing **telephone copper wires** to carry internet data. Works simultaneously with voice calls on the same wire by using different frequency bands. Typically offers lower speeds (a few Mbps up to ~50 Mbps) and is limited by distance from the ISP's telephone exchange.
- **Cable Internet:** Uses the same **coaxial cable** that delivers your cable TV. Can deliver higher speeds than DSL (tens to hundreds of Mbps) but the bandwidth is **shared** among neighbors in the same cable segment — so speeds drop during peak hours. Uses a standard called HFC (Hybrid Fiber-Coaxial).
- **FTTH (Fiber to the Home):** Optical fiber runs directly from the ISP to your home. This is the fastest home access technology, capable of gigabit speeds, but it's expensive to install.
- **Fixed Wireless:** Uses radio signals (often from a nearby tower) to deliver internet to a fixed location (home/office). No cable is needed. Good for rural areas.
- **LEO Satellites (Low Earth Orbit):** Satellites orbit at low altitudes (~550km), dramatically reducing the signal delay compared to traditional satellites. SpaceX Starlink is the key example — useful for extremely remote areas with no wired infrastructure.

**Enterprise and Campus:**

- **Ethernet:** Wired connection using twisted-pair copper cables. Extremely fast (100 Mbps to 10 Gbps) and reliable. Standard in offices, universities, and data centers.
- **WiFi (802.11):** Wireless LAN using radio frequencies. Devices connect to a wireless access point (AP), which itself is typically wired via Ethernet to the broader network. Convenient but affected by interference and walls.

**Wide-Area Wireless:**

- **4G LTE:** Offers typical download speeds of 10–50 Mbps. Widely deployed globally.
- **5G:** Fifth-generation mobile network. Offers speeds potentially in the Gbps range with very low latency. Still rolling out globally.

---

### 3.2 Physical Media — What Carries the Data

Physical media is the actual "stuff" that carries the signal — it's the road the data travels on.

- **Twisted-Pair Copper Wire:** Two insulated copper wires twisted around each other to reduce interference. Very common (telephone lines, Ethernet cables). Cheapest option. Categories like Cat5e, Cat6 define speed capability.
- **Coaxial Cable:** A central copper conductor surrounded by shielding. Higher bandwidth than twisted pair. Used in cable TV and cable internet (HFC networks).
- **Fiber Optic Cable:** Uses pulses of light instead of electrical signals. Extremely fast, very low signal loss, immune to electromagnetic interference. Used for backbone internet links and FTTH. Expensive to install but best performing.
- **Terrestrial Radio Channels:** Wireless signals through the atmosphere. Affected by environmental interference, buildings, and distance. Used in WiFi, 4G, 5G, Bluetooth.
- **Satellite Radio Channels:** Data sent to/from satellites in orbit. Traditional geostationary satellites (GEO) orbit at ~36,000 km, causing high latency (~280ms delay). LEO satellites orbit much closer (~550 km), reducing latency significantly (~20–40ms).

---

### 3.3 Packet Switching — How Data Travels Through the Core

Once your data enters the internet's core, it travels via **packet switching**.

Data is broken into small units called **packets**. Each packet has a **header** (containing destination address and other info) and a **payload** (the actual data). Packets are sent independently through routers, and each router decides the next hop based on the header.

**Store-and-Forward Transmission:**
This is a critical concept. A packet switch (router) must receive the **entire packet completely** before it can begin forwarding it to the next link. So the packet is:

1. Received fully at the router
2. Stored briefly in memory (buffer)
3. Then forwarded out to the next link

This means there's a delay at each hop equal to the time it takes the entire packet to arrive. If packet size is L bits and link speed is R bits/second, the store-and-forward delay at one hop is **L/R seconds**. For N links (N-1 routers), total delay = **N × L/R**.

---

## 4. Key Terms

| Term | Meaning |
|---|---|
| Access Network | The link connecting an end system to the first router of the internet |
| DSL | Internet over telephone copper lines; uses different frequencies for data and voice |
| Cable Internet (HFC) | Internet over coaxial cable TV lines; bandwidth shared among local users |
| FTTH | Fiber optic cable run directly from ISP to home; fastest home connection |
| Fixed Wireless | Internet via radio signal to a fixed location; no cable needed |
| LEO Satellite | Low-orbit satellite internet (like Starlink) with lower latency than old satellites |
| Ethernet | Wired LAN connection using twisted-pair cable; fast and reliable |
| WiFi (802.11) | Wireless LAN using radio frequencies; connects to a wired access point |
| 4G/5G | Wide-area mobile wireless; 5G offers far greater speed and lower latency |
| Twisted-Pair Copper | Cheapest wired medium; two twisted wires used in phone lines and Ethernet |
| Coaxial Cable | Shielded copper cable with higher bandwidth; used in cable TV/internet |
| Fiber Optic | Light-based data transmission; fastest, lowest loss, immune to interference |
| Terrestrial Radio | Wireless signals through air; used in WiFi, 4G, 5G |
| GEO Satellite | High-orbit satellite (~36,000 km); high latency (~280ms) |
| Packet | A small chunk of data with a header (routing info) and payload (actual data) |
| Packet Switching | Sending independent packets through a network of routers using headers |
| Store-and-Forward | Routers receive a full packet before forwarding it; adds L/R delay per hop |
| Header | The control part of a packet containing destination address and metadata |
| Router | A packet switch in the network core that forwards packets toward their destination |
| Bandwidth / Throughput | The rate at which data is transmitted on a link (in bits per second) |

---

## 5. How It Works

**Access → Core → Destination (full flow):**

1. Your device (phone/laptop) connects to an **access network** — Wi-Fi, mobile 4G/5G, DSL, cable, or fiber.
2. The access network links your device to your **ISP's first router** — this is where your device "enters" the internet.
3. Your application data is broken into **packets**, each with a header (destination IP, etc.) and payload.
4. Each packet enters the **network core** — a mesh of routers connected by high-speed links.
5. At each router, **store-and-forward** happens: the full packet arrives → is stored in a buffer → router reads the header → decides the next hop → forwards it.
6. This hop-by-hop process repeats across multiple routers until the packet reaches the destination's access network.
7. The destination reassembles the packets back into the original data (handled by TCP, covered in later chapters).
8. A response is sent back via the same packet-switching process.

**Physical media carries the signal at every step** — twisted-pair or fiber between your home and the ISP, then fiber and coaxial cables in the ISP's own network, then fiber for backbone links.

---

## 6. Real Example

When you open YouTube on your phone over 4G:

- Your phone sends a request through the **4G access network** (a terrestrial radio channel to a cell tower, then fiber from the tower to your ISP).
- The request is broken into **packets** and enters the internet's core.
- Each packet travels hop-by-hop through routers via **store-and-forward** — stored, examined, forwarded at each router.
- The packets reach Google's data center servers (likely connected via **fiber optic** backbone links).
- The video data is split into thousands of packets, sent back through the same packet-switching process.
- Your phone buffers and plays the video. If your 4G signal drops (poor radio channel), packets are delayed or lost, causing buffering.

---

## 7. Mini Implementation Idea

**In Termux — trace the packet path and measure delays:**

```bash
pkg install traceroute
traceroute youtube.com
```

Each numbered line is one **router hop**. The time shown (in ms) is the round-trip delay at that hop — you can literally see store-and-forward delays accumulating. Try also:

```bash
# Check your access type's speed effect
ping google.com          # measure round-trip delay
ping -c 10 google.com   # 10 pings — see consistency
```

**Python — simulate store-and-forward delay:**

```python
# Simulate store-and-forward delay calculation
def store_forward_delay(packet_size_bits, link_speed_bps, num_links):
    delay_per_hop = packet_size_bits / link_speed_bps
    total_delay = num_links * delay_per_hop
    print(f"Delay per hop: {delay_per_hop:.4f} seconds")
    print(f"Total delay ({num_links} links): {total_delay:.4f} seconds")

# Example: 1000-bit packet, 1 Mbps link, 3 hops
store_forward_delay(1000, 1_000_000, 3)
```

---

## 8. Common Mistakes

- **Confusing DSL and Dial-up:** DSL uses the phone line but is always-on and much faster than dial-up, which uses the voice channel itself.
- **Thinking cable internet speed is always as advertised:** Cable bandwidth is **shared** among neighbors — speeds drop during peak hours. DSL is dedicated, so it's more consistent (though slower).
- **Believing WiFi is a physical medium:** WiFi is an access technology, not a physical medium type. Its physical medium is **terrestrial radio channels** (electromagnetic waves through air).
- **Confusing store-and-forward with buffering in video:** These are different. Store-and-forward is what routers do for every packet at every hop. Video buffering is an application-layer technique.
- **Thinking all satellite internet is slow:** Old **GEO satellites** have high latency (~280ms). New **LEO satellites** like Starlink have much lower latency (~20–40ms) and are usable for most internet tasks.
- **Missing the formula:** Many students forget the store-and-forward delay formula: **N × L/R** for N links, L-bit packet, R bps link speed.

---

## 9. Memory Hook

**Access Network = Your driveway. Physical Media = The road type (dirt, asphalt, highway). Packet Switching = A courier who cuts your letter into pieces, sends each piece separately, and they meet at the destination.**

For store-and-forward specifically: think of a **relay race** — a runner (packet) must **completely arrive** at the checkpoint before the next runner can carry the baton forward. No partial handoffs allowed.

---

## 10. Ultra-Short Revision

Access networks are how end systems connect to the internet's first router — types include DSL (phone line), Cable/HFC (shared coaxial), FTTH (fiber directly home), Fixed Wireless, LEO satellites, Ethernet, WiFi, and 4G/5G. Physical media is what actually carries the signal — twisted-pair copper, coaxial cable, fiber optics (fastest, light-based), and radio channels (wireless). Once in the network core, data travels via packet switching — broken into packets, forwarded hop-by-hop through routers. Each router uses store-and-forward: it must receive the whole packet before forwarding it, adding a delay of L/R seconds per link. Total end-to-end store-and-forward delay = N × L/R for N links.

---

## 11. Interview / Exam Questions

**1. What is the difference between DSL and Cable internet access?**
DSL uses telephone copper lines and gives dedicated bandwidth to each user, while Cable uses coaxial TV lines but shares bandwidth among neighborhood users, causing slowdowns at peak times.

**2. Why do LEO satellites have lower latency than traditional GEO satellites?**
GEO satellites orbit at ~36,000 km causing ~280ms signal delay. LEO satellites like Starlink orbit at ~550 km, reducing latency to ~20–40ms.

**3. What is store-and-forward transmission, and what delay does it add?**
A router must receive a complete packet before forwarding it. For a packet of L bits on an R bps link, this adds L/R seconds per hop. For N links, total delay is N × L/R.

**4. Why is fiber optic cable faster and better than copper wire?**
Fiber uses pulses of light instead of electrical signals, giving far higher bandwidth, lower signal loss over long distances, and complete immunity to electromagnetic interference.

**5. What is packet switching and why is it better than sending data as one unit?**
Packet switching breaks data into small packets sent independently through routers. It's more efficient because links can be shared among many users simultaneously, and if one packet is lost, only that packet needs to be retransmitted, not the whole file.
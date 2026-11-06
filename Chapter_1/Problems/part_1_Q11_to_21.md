# Chapter 1 – Review Questions

## Section 1.3

---

## R11. End-to-End Delay with One Packet Switch

Suppose there is exactly one packet switch between a sending host and a receiving host.

### Given

- Packet length = **L** bits
- Transmission rate (Sender → Switch) = **R₁**
- Transmission rate (Switch → Receiver) = **R₂**

### Assumptions

- Store-and-forward packet switching
- Ignore propagation delay
- Ignore queuing delay
- Ignore processing delay

### Explanation

In **store-and-forward packet switching**, the switch must receive the **entire packet** before forwarding it to the next link.

### Delay from Sender to Switch

$$
\frac{L}{R_1}
$$

### Delay from Switch to Receiver

$$
\frac{L}{R_2}
$$

### Total End-to-End Delay

$$
\boxed{
\frac{L}{R_1}
+
\frac{L}{R_2}
}
$$

### Final Answer

The total end-to-end delay is

$$
\boxed{
\frac{L}{R_1}
+
\frac{L}{R_2}
}
$$

---

## R12. Circuit Switching vs Packet Switching

### Advantages of Circuit Switching

Circuit switching establishes a **dedicated communication path** before data transmission begins.

Advantages:

- Guaranteed bandwidth
- Predictable delay
- No congestion after the circuit is established
- Ideal for real-time voice and video communication

### Disadvantages

- Bandwidth remains reserved even when no data is being transmitted.
- Inefficient for bursty traffic.

---

### Advantages of Packet Switching

Packet switching does not reserve bandwidth in advance.

Advantages:

- Efficient bandwidth utilization
- Supports many users simultaneously
- Excellent for bursty Internet traffic
- No dedicated circuit is required
- Better overall network efficiency

---

### Advantages of TDM over FDM

#### Time Division Multiplexing (TDM)

- Uses the entire channel bandwidth during each assigned time slot.
- Easy to implement digitally.
- Minimal interference between users.
- Well suited for digital communication systems.

#### Frequency Division Multiplexing (FDM)

- Users transmit simultaneously on different frequency bands.
- Requires guard bands.
- More susceptible to interference.
- Less efficient for digital systems.

### Conclusion

TDM generally provides better bandwidth efficiency and simpler implementation than FDM for digital communication.

---

## R13. Statistical Multiplexing

### Given

- Link capacity = **2 Mbps**
- Each user transmits at **1 Mbps**
- Each user is active only **20%** of the time.

---

### (a) Circuit Switching

Each user requires

$$
1\ \text{Mbps}
$$

The total link capacity is

$$
2\ \text{Mbps}
$$

Number of users supported

$$
\frac{2}{1}=2
$$

### Answer

$$
\boxed{2\ \text{users}}
$$

---

### (b) Packet Switching

If only **two users** transmit simultaneously,

$$
2\times1=2\ \text{Mbps}
$$

The link exactly supports the traffic.

Therefore:

- No queue forms.
- Queuing delay is essentially zero.

If **three users** transmit simultaneously,

$$
3\times1=3\ \text{Mbps}
$$

Available bandwidth is only

$$
2\ \text{Mbps}
$$

Since

$$
3>2
$$

the arrival rate exceeds the transmission rate.

Therefore:

- Packets accumulate.
- A queue forms.
- Queuing delay increases.

---

### (c) Probability that a User is Transmitting

Each user is active **20%** of the time.

Therefore,

$$
P=0.2
$$

Answer

$$
\boxed{0.2}
$$

---

### (d) Probability that Three Users Transmit Simultaneously

The probability is

$$
0.2^3
$$

which equals

$$
0.008
$$

or

$$
0.8\%
$$

Answer

$$
\boxed{0.008}
$$

Since the queue grows only when all three users transmit simultaneously, the fraction of time the queue grows is

$$
\boxed{0.008}
$$
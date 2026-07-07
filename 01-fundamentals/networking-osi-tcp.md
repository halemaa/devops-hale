\# Networking: OSI \& TCP/IP



\## What it is

Networking is how machines talk to each other. Two mental models dominate:

the \*\*OSI model\*\* (7 layers, theoretical/teaching) and the \*\*TCP/IP model\*\*

(4 layers, what the internet actually uses). Both describe the same reality:

data gets wrapped in headers on the way out and unwrapped on the way in.



\## The two models side by side

```mermaid

flowchart LR

&#x20;   subgraph OSI\["OSI Model (7 layers)"]

&#x20;       L7\[7. Application]

&#x20;       L6\[6. Presentation]

&#x20;       L5\[5. Session]

&#x20;       L4\[4. Transport]

&#x20;       L3\[3. Network]

&#x20;       L2\[2. Data Link]

&#x20;       L1\[1. Physical]

&#x20;   end

&#x20;   subgraph TCPIP\["TCP/IP Model (4 layers)"]

&#x20;       T4\[Application]

&#x20;       T3\[Transport]

&#x20;       T2\[Internet]

&#x20;       T1\[Network Access]

&#x20;   end

&#x20;   L7 --> T4

&#x20;   L6 --> T4

&#x20;   L5 --> T4

&#x20;   L4 --> T3

&#x20;   L3 --> T2

&#x20;   L2 --> T1

&#x20;   L1 --> T1

```



\## What each layer actually does

\- \*\*L7 Application\*\* — HTTP, DNS, SSH, SMTP. The protocols apps speak.

\- \*\*L6 Presentation\*\* — encryption, encoding (TLS, JSON, character sets).

\- \*\*L5 Session\*\* — establishes and manages sessions between hosts.

\- \*\*L4 Transport\*\* — TCP (reliable, ordered) or UDP (fast, best-effort). Ports live here.

\- \*\*L3 Network\*\* — IP addressing and routing. IP packets.

\- \*\*L2 Data Link\*\* — MAC addresses, switches, frames (Ethernet, Wi-Fi).

\- \*\*L1 Physical\*\* — cables, radio waves, voltages.



\## Encapsulation — how a request travels

```mermaid

flowchart LR

&#x20;   App\[HTTP GET] --> TCP\[+ TCP header/port]

&#x20;   TCP --> IP\[+ IP header/addresses]

&#x20;   IP --> Eth\[+ Ethernet frame/MAC]

&#x20;   Eth --> Wire\[Bits on the wire]

```

Each layer wraps the payload with its own header. The receiver unwraps in reverse.



\## TCP vs UDP

| | TCP | UDP |

|---|---|---|

| Reliability | Guaranteed delivery | Best-effort |

| Ordering | Ordered | Unordered |

| Handshake | 3-way (SYN, SYN-ACK, ACK) | None |

| Used for | HTTP(S), SSH, SMTP, databases | DNS, VoIP, gaming, video |

| Overhead | Higher | Lower |



\## The 3-way handshake (memorize this)

```mermaid

sequenceDiagram

&#x20;   Client->>Server: SYN

&#x20;   Server->>Client: SYN-ACK

&#x20;   Client->>Server: ACK

&#x20;   Note over Client,Server: Connection established

```



\## IP addressing basics

\- \*\*IPv4\*\* — 32-bit, dotted decimal (e.g. `192.168.1.10`). Running out; NAT extends its life.

\- \*\*IPv6\*\* — 128-bit, hex with colons. The future, slow adoption.

\- \*\*CIDR\*\* — `/24` means first 24 bits are the network. `192.168.1.0/24` = 256 addresses.

\- \*\*Private ranges\*\* — `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. Not routable on the public internet.

\- \*\*Loopback\*\* — `127.0.0.1` = this machine.



\## Ports worth knowing by heart

\- 22 SSH




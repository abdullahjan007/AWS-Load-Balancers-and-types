# AWS-Load-Balancers-and-types

# AWS Load Balancers — ALB, NLB aur GWLB

> Ye mere AWS Load Balancers ke learning notes hain. In mein concepts ko beginner-friendly Roman Urdu mein examples ke saath explain kiya gaya hai.

## Table of Contents

1. [Load Balancer kya hota hai?](#1-load-balancer-kya-hota-hai)
2. [Networking ki zaroori basics](#2-networking-ki-zaroori-basics)
3. [Load Balancer ke important terms](#3-load-balancer-ke-important-terms)
4. [Application Load Balancer (ALB)](#4-application-load-balancer-alb)
5. [Network Load Balancer (NLB)](#5-network-load-balancer-nlb)
6. [Gateway Load Balancer (GWLB)](#6-gateway-load-balancer-gwlb)
7. [GWLB Flow Stickiness aur 5-tuple](#7-gwlb-flow-stickiness-aur-5-tuple)
8. [Kya teeno Load Balancers saath use ho sakte hain?](#8-kya-teeno-load-balancers-saath-use-ho-sakte-hain)
9. [HTTPS request ka step-by-step safar](#9-https-request-ka-step-by-step-safar)
10. [ALB vs NLB vs GWLB](#10-alb-vs-nlb-vs-gwlb)
11. [Important takeaways](#11-important-takeaways)

---

## 1. Load Balancer kya hota hai?

Load Balancer aik aisa component hai jo incoming traffic ko multiple backend servers ya targets mein distribute karta hai.

### Load Balancer ki zaroorat kyun hoti hai?

Socho aik website sirf aik server par chal rahi hai. Agar aik hi waqt mein hazaron users website access karein, toh server overload ho sakta hai.

Load Balancer incoming traffic ko multiple servers mein distribute kar deta hai.

```text
                  Users
                    |
                    v
              Load Balancer
              /     |     \\
             v      v      v
          Server 1 Server 2 Server 3
```

### Load Balancer ke main benefits

- **Traffic Distribution:** Incoming traffic ko multiple targets mein distribute karta hai.
- **High Availability:** Healthy targets ko traffic bhejne mein help karta hai.
- **Health Checks:** Check karta hai ke backend targets healthy hain ya nahi.
- **Scalability:** Multiple servers ke zariye application ko zyada traffic handle karne mein help milti hai.

Yaad rakho: Sirf Load Balancer laga dene se poori application automatically highly available nahi ban jati. Backend servers, Availability Zones, health checks aur application architecture bhi sahi design honi chahiye.

---

## 2. Networking ki zaroori basics

NLB aur GWLB samajhne se pehle TCP, UDP, TLS aur ports ka basic concept samajhna zaroori hai.

### TCP (Transmission Control Protocol)

TCP aik **connection-oriented transport protocol** hai.

Iska matlab hai ke data transfer se pehle client aur server ke darmiyan connection establish hota hai.

TCP ki important properties:

- Reliable data delivery.
- Data correct order mein deliver karna.
- Acknowledgements ke zariye confirm karna ke data receive hua.
- Data lost ho jaye toh retransmission karna.

TCP connection establish karne ke liye aam tor par three-way handshake use hota hai:

```text
Client                         Server
  |                               |
  | -------- SYN ---------------> |
  | <----- SYN-ACK -------------- |
  | -------- ACK ---------------> |
  |                               |
  |       Connection ready        |
```

Examples:

- Traditional HTTP/1.1 aur HTTP/2.
- HTTPS over TCP.
- SSH.
- Database connections.

### UDP (User Datagram Protocol)

UDP aik **connectionless transport protocol** hai.

TCP ke unlike, UDP mein TCP jaisa three-way handshake nahi hota. UDP khud delivery confirmation, retransmission ya packet ordering guarantee nahi karta.

UDP ka overhead kam hota hai, is liye low-latency applications mein useful hota hai.

Examples:

- DNS queries (commonly UDP).
- Online gaming.
- Voice aur video communication.

Note: UDP use karne wali application apne level par reliability mechanisms implement kar sakti hai.

### TLS (Transport Layer Security) aur HTTPS

TLS communication ko secure banata hai.

TLS ke main benefits:

- **Encryption:** Data ko unauthorized logon ke liye readable nahi rehne deta.
- **Authentication:** Certificate ke zariye server ki identity verify karne mein help karta hai.
- **Integrity:** Data mein unauthorized changes detect karne mein help karta hai.

HTTPS ka matlab hai HTTP communication ko TLS ke zariye secure karna.

Traditional HTTPS (HTTP/1.1 aur HTTP/2) ka protocol stack:

```text
HTTP
  |
 TLS
  |
 TCP
  |
 IP
```

Examples:

| URL | Protocol stack | Default port |
|---|---|---:|
| `http://example.com` | HTTP over TCP | 80 |
| `https://example.com` | HTTP over TLS over TCP | 443 |

**Important exception:** HTTP/3, QUIC use karta hai jo UDP ke upar chalta hai. Is liye har HTTPS connection lazmi nahi ke TCP use kare.

### Ports

IP address network par destination ko identify karta hai. Port number traffic ko us destination par chalne wali specific service ya application tak pohanchane mein help karta hai.

Common ports:

| Service | Common port |
|---|---:|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| DNS | 53 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| Redis | 6379 |

Port khud application nahi hota. Application ya service kisi port par listen karti hai.

### Custom TCP application kya hoti hai?

Har application website ya browser-based nahi hoti.

Example: Aik factory mein machines hain jo apna temperature aur sensor data aik central server ko bhejti hain. Company ne aik custom application banayi hai jo TCP port `5000` par listen karti hai.

- TCP reliable transport provide karta hai.
- Custom application define karti hai ke message ka format kya hoga aur data ka matlab kya hai.

Custom TCP applications ki doosri examples:

- Online game servers.
- IoT systems.
- Company ke internal software.
- Custom database ya messaging services.

---

## 3. Load Balancer ke important terms

### Listener

Listener Load Balancer ka entry point hota hai. Is mein protocol aur port configure kiye jate hain.

Examples:

- TCP listener on port 443.
- TLS listener on port 443.
- HTTP listener on port 80.
- HTTPS listener on port 443.

Listener decide karta hai ke kis protocol aur port par aane wali traffic ko receive karna hai.

### Target Group

Target Group mein woh backend targets register hote hain jin tak Load Balancer traffic forward kar sakta hai.

Targets ki examples:

- EC2 instances.
- Private IP addresses.
- Containers, supported configurations mein.
- NLB ke case mein ALB bhi target ho sakta hai.

### Health Check

Health Check ke zariye Load Balancer check karta hai ke target traffic receive karne ke liye healthy hai ya nahi.

Agar target unhealthy ho jaye, toh Load Balancer usay normal traffic distribution se hata sakta hai. Exact behavior Load Balancer aur target group configuration par depend karta hai.

---

## 4. Application Load Balancer (ALB)

ALB **Layer 7 (Application Layer)** par kaam karta hai.

Ye HTTP aur HTTPS requests ko application level par samajh sakta hai.

ALB in cheezon ki bunyaad par routing kar sakta hai:

- URL path.
- Hostname.
- HTTP headers.
- HTTP request methods.
- Query strings, supported rules ke mutabiq.

### Example: E-commerce website

Suppose aik website hai:

`https://shop.example.com`

Is website ke multiple application parts hain:

- Products.
- Payments.
- Login.

ALB rules kuch is tarah ho sakte hain:

| Request path | ALB traffic kahan bhejega? |
|---|---|
| `/products` | Products Target Group |
| `/payment` | Payment Target Group |
| `/login` | Login Target Group |
| `/` | Frontend Target Group |

```text
                   Users
                     |
                     v
                    ALB
                     |
              URL path check
              /      |      \\
             v       v       v
         Products Payment  Login
         Service  Service  Service
```

### ALB ki important features

- Layer 7 load balancing.
- HTTP aur HTTPS support.
- Path-based routing.
- Host-based routing.
- HTTP headers aur methods par rules.
- TLS termination.
- WebSocket support.
- Multiple backend services ko route karna.

### TLS termination ALB par

Agar user HTTPS request bhejta hai aur ALB par HTTPS listener configured hai, toh ALB TLS connection terminate kar sakta hai.

Iska matlab:

1. Browser aur ALB ke darmiyan encrypted connection hota hai.
2. ALB certificate use karke TLS connection handle karta hai.
3. ALB HTTP request ko inspect kar sakta hai.
4. ALB backend ko configured protocol ke mutabiq request forward karta hai.

### ALB kab use karna chahiye?

Jab application HTTP/HTTPS par chalti ho aur URL, hostname ya HTTP details ke basis par routing chahiye ho, toh ALB useful hota hai.

---

## 5. Network Load Balancer (NLB)

NLB **Layer 4 (Transport Layer)** par kaam karta hai.

Ye network connections aur transport-level information ke basis par traffic handle karta hai.

NLB HTTP URL path jaise `/products` ko ALB ki tarah inspect karke routing nahi karta.

### NLB ki important features

- TCP support.
- UDP support.
- TLS listener support.
- High performance aur low latency ke liye design.
- Static IP addresses provide kar sakta hai (Availability Zone ke mutabiq).
- Supported configurations mein client source IP preserve kar sakta hai.
- Healthy targets ki taraf traffic forward karta hai.

### NLB ka basic flow

```text
                 Internet Users
                       |
                       v
                      NLB
                       |
                 Listener :443
                       |
                       v
                  Target Group
                 /      |      \\
                v       v       v
             Server 1 Server 2 Server 3
```

### NLB Listener ke types

#### 1. TCP Listener

TCP listener TCP connections receive karta hai.

Example:

- Listener protocol: TCP.
- Listener port: 443.
- Target group: backend target.

Agar user HTTPS traffic bhej raha hai aur NLB par TCP listener port 443 configured hai, toh NLB TLS ko decrypt nahi karta. Woh encrypted TLS data ko TCP connection ke through target tak forward karta hai.

Isay aam tor par **TLS passthrough** kaha jata hai.

Backend par TLS terminate ho sakta hai.

#### 2. TLS Listener

TLS listener NLB par TLS connection terminate kar sakta hai.

Example:

- Listener protocol: TLS.
- Listener port: 443.
- Certificate: AWS Certificate Manager (ACM) se configured certificate.
- Target group: TCP ya TLS configuration ke mutabiq.

Is case mein NLB TLS connection handle karta hai aur decrypted application data target ki taraf forward kar sakta hai. Backend connection ki security target group configuration par depend karti hai.

#### TCP listener port 443 vs TLS listener port 443

| TCP listener :443 | TLS listener :443 |
|---|---|
| NLB TLS decrypt nahi karta | NLB TLS terminate karta hai |
| Encrypted traffic target tak forward hoti hai | TLS NLB par handle hota hai |
| Backend TLS terminate kar sakta hai | Backend connection TCP ya TLS ho sakta hai, configuration ke mutabiq |

**Yaad rakho:** Port 443 hone ka matlab ye nahi ke listener lazmi TLS listener hai. Listener ka protocol bhi dekhna hota hai.

#### 3. UDP Listener

UDP listener UDP traffic receive karta hai.

Examples:

- Online gaming.
- Real-time communication.
- UDP-based custom applications.

### NLB custom TCP application ke liye

Suppose factory ki machines aik custom server ko data bhejti hain.

- Custom server TCP port `5000` par listen karta hai.
- Machines TCP connections establish karti hain.
- NLB TCP listener port `5000` par traffic receive karta hai.
- NLB traffic ko registered backend targets mein forward karta hai.

Yahan ALB ki zaroorat nahi, kyun ke ye HTTP website routing ka use case nahi hai.

### NLB aur ALB ka farq

ALB request ka URL path aur hostname samajh sakta hai. NLB transport-level connection handle karta hai.

Example:

- ALB `/products` ko Products service tak bhej sakta hai.
- NLB TCP port 443 par aane wali connection ko configured target tak forward karta hai, lekin URL path ki bunyaad par routing nahi karta.

---

## 6. Gateway Load Balancer (GWLB)

GWLB ka purpose ALB aur NLB se different hai.

GWLB **Layer 3 (Network Layer)** par IP traffic ko security appliances, jaise firewalls, tak distribute karne ke liye use hota hai.

Examples of security appliances:

- Firewalls.
- IDS (Intrusion Detection System).
- IPS (Intrusion Prevention System).
- Deep Packet Inspection (DPI) appliances.

### GWLB ka basic architecture

```text
                  Network Traffic
                         |
                         v
                        GWLB
                         |
                  /      |      \\
                 v       v       v
             Firewall 1 Firewall 2 Firewall 3
```

GWLB khud firewall nahi hota. Ye traffic ko security appliances tak distribute karta hai.

### GWLB traffic kaise bhejta hai?

GWLB security appliances ke saath traffic exchange karne ke liye **GENEVE encapsulation** use karta hai.

- GENEVE UDP port `6081` use karta hai.
- Original packet ko outer packet ke andar encapsulate kiya jata hai.
- Security appliance outer encapsulation remove karke original packet inspect kar sakta hai.
- Appliance inspected traffic ko GWLB ke through wapas bhejta hai.

Simplified concept:

```text
Outer GENEVE / UDP packet
  |
  |-- Original user packet
        |
        |-- IP
        |-- TCP / UDP
        |-- TLS / Application data
```

### GWLB Endpoint (GWLBe)

GWLB ko VPC traffic path mein insert karne ke liye **Gateway Load Balancer Endpoint** use hota hai.

Route tables traffic ko endpoint ki taraf direct karti hain. Endpoint traffic ko GWLB service tak pohanchata hai.

Simplified inbound inspection path:

```text
Internet
   |
   v
Internet Gateway
   |
   v
Route Table
   |
   v
Gateway Load Balancer Endpoint
   |
   v
GWLB
   |
   v
Firewall Appliance
   |
   v
GWLB / Endpoint return path
   |
   v
Application Network
```

Actual architecture mein route tables, subnet design, endpoint placement aur return path carefully configure karna hota hai.

### Firewall kya check karta hai?

Firewall ki capabilities aur configuration par depend karta hai. Woh check kar sakta hai:

- Source IP.
- Destination IP.
- Source aur destination ports.
- Protocol.
- Configured security policies.
- Suspicious traffic patterns.

Agar traffic allowed ho toh firewall usay aage bhejta hai. Agar blocked ho toh drop kar sakta hai.

### HTTPS aur firewall inspection

Agar HTTPS traffic encrypted hai aur firewall TLS decrypt nahi kar raha, toh firewall normally HTTP URL path ya request body ka actual content nahi dekh sakta.

Woh available network-level metadata aur encrypted traffic ko inspect kar sakta hai. Actual HTTPS content inspect karne ke liye suitable TLS inspection/decryption configuration chahiye hoti hai.

---

## 7. GWLB Flow Stickiness aur 5-tuple

Ye GWLB ka aik important concept hai, khaas tor par jab stateful firewalls use ho rahe hon.

### Network Flow kya hota hai?

Network flow do endpoints ke darmiyan communication ka aik specific flow hota hai.

Example:

- Source IP: `192.168.1.10`
- Destination IP: `203.0.113.20`
- Source Port: `52000`
- Destination Port: `443`
- Protocol: `TCP`

Ek TCP connection mein multiple packets exchange ho sakte hain. Ye packets same flow ka hissa hote hain.

### Stateful Firewall kya hota hai?

Stateful firewall connection ki state track karta hai.

Example:

1. Client server ko connection request bhejta hai.
2. Firewall connection ko allow karta hai.
3. Firewall apni state table mein connection ki information save karta hai.
4. Agle packets aane par firewall saved state check karta hai.

### Problem: Packets different firewalls par chale jayein

Suppose do firewalls hain:

- Firewall 1.
- Firewall 2.

```text
                 GWLB
                   |
             /           \\
            v             v
       Firewall 1     Firewall 2
```

Ab:

1. Pehla packet Firewall 1 par gaya.
2. Firewall 1 ne connection allow karke state save kar li.
3. Doosra packet Firewall 2 par chala gaya.
4. Firewall 2 ke paas us connection ki state nahi ho sakti.
5. Firewall 2 packet ko unknown connection samajh kar drop kar sakta hai.

Ye problem tab aa sakti hai jab firewalls apni connection state share nahi karte.

### Flow Stickiness kya karti hai?

Flow stickiness ka purpose ye hai ke same network flow ke packets consistently same security appliance par jayein.

```text
                  GWLB
                   |
                   v
               Firewall 1
                   |
          Packet 1, Packet 2,
          Packet 3, Packet 4
```

Ab Firewall 1 ke paas us flow ki state available rehti hai.

### 5-tuple kya hota hai?

GWLB flow identify karne ke liye 5-tuple information use kar sakta hai:

| Field | Example |
|---|---|
| Source IP | `192.168.1.10` |
| Destination IP | `203.0.113.20` |
| Source Port | `52000` |
| Destination Port | `443` |
| Protocol | `TCP` |

In paanch values ki combination ko 5-tuple kehte hain.

GWLB flow hash ki bunyaad par appliance select karta hai. Same flow ke packets ke liye consistent selection hoti hai, configured stickiness behavior ke mutabiq.

**Simple analogy:** Socho bank mein tumhara aik transaction Officer 1 handle kar raha hai. Officer 1 ke paas transaction ki history hai. Agar usi transaction ka agla step Officer 2 ko de diya jaye aur Officer 2 ke paas history na ho, toh confusion ho sakti hai. Flow stickiness ka idea ye hai ke usi flow ko usi appliance par rakha jaye.

Note: Flow stickiness ka exact behavior target group ki stickiness configuration aur traffic type par depend karta hai.

---

## 8. Kya teeno Load Balancers saath use ho sakte hain?

Haan, ALB, NLB aur GWLB aik hi overall AWS architecture mein use ho sakte hain. Lekin har aik ka role different hota hai.

Example e-commerce application:

- GWLB: Traffic ko firewall appliances tak inspection ke liye bhejta hai.
- NLB: Network-level connection handling aur forwarding karta hai.
- ALB: HTTP/HTTPS requests ko URL ya hostname ke basis par backend services tak route karta hai.

### Conceptual architecture

```text
                Internet Users
                      |
                      v
              Internet Gateway
                      |
                      v
              GWLB Endpoint
                      |
                      v
                    GWLB
                      |
                      v
                 Firewall
                      |
              Inspected traffic
                      |
                      v
                    NLB
                      |
                      v
                    ALB
                 /    |    \\
                v     v     v
           Frontend Products Payment
```

**Important architecture clarification:**

Ye aik conceptual flow hai, universal fixed chain nahi.

- GWLB ko VPC routing aur Gateway Load Balancer Endpoints ke through traffic path mein insert kiya jata hai.
- NLB ke target group mein ALB ko target ke taur par register kiya ja sakta hai.
- GWLB ko simply NLB ke normal target group mein rakh kar direct NLB → GWLB → ALB chain nahi banayi jati.
- Actual traffic path route tables, subnets, endpoint placement, target groups aur return routing par depend karta hai.

### Kab kaunsa Load Balancer use karein?

| Scenario | Suitable choice |
|---|---|
| Simple HTTP/HTTPS website | ALB |
| URL path ya hostname-based routing | ALB |
| High-performance TCP application | NLB |
| UDP-based application | NLB |
| Static IP requirement | NLB |
| Firewall/security appliance fleet | GWLB |
| Security inspection + application routing | GWLB + ALB |
| Network-level entry + ALB routing | NLB + ALB |
| Security inspection + NLB + ALB | Teeno, agar architecture ko teeno capabilities chahiye hon |

Har project mein teeno Load Balancers use karna zaroori nahi hota.

---

## 9. HTTPS request ka step-by-step safar

Ab aik example se samajhte hain ke user ki website request architecture mein kaise travel kar sakti hai.

Example:

User browser mein type karta hai:

```text
https://shop.example.com/products
```

Assume karte hain ke:

- Website AWS par hosted hai.
- DNS domain ko NLB ke endpoint tak resolve karta hai.
- NLB par TCP listener port 443 hai.
- NLB target group mein ALB registered hai.
- GWLB ko VPC routing ke zariye inspection path mein insert kiya gaya hai.
- Firewall traffic ko allow karta hai.
- ALB par HTTPS listener configured hai.

### Step 1: DNS lookup

Browser ko `shop.example.com` ka destination maloom karna hota hai.

DNS lookup domain ko configured AWS endpoint ke address tak resolve karne mein help karta hai.

### Step 2: Browser connection establish karta hai

Traditional HTTPS (HTTP/1.1 ya HTTP/2) mein browser TCP connection establish karta hai.

TCP three-way handshake:

```text
Browser                       Destination
   |                               |
   | -------- SYN --------------> |
   | <----- SYN-ACK -------------- |
   | -------- ACK --------------> |
   |                               |
   |       TCP connection ready    |
```

Iske baad TLS handshake hota hai aur encrypted communication establish hoti hai.

### Step 3: HTTPS traffic NLB tak pohanchti hai

Example traffic details:

| Field | Example |
|---|---|
| Source IP | `198.51.100.25` |
| Source Port | `52000` |
| Destination | NLB endpoint |
| Destination Port | `443` |
| Transport protocol | TCP |
| Application protocol | HTTPS |

Ye example IP address documentation ke liye reserved hai.

### Step 4: NLB listener traffic receive karta hai

NLB par TCP listener port 443 configured hai.

NLB:

1. Incoming TCP connection receive karta hai.
2. Listener protocol aur port ke mutabiq traffic handle karta hai.
3. Associated target group ka healthy target select karta hai.
4. Traffic configured target, yani ALB, ki taraf forward karta hai.

TCP listener hone ki wajah se NLB TLS decrypt nahi karta.

### Step 5: GWLB security inspection path

Agar architecture mein GWLB endpoint aur routing inspection ke liye configured hain, toh traffic route table ke mutabiq GWLB endpoint aur security appliance path se guzarti hai.

GWLB:

1. Traffic ko GENEVE ke zariye security appliance tak bhejta hai.
2. Firewall original packet inspect karta hai.
3. Firewall configured rules ke mutabiq traffic allow ya block karta hai.
4. Allowed traffic inspection path se aage return hoti hai.

Actual path routing design par depend karta hai. GWLB har architecture mein NLB ke baad ek simple inline hop nahi hota.

### Step 6: ALB HTTP request process karta hai

ALB par HTTPS listener configured hai.

ALB TLS terminate kar sakta hai aur HTTP request ko inspect kar sakta hai.

Example request:

```http
GET /products HTTP/1.1
Host: shop.example.com
```

ALB `/products` path ko identify karta hai aur Products Target Group select karta hai.

### Step 7: Backend request process karta hai

Products Target Group mein multiple healthy targets ho sakte hain.

ALB un mein se target select karke request forward karta hai.

Backend:

1. Request receive karta hai.
2. Products data retrieve karta hai.
3. Response generate karta hai.
4. Response ALB ko bhejta hai.

### Step 8: Response user tak wapas jata hai

Response configured return path ke mutabiq user tak wapas jata hai.

Agar firewall stateful hai aur dono directions ki inspection required hai, toh return traffic routing ko bhi sahi configure karna zaroori hai. Is se firewall connection ki state correctly track kar sakta hai.

---

## 10. ALB vs NLB vs GWLB

| Feature | ALB | NLB | GWLB |
|---|---|---|---|
| Main layer | Layer 7 | Layer 4 | Layer 3 |
| Main purpose | Application routing | Network connection handling | Security appliances distribution |
| Traffic focus | HTTP/HTTPS | TCP/UDP/TLS aur supported protocols | IP traffic |
| URL path routing | Yes | No | No |
| Host-based routing | Yes | No | No |
| Firewall appliance integration | Direct main purpose nahi | Direct main purpose nahi | Yes |
| Health checks | Yes | Yes | Yes, appliance targets ke liye |
| Example | `/products` ko Products service bhejna | TCP 443 connection forward karna | Traffic Firewall 1 ya Firewall 2 tak bhejna |

### Easy way to remember

- **ALB:** Application request ko samajhta hai.
- **NLB:** Network connection ko handle karta hai.
- **GWLB:** Traffic ko security appliances tak pohanchata hai.

---

## 11. Important takeaways

1. Load Balancer incoming traffic ko multiple targets mein distribute karta hai.
2. ALB Layer 7 par HTTP/HTTPS application-level routing karta hai.
3. NLB Layer 4 par network connections aur transport-level traffic handle karta hai.
4. GWLB Layer 3 par IP traffic ko security appliances tak distribute karta hai.
5. TCP reliable aur ordered delivery provide karta hai.
6. UDP connectionless hai aur TCP jaisi built-in delivery guarantees nahi deta.
7. TLS communication ko encrypt aur authenticate karne mein help karta hai.
8. Traditional HTTPS mein HTTP, TLS aur TCP use hote hain; HTTP/3 QUIC over UDP use karta hai.
9. NLB ka TCP listener port 443 TLS traffic ko decrypt kiye baghair forward kar sakta hai.
10. NLB ka TLS listener TLS connection terminate kar sakta hai.
11. GWLB security appliance nahi, balkeh appliances tak traffic distribute karne wala Load Balancer hai.
12. GWLB Endpoint aur VPC route tables traffic ko inspection path mein insert karne ke liye important hain.
13. Stateful firewall ke liye flow stickiness important hai.
14. 5-tuple mein Source IP, Destination IP, Source Port, Destination Port aur Protocol shamil hote hain.
15. ALB, NLB aur GWLB aik architecture mein combine ho sakte hain, lekin actual design routing aur requirements par depend karta hai.

---

## Official AWS Documentation

- [Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)
- [Gateway Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html)
- [Listeners for Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-listeners.html)
- [Gateway Load Balancer endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-load-balancer-endpoints.html)

---

**Learning note:** Ye notes concepts ko samajhne ke liye hain. Production architecture banate waqt AWS ki latest documentation, service limits, routing requirements aur security configuration zaroor verify karein.

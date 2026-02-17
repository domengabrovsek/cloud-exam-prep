# Section 2: Managing and Provisioning a Cloud Solution Infrastructure (~17.5% of Exam)

> **Quick context:** This section tests hands-on infrastructure management -- configuring networks, storage, compute, and AI/ML systems at architect level. Expect 9-11 questions on the exam. Unlike ACE, PCA questions focus on *why* you choose a particular configuration, not just *how* to set it up.

> Last updated: February 2026

---

## 2.1 Configuring Network Topologies

### Extending to On-premises Environments (Hybrid Networking)

Hybrid networking connects your on-premises data centers to Google Cloud. The two primary options are **Cloud VPN** and **Cloud Interconnect**, each with different cost, bandwidth, latency, and SLA characteristics.

#### Cloud VPN (HA VPN)

HA VPN provides encrypted IPsec tunnels over the public internet with a **99.99% SLA** when configured with the proper redundancy pattern (two tunnels across two interfaces).

**Key facts:**
- Each HA VPN gateway has **two interfaces**, each with its own external IP
- For 99.99% SLA: configure **two tunnels** (one per interface) to the peer gateway
- Maximum throughput per tunnel: **3 Gbps** (with proper MTU and single flow)
- Aggregate bandwidth scales by adding tunnels (up to 8 tunnels per gateway pair)
- Supports **dynamic routing only** (BGP via Cloud Router) -- no static routing on HA VPN
- Classic VPN (deprecated for new deployments) supported static routes but only offered 99.9% SLA
- Encrypted in transit using IKEv2

**HA VPN redundancy patterns:**

| Pattern | Tunnels | SLA | Use case |
|---------|---------|-----|----------|
| Two tunnels, one gateway each side | 2 | 99.99% | Standard HA setup |
| Two HA VPN gateways to two peer gateways | 4 | 99.99% | Higher aggregate bandwidth |
| Single tunnel | 1 | No SLA | Dev/test only |

**gcloud commands:**
```bash
# Create an HA VPN gateway
gcloud compute vpn-gateways create my-ha-vpn \
  --network=my-vpc \
  --region=us-central1

# Create a Cloud Router (required for HA VPN)
gcloud compute routers create my-router \
  --network=my-vpc \
  --region=us-central1 \
  --asn=65001

# Create a peer VPN gateway (external)
gcloud compute external-vpn-gateways create my-peer-gw \
  --interfaces 0=203.0.113.1,1=203.0.113.2

# Create VPN tunnels (one per interface for 99.99% SLA)
gcloud compute vpn-tunnels create tunnel-0 \
  --vpn-gateway=my-ha-vpn \
  --peer-external-gateway=my-peer-gw \
  --peer-external-gateway-interface=0 \
  --ike-version=2 \
  --shared-secret=my-secret \
  --router=my-router \
  --vpn-gateway-interface=0 \
  --region=us-central1

gcloud compute vpn-tunnels create tunnel-1 \
  --vpn-gateway=my-ha-vpn \
  --peer-external-gateway=my-peer-gw \
  --peer-external-gateway-interface=1 \
  --ike-version=2 \
  --shared-secret=my-secret \
  --router=my-router \
  --vpn-gateway-interface=1 \
  --region=us-central1

# Add BGP peers on the Cloud Router
gcloud compute routers add-bgp-peer my-router \
  --peer-name=peer-0 \
  --interface=if-tunnel-0 \
  --peer-ip-address=169.254.0.2 \
  --peer-asn=65002 \
  --region=us-central1
```

#### Cloud Interconnect

Cloud Interconnect provides **private, dedicated connectivity** between on-premises and Google Cloud -- traffic does NOT traverse the public internet.

| Feature | Dedicated Interconnect | Partner Interconnect |
|---------|----------------------|---------------------|
| **Bandwidth** | 10 Gbps or 100 Gbps per link | 50 Mbps -- 50 Gbps |
| **Physical connection** | Direct to Google PoP | Via service provider |
| **SLA** | 99.99% (with redundancy) | 99.99% (with redundancy) |
| **Use when** | High bandwidth, low latency, colocation near Google PoP | No colocation near Google PoP, or need < 10 Gbps |
| **Encryption** | Not encrypted by default (use MACsec or VPN overlay) | Not encrypted by default |
| **Setup time** | Weeks (physical cross-connect) | Days-weeks (provider dependent) |
| **Cost** | Port fee + VLAN attachment fee | Provider fee + VLAN attachment fee |

**Dedicated Interconnect redundancy for 99.99% SLA:**
- Minimum: **4 connections** across **2 metro areas** (2 connections per metro)
- Each metro must have connections to **2 different edge availability domains**
- Google provides a topology validation tool in the console

**Partner Interconnect** is the right choice when:
- Your data center is not near a Google PoP (colocation facility)
- You need less than 10 Gbps
- You want faster provisioning than Dedicated

**MACsec for Cloud Interconnect:**
- Layer 2 encryption for Dedicated Interconnect links
- Encrypts traffic without the overhead of IPsec
- Available on 10 Gbps and 100 Gbps links

#### Cloud Router and BGP

Cloud Router is the managed BGP speaker that enables dynamic route exchange between Google Cloud and external networks.

**Key concepts:**
- **ASN (Autonomous System Number):** Each Cloud Router needs an ASN; on-premises routers need their own ASN
- **BGP session:** Established over each VPN tunnel or VLAN attachment
- **Route advertisement:** Cloud Router advertises VPC subnet routes to on-premises; learns on-premises routes
- **Custom route advertisements:** Override default behavior to advertise specific prefixes
- **Graceful restart:** Maintains forwarding during Cloud Router maintenance

```bash
# Advertise custom routes (e.g., only specific subnets)
gcloud compute routers update my-router \
  --advertisement-mode=CUSTOM \
  --set-advertisement-groups=ALL_SUBNETS \
  --set-advertisement-ranges=10.0.0.0/8 \
  --region=us-central1
```

#### Decision: VPN vs Interconnect

| Criteria | HA VPN | Dedicated Interconnect | Partner Interconnect |
|----------|--------|----------------------|---------------------|
| **Bandwidth needed** | < 3 Gbps per tunnel (scale with multiple) | 10-100 Gbps per link | 50 Mbps-50 Gbps |
| **Latency sensitivity** | Tolerant (internet path) | Critical (private path) | Moderate (provider path) |
| **Encryption required** | Built-in (IPsec) | Add MACsec or VPN overlay | Add VPN overlay |
| **Budget** | Lowest | Highest | Mid-range |
| **Setup speed** | Hours | Weeks | Days-weeks |
| **Near Google PoP** | Not required | Required | Not required |

**Exam tips:**
- "Need encrypted connectivity quickly with < 3 Gbps?" --> **HA VPN**
- "Need 10+ Gbps dedicated private link, data center near Google PoP?" --> **Dedicated Interconnect**
- "Need private connectivity but no Google PoP nearby?" --> **Partner Interconnect**
- HA VPN **requires** Cloud Router (BGP) -- no static routing option
- Cloud Interconnect is **not encrypted by default** -- this is a common exam trap
- 99.99% SLA for Interconnect requires redundancy across **two metro areas**

**Docs:**
- [Cloud VPN overview](https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview)
- [Cloud Interconnect overview](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/overview)
- [Cloud Router overview](https://cloud.google.com/network-connectivity/docs/router/concepts/overview)
- [Choosing a Network Connectivity product](https://cloud.google.com/network-connectivity/docs/how-to/choose-product)

---

### Extending to Multicloud

#### Cross-Cloud Interconnect

Cross-Cloud Interconnect provides **dedicated, high-bandwidth connectivity** between Google Cloud and another cloud provider (AWS, Azure, Oracle Cloud).

**Key facts:**
- Available in **10 Gbps and 100 Gbps** link sizes
- Google manages the physical connection between clouds
- Supports **99.99% SLA** with proper redundancy (same pattern as Dedicated Interconnect)
- Traffic does NOT traverse the public internet
- Available in specific metro locations where Google has cross-connect to other providers
- You configure VLAN attachments on the Google side, and equivalent constructs on the other cloud (e.g., AWS Direct Connect, Azure ExpressRoute)

**When to use Cross-Cloud Interconnect:**
- High-bandwidth, low-latency multicloud architectures
- Data replication between clouds
- Multicloud disaster recovery
- Workloads split across providers (e.g., database on Google Cloud, application on AWS)

#### Google Cloud to Google Cloud Networking

For connecting multiple Google Cloud projects or organizations:

| Method | Use case | Transitivity |
|--------|----------|-------------|
| **Shared VPC** | Single org, centralized networking | N/A (same VPC) |
| **VPC Peering** | Cross-project or cross-org, low latency | Non-transitive |
| **Cloud VPN** | Cross-org with encryption needed | Non-transitive (per tunnel) |
| **Network Connectivity Center** | Hub-and-spoke for complex topologies | Transitive (via hub) |

**Exam tips:**
- Cross-Cloud Interconnect is for cloud-to-cloud, NOT cloud-to-on-premises
- VPC Peering is **non-transitive** -- if A peers with B and B peers with C, A cannot reach C through B
- For transitive connectivity across many VPCs, use **Network Connectivity Center** as a hub

**Docs:**
- [Cross-Cloud Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/cross-cloud-interconnect)
- [Network Connectivity Center](https://cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview)

---

### Security Protection

#### Cloud Armor

Cloud Armor provides **DDoS protection and WAF (Web Application Firewall)** capabilities for applications behind external load balancers.

**Key facts:**
- Operates at the **edge** of Google's network (PoPs worldwide)
- Supports **preconfigured WAF rules** (OWASP Top 10: SQLi, XSS, RCE, etc.)
- **Adaptive Protection** uses ML to detect and mitigate L7 DDoS attacks automatically
- **Rate limiting** rules to throttle abusive clients
- **Bot management** to identify and block automated threats
- **Named IP lists** for allow/deny by provider-managed IP ranges (e.g., CDN providers)
- **Two tiers:** Standard (pay-per-policy) and Managed Protection Plus (monthly subscription with DDoS response team)

**Security policy types:**
- **Backend security policies:** Attached to backend services (most common)
- **Edge security policies:** Attached to backend buckets (Cloud CDN)
- **Network edge security policies:** For Network Load Balancers and VM instances

```bash
# Create a Cloud Armor security policy
gcloud compute security-policies create my-policy \
  --description="WAF and DDoS protection"

# Add a WAF rule (block SQL injection)
gcloud compute security-policies rules create 1000 \
  --security-policy=my-policy \
  --expression="evaluatePreconfiguredWaf('sqli-v33-stable')" \
  --action=deny-403

# Add rate limiting rule
gcloud compute security-policies rules create 2000 \
  --security-policy=my-policy \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=300

# Attach policy to a backend service
gcloud compute backend-services update my-backend \
  --security-policy=my-policy \
  --global
```

#### Cloud IDS (Intrusion Detection System)

Cloud IDS provides **network-based threat detection** powered by Palo Alto Networks threat intelligence.

**Key facts:**
- Detects malware, spyware, command-and-control attacks, and other network threats
- Uses **packet mirroring** to inspect traffic (does NOT inline/block -- detection only)
- Integrates with Cloud Logging and Security Command Center
- Deploy IDS endpoints in each VPC where you need monitoring
- Can inspect north-south (internet) and east-west (internal) traffic
- For **blocking**, pair Cloud IDS with firewall rules or Cloud Armor

```bash
# Create a Cloud IDS endpoint
gcloud ids endpoints create my-ids-endpoint \
  --network=my-vpc \
  --zone=us-central1-a \
  --severity=MEDIUM \
  --async
```

#### Firewall Policies and Rules

Google Cloud offers a **hierarchical firewall** model:

| Level | Scope | Precedence |
|-------|-------|-----------|
| **Hierarchical firewall policies** | Organization or folder | Evaluated first |
| **Global network firewall policies** | VPC network (all regions) | Evaluated second |
| **Regional network firewall policies** | VPC network (specific region) | Evaluated third |
| **VPC firewall rules** | VPC network (legacy) | Evaluated last |

**Hierarchical firewall policies:**
- Applied at **organization or folder** level
- Rules cascade down to all projects/VPCs within that org/folder
- Can use `goto_next` action to delegate to lower-level policies
- Enables centralized security governance

**Firewall Insights:**
- Identifies **shadowed rules** (rules that never match because a higher-priority rule covers them)
- Shows **overly permissive rules** (rules that allow traffic not seen in logs)
- Provides **hit count** analysis for each rule
- Requires **Firewall Rules Logging** to be enabled

```bash
# Create a hierarchical firewall policy
gcloud compute firewall-policies create \
  --organization=123456789 \
  --short-name=org-policy

# Add a rule to the policy
gcloud compute firewall-policies rules create 1000 \
  --firewall-policy=org-policy \
  --direction=INGRESS \
  --action=deny \
  --src-ip-ranges=0.0.0.0/0 \
  --layer4-configs=tcp:22 \
  --description="Deny SSH from internet at org level"

# Associate with a folder
gcloud compute firewall-policies associations create \
  --firewall-policy=org-policy \
  --folder=987654321

# Create a VPC firewall rule (legacy style)
gcloud compute firewall-rules create allow-internal \
  --network=my-vpc \
  --allow=tcp,udp,icmp \
  --source-ranges=10.0.0.0/8 \
  --direction=INGRESS \
  --priority=1000
```

**Exam tips:**
- Cloud Armor protects against **DDoS and application-layer attacks** -- works with external HTTP(S) load balancers
- Cloud IDS **detects** threats but does NOT **block** them -- it is detection only
- Hierarchical firewall policies are evaluated **before** VPC firewall rules
- `goto_next` in hierarchical policies delegates evaluation to the next level (not the same as `allow`)
- Firewall Insights requires logging enabled to provide recommendations
- Cloud Armor **Adaptive Protection** is the ML-based L7 DDoS defense -- exam may describe symptoms of a DDoS attack and expect you to enable this

**Docs:**
- [Cloud Armor overview](https://cloud.google.com/armor/docs/cloud-armor-overview)
- [Cloud IDS overview](https://cloud.google.com/intrusion-detection-system/docs/overview)
- [Firewall policies overview](https://cloud.google.com/firewall/docs/firewall-policies-overview)
- [Firewall Insights](https://cloud.google.com/firewall/docs/using-firewall-insights)

---

### VPC Design and Load Balancing

#### VPC Network Architectures

| Architecture | Description | When to use | Limitations |
|-------------|-------------|-------------|-------------|
| **Shared VPC** | Host project owns the network; service projects deploy resources into shared subnets | Single organization, centralized network admin, separation of duties | Same org only; host project limit of 1 per service project |
| **VPC Peering** | Two VPCs exchange routes directly | Cross-org connectivity, low-latency inter-VPC | Non-transitive, 25 peering limit per VPC, no overlapping CIDRs |
| **Hub-and-spoke (NCC)** | Network Connectivity Center hub with spoke VPCs | Transitive routing across many VPCs, hybrid connectivity hub | Additional cost for NCC resources |
| **Hub-and-spoke (NVA)** | Appliance-based routing through a hub VPC | Need advanced firewall/IPS between spokes | Complex, single point of failure without HA |

**Shared VPC deep dive:**
- **Host project:** Owns the VPC, subnets, firewall rules, Cloud NAT, VPN/Interconnect
- **Service projects:** Deploy VMs, GKE clusters, Cloud Run, etc. into host project subnets
- **IAM roles:** `roles/compute.networkUser` on subnet or project for service project admins
- Each service project can be attached to **only one** host project
- Subnets can be shared selectively (not all subnets must be shared)

```bash
# Enable Shared VPC on the host project
gcloud compute shared-vpc enable HOST_PROJECT_ID

# Attach a service project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# Grant network user role on a specific subnet
gcloud projects add-iam-policy-binding HOST_PROJECT_ID \
  --member="serviceAccount:service-PROJECT_NUM@compute-system.iam.gserviceaccount.com" \
  --role="roles/compute.networkUser"
```

#### Load Balancing

Google Cloud load balancers are categorized by **layer** (L4 vs L7), **traffic type** (internal vs external), and **proxy mode** (proxy vs passthrough).

| Load Balancer | Layer | Scope | Proxy/Passthrough | Use case |
|--------------|-------|-------|-------------------|----------|
| **External Application LB** | L7 | Global (or Regional) | Proxy | Internet-facing HTTP(S) apps, URL-based routing, Cloud CDN, Cloud Armor |
| **Internal Application LB** | L7 | Regional (or Cross-region) | Proxy | Internal HTTP(S) microservices, gRPC, URL routing |
| **External Network LB (proxy)** | L4 | Global (or Regional) | Proxy | TCP/SSL termination, global reach for non-HTTP |
| **External Network LB (passthrough)** | L4 | Regional | Passthrough | UDP, ESP, TCP with client IP preservation |
| **Internal Network LB (passthrough)** | L4 | Regional | Passthrough | Internal TCP/UDP services, next-hop for routing |
| **Internal Network LB (proxy)** | L4 | Regional (or Cross-region) | Proxy | Internal TCP with proxy features |

**Key architectural decisions:**

| Requirement | Choose |
|-------------|--------|
| URL-based routing, path rules, host rules | Application LB (L7) |
| WebSocket support | Application LB (L7) |
| gRPC load balancing | Application LB (L7) |
| Cloud CDN integration | External Application LB |
| Cloud Armor WAF | External Application LB (or External Network LB proxy) |
| Preserve client source IP | Passthrough Network LB |
| Non-HTTP protocol (UDP, ESP) | Passthrough Network LB |
| Internal service-to-service | Internal Application LB or Internal Network LB |
| Global anycast IP | External Application LB or External Network LB (proxy) |
| Cross-region failover for internal | Cross-region Internal Application LB |

```bash
# Create a global external Application Load Balancer
# Step 1: Health check
gcloud compute health-checks create http my-health-check \
  --port=80 --request-path=/healthz

# Step 2: Backend service
gcloud compute backend-services create my-backend \
  --protocol=HTTP \
  --health-checks=my-health-check \
  --global

# Step 3: Add backend (MIG)
gcloud compute backend-services add-backend my-backend \
  --instance-group=my-mig \
  --instance-group-zone=us-central1-a \
  --global

# Step 4: URL map
gcloud compute url-maps create my-url-map \
  --default-service=my-backend

# Step 5: Target proxy
gcloud compute target-http-proxies create my-proxy \
  --url-map=my-url-map

# Step 6: Forwarding rule (global)
gcloud compute forwarding-rules create my-lb \
  --global \
  --target-http-proxy=my-proxy \
  --ports=80
```

#### Cloud NAT

Cloud NAT provides **outbound-only** internet access for VMs without external IPs.

**Key facts:**
- Regional resource, configured per Cloud Router
- Supports Compute Engine VMs and GKE nodes
- Does NOT support inbound connections (NAT is egress only)
- Uses a pool of NAT IP addresses (auto-allocated or manual)
- Configurable min/max ports per VM for connection scaling
- **Dynamic Port Allocation (DPA):** Automatically adjusts ports per VM based on usage
- Logging: can log translations, errors, or both

```bash
# Create Cloud NAT
gcloud compute routers nats create my-nat \
  --router=my-router \
  --region=us-central1 \
  --auto-allocate-nat-external-ips \
  --nat-all-subnet-ip-ranges \
  --enable-dynamic-port-allocation \
  --enable-logging
```

#### Private Google Access and Private Service Connect

**Private Google Access (PGA):**
- Allows VMs **without external IPs** to reach Google APIs (e.g., Cloud Storage, BigQuery)
- Enabled **per subnet**
- Uses the default route to Google (`199.36.153.8/30` for `restricted.googleapis.com` or `199.36.153.4/30` for `private.googleapis.com`)
- Does NOT require Cloud NAT

**Private Service Connect (PSC):**
- Creates **private endpoints** within your VPC for Google APIs or published services
- Consumer creates a **forwarding rule** with an internal IP that maps to the service
- Producer publishes services via **service attachments**
- Supports Google APIs, third-party services, and your own services
- IP address is from your VPC -- enables DNS resolution to private IP

```bash
# Enable Private Google Access on a subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access

# Create a Private Service Connect endpoint for Google APIs
gcloud compute addresses create psc-endpoint-ip \
  --region=us-central1 \
  --subnet=my-subnet \
  --addresses=10.0.1.100

gcloud compute forwarding-rules create psc-endpoint \
  --region=us-central1 \
  --network=my-vpc \
  --address=psc-endpoint-ip \
  --target-google-apis-bundle=all-apis
```

**Exam tips:**
- Shared VPC = **same org**, centralized networking. VPC Peering = **cross-org** possible, but **non-transitive**
- "Non-transitive" means A<->B and B<->C does NOT give A<->C -- this is the #1 VPC Peering trap
- Know which LB supports Cloud Armor: **External Application LB** (primary) and **External Network LB (proxy)**
- Cloud NAT is **outbound only** -- it does NOT enable inbound connections
- Private Google Access requires **no external IP** on VMs and is enabled at the **subnet** level
- Private Service Connect gives you a **private IP in your VPC** for Google APIs -- more control than PGA
- For global HTTP(S) traffic, you want **External Application LB** with a global anycast IP

**Docs:**
- [VPC overview](https://cloud.google.com/vpc/docs/overview)
- [Shared VPC](https://cloud.google.com/vpc/docs/shared-vpc)
- [Load balancing overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)
- [Cloud NAT overview](https://cloud.google.com/nat/docs/overview)
- [Private Google Access](https://cloud.google.com/vpc/docs/private-google-access)
- [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect)

---

### DNS Architecture

#### Cloud DNS

Cloud DNS is a high-performance, resilient, **global** managed DNS service with 100% SLA.

**Zone types:**

| Zone type | Description | Use case |
|-----------|-------------|----------|
| **Public zone** | Resolves names from the internet | Public-facing services |
| **Private zone** | Resolves names only within authorized VPC networks | Internal services, split-horizon DNS |
| **Forwarding zone** | Forwards queries to on-premises or external DNS servers | Hybrid DNS resolution (cloud to on-premises) |
| **Peering zone** | Forwards queries to another VPC's DNS resolution order | Cross-VPC DNS resolution without full peering |
| **Reverse lookup zone** | PTR records for IP-to-name resolution | Compliance, logging requirements |

**Split-horizon DNS:**
- Same domain name resolves differently depending on where the query originates
- Use a **private zone** for internal resolution (e.g., `app.example.com` -> `10.0.1.5`)
- Use a **public zone** for external resolution (e.g., `app.example.com` -> `34.120.0.1`)
- Private zones take precedence for queries from authorized VPCs

**DNS policies:**
- **Inbound DNS forwarding:** Allows on-premises networks to resolve names in Cloud DNS private zones (creates inbound forwarder IPs in each subnet)
- **Outbound DNS forwarding:** Cloud VMs can resolve names using on-premises DNS (via forwarding zones)
- **Response policies:** Override DNS responses for specific names (useful for blocking domains or redirecting traffic)

**DNSSEC:**
- Authenticates DNS responses to prevent spoofing
- Supported on **public zones only** (not private zones)
- Google manages key signing and rotation in managed DNSSEC mode

```bash
# Create a private DNS zone
gcloud dns managed-zones create my-private-zone \
  --dns-name="internal.example.com." \
  --description="Internal DNS" \
  --visibility=private \
  --networks=my-vpc

# Create a forwarding zone (to on-premises DNS)
gcloud dns managed-zones create onprem-forward \
  --dns-name="onprem.example.com." \
  --description="Forward to on-prem" \
  --visibility=private \
  --networks=my-vpc \
  --forwarding-targets=10.0.0.53,10.0.0.54

# Create DNS records
gcloud dns record-sets create app.internal.example.com. \
  --zone=my-private-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=10.0.1.5

# Create a DNS policy for inbound forwarding
gcloud dns policies create inbound-policy \
  --description="Allow on-prem to resolve cloud DNS" \
  --networks=my-vpc \
  --enable-inbound-forwarding

# Enable DNSSEC on a public zone
gcloud dns managed-zones update my-public-zone \
  --dnssec-state=on
```

**Exam tips:**
- **Split-horizon DNS** = same domain, different answers for internal vs external queries. Use private + public zones for the same domain name.
- **Forwarding zones** = cloud queries go to on-premises DNS. **Inbound DNS policy** = on-premises queries come to Cloud DNS. Know the difference.
- DNSSEC is **public zones only** -- you cannot enable it on private zones.
- Cloud DNS has a **100% SLA** -- one of the few Google Cloud services with this.
- **Peering zones** let you look up DNS in another VPC without full VPC peering -- useful for hub-and-spoke DNS.

**Docs:**
- [Cloud DNS overview](https://cloud.google.com/dns/docs/overview)
- [DNS policies](https://cloud.google.com/dns/docs/policies-overview)
- [DNSSEC](https://cloud.google.com/dns/docs/dnssec)
- [Split-horizon DNS](https://cloud.google.com/dns/docs/best-practices#use_split-horizon_dns)

---

## 2.2 Configuring Individual Storage Systems

### Data Storage Allocation

#### Cloud Storage Classes

| Storage class | Min duration | Use case | Retrieval cost | Availability SLA |
|--------------|-------------|----------|----------------|-----------------|
| **Standard** | None | Frequently accessed data, hot data | None | 99.95% (multi/dual-region), 99.9% (region) |
| **Nearline** | 30 days | Data accessed < 1x/month | Per GB | 99.9% (multi/dual-region), 99.0% (region) |
| **Coldline** | 90 days | Data accessed < 1x/quarter | Per GB (higher) | 99.9% (multi/dual-region), 99.0% (region) |
| **Archive** | 365 days | Data accessed < 1x/year | Per GB (highest) | 99.9% (multi/dual-region), 99.0% (region) |

**Critical exam facts about Archive storage:**
- Archive storage has **the same latency and throughput** as other classes -- it is NOT slow
- The cost difference is in **storage price** (cheapest) vs **retrieval price** (most expensive)
- Minimum storage duration charge: 365 days (delete earlier and you still pay for 365 days)
- Common exam trap: "Archive is slow" -- FALSE. It is fast to read, just expensive to read.

#### Bucket Location Types

| Location type | Redundancy | Use case | Example |
|--------------|-----------|----------|---------|
| **Region** | Single region, multiple zones | Lowest latency for regional workloads, compute-local data | `us-central1` |
| **Dual-region** | Two specific regions | HA with controlled data placement, compliance | `us-central1` + `us-east1` |
| **Multi-region** | Multiple regions across a continent | Highest availability, global content serving | `US`, `EU`, `ASIA` |

**Dual-region and Turbo Replication:**
- Dual-region buckets replicate data to **both regions**
- Default replication: RPO is "most objects within 15 minutes"
- **Turbo Replication:** Guarantees RPO of **15 minutes** for **100% of objects** (additional cost)
- Useful for DR scenarios requiring strict RPO guarantees

**Autoclass:**
- Automatically transitions objects between storage classes based on access patterns
- Moves frequently accessed objects to Standard, infrequently accessed to Nearline/Coldline/Archive
- No retrieval fees when Autoclass manages the transitions
- Set at **bucket level** -- cannot be per-object
- Simplifies lifecycle management for unpredictable access patterns

```bash
# Create a dual-region bucket with Turbo Replication
gcloud storage buckets create gs://my-dr-bucket \
  --location=us-central1+us-east1 \
  --default-storage-class=STANDARD \
  --enable-turbo-replication

# Create a bucket with Autoclass
gcloud storage buckets create gs://my-autoclass-bucket \
  --location=us-central1 \
  --enable-autoclass

# Change storage class of existing objects
gcloud storage objects update gs://my-bucket/my-file \
  --storage-class=NEARLINE
```

**Exam tips:**
- Archive storage is **NOT slow** -- same performance as Standard, just expensive to retrieve
- Turbo Replication is for dual-region buckets, guarantees **RPO < 15 min** for ALL objects
- Autoclass eliminates the need to manually manage lifecycle rules for storage class transitions
- Multi-region = highest availability but no control over which specific regions
- Dual-region = specific two regions, good for compliance + DR

**Docs:**
- [Cloud Storage classes](https://cloud.google.com/storage/docs/storage-classes)
- [Bucket locations](https://cloud.google.com/storage/docs/locations)
- [Turbo Replication](https://cloud.google.com/storage/docs/turbo-replication)
- [Autoclass](https://cloud.google.com/storage/docs/autoclass)

---

### Data Processing and Compute Provisioning

#### Storage-Compute Separation Patterns

A core Google Cloud architecture principle: **separate storage from compute** to enable independent scaling and cost optimization.

| Pattern | Storage | Compute | Benefit |
|---------|---------|---------|---------|
| BigQuery | Managed columnar storage | Serverless query slots | Pay separately for storage and queries; scale queries without moving data |
| Dataproc + GCS | Cloud Storage | Ephemeral Dataproc clusters | Spin up clusters on demand; data persists after cluster deletion |
| GKE + GCS/Filestore | GCS (object) or Filestore (NFS) | GKE pods | Scale pods independently of storage; persistent volumes decouple lifecycle |
| Vertex AI + GCS | GCS for training data | Training VMs (GPUs/TPUs) | Use different compute profiles for different training runs |

**Why this matters for the exam:**
- Architect questions will present scenarios where tightly coupling storage and compute causes scaling or cost problems
- The answer is almost always: "move data to Cloud Storage / BigQuery and use ephemeral compute"

---

### Security and Access Management for Storage

#### Cloud Storage Access Control

| Method | Scope | Use case |
|--------|-------|----------|
| **Uniform bucket-level access** | Bucket (recommended) | Simplified IAM-only access; no object-level ACLs |
| **Fine-grained (ACLs)** | Per-object | Legacy; needed only if different objects need different permissions |
| **Signed URLs** | Temporary per-object | Grant time-limited access without requiring Google account |
| **Signed policy documents** | Upload control | Control what can be uploaded (size, type, destination) |

**Uniform bucket-level access** is the recommended approach:
- Uses only IAM for access control (no ACLs)
- Once enabled for 90 days, it becomes **permanent** and cannot be reversed
- Simplifies auditing and policy management

**Signed URLs:**
```bash
# Generate a signed URL (valid for 1 hour)
gcloud storage sign-url gs://my-bucket/my-file.pdf \
  --duration=1h \
  --private-key-file=key.json
```

**Customer-Managed Encryption Keys (CMEK):**
- Default: Google-managed encryption (GMEK) -- automatic, no configuration
- CMEK: You create and manage keys in **Cloud KMS**; set as default on a bucket or per-object
- CSEK (Customer-Supplied): You provide the raw key with each API call -- Google never stores it
- Supports **key rotation** -- old objects remain readable with old key versions

```bash
# Set CMEK as default encryption on a bucket
gcloud storage buckets update gs://my-bucket \
  --default-encryption-key=projects/my-proj/locations/us-central1/keyRings/my-ring/cryptoKeys/my-key
```

**VPC Service Controls for Storage:**
- Creates a **service perimeter** around projects to prevent data exfiltration
- Even authorized users cannot copy data out of the perimeter to unauthorized projects
- Restricts Cloud Storage API calls to originate from within the perimeter
- Can define **access levels** (IP ranges, device trust) for exceptions

**Exam tips:**
- Default is **uniform bucket-level access** for new buckets -- exam prefers this
- Signed URLs grant temporary access **without requiring authentication** -- great for sharing with external users
- CMEK = you control the key in Cloud KMS. CSEK = you supply the key in every request. GMEK = Google handles everything.
- VPC Service Controls prevent **data exfiltration** -- even an IAM-authorized user inside the perimeter cannot copy data to a project outside the perimeter

**Docs:**
- [Cloud Storage access control](https://cloud.google.com/storage/docs/access-control)
- [Signed URLs](https://cloud.google.com/storage/docs/access-control/signed-urls)
- [CMEK for Cloud Storage](https://cloud.google.com/storage/docs/encryption/customer-managed-keys)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs/overview)

---

### Configuration for Data Transfer and Latency

| Transfer method | Data size | Source | Speed |
|----------------|-----------|--------|-------|
| **gsutil / gcloud storage** | Small-medium (< 1 TB) | On-premises or other cloud | Network speed |
| **Storage Transfer Service** | Any size | AWS S3, Azure Blob, HTTP, GCS-to-GCS | Managed, scheduled, repeatable |
| **Transfer Appliance** | Large (10 TB - 1 PB) | On-premises (limited bandwidth) | Physical shipping (days-weeks) |
| **BigQuery Data Transfer** | N/A | SaaS (Google Ads, etc.) | Managed, scheduled |

**Storage Transfer Service key features:**
- Supports **scheduled and recurring transfers**
- Source: AWS S3, Azure Blob Storage, HTTP/HTTPS sources, other GCS buckets
- Can filter by prefix, modification time, file extension
- Supports **bandwidth throttling** to avoid saturating network links
- **Event-driven transfers:** Trigger transfers on S3 event notifications

**Transfer Appliance:**
- Physical device shipped to your data center (100 TB or 480 TB capacity)
- Data is encrypted on the appliance (AES-256)
- Best when: `(Data size / Network bandwidth) > 1 week`
- Google uploads data to GCS after receiving the appliance back

**Latency optimization strategies:**
- **Dual-region buckets:** Place data close to compute in two regions
- **Multi-region buckets:** Automatic geo-redundancy across a continent
- **Cloud CDN with GCS backend:** Cache static content at edge PoPs
- **Parallel composite uploads:** Split large files into parts, upload in parallel, compose

**Exam tips:**
- Transfer Appliance is for **offline** bulk transfer when network is the bottleneck
- Storage Transfer Service is for **online** transfers from other clouds or HTTP sources
- For GCS-to-GCS transfers (e.g., cross-region replication), use **Storage Transfer Service**, not gsutil
- Cloud CDN can front a GCS bucket for low-latency static content delivery globally

**Docs:**
- [Storage Transfer Service](https://cloud.google.com/storage-transfer/docs/overview)
- [Transfer Appliance](https://cloud.google.com/transfer-appliance/docs/overview)

---

### Data Retention and Data Lifecycle Management

#### Object Lifecycle Management

Define rules on a bucket to automatically transition or delete objects based on age, creation date, storage class, or other conditions.

**Common lifecycle rules:**

| Condition | Action | Example |
|-----------|--------|---------|
| `age > 30` | `SetStorageClass: NEARLINE` | Move to Nearline after 30 days |
| `age > 90` | `SetStorageClass: COLDLINE` | Move to Coldline after 90 days |
| `age > 365` | `SetStorageClass: ARCHIVE` | Move to Archive after 1 year |
| `age > 730` | `Delete` | Delete after 2 years |
| `isLive: false` | `Delete` after 30 days | Delete non-current versions after 30 days |
| `numNewerVersions > 3` | `Delete` | Keep only 3 versions |

```bash
# Apply lifecycle rules from a JSON file
gcloud storage buckets update gs://my-bucket \
  --lifecycle-file=lifecycle.json
```

Example `lifecycle.json`:
```json
{
  "rule": [
    {
      "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
      "condition": {"age": 30}
    },
    {
      "action": {"type": "SetStorageClass", "storageClass": "ARCHIVE"},
      "condition": {"age": 365}
    },
    {
      "action": {"type": "Delete"},
      "condition": {"age": 730}
    }
  ]
}
```

#### Retention Policies and Compliance (WORM)

| Feature | Description | Reversible? |
|---------|-------------|-------------|
| **Retention policy** | Minimum retention period for objects | Yes (can remove/shorten before locking) |
| **Bucket Lock** | Locks the retention policy permanently | **No** -- irreversible, cannot shorten or remove |
| **Object Hold (event-based)** | Prevents deletion until hold is released by an event | Yes (release the hold) |
| **Object Hold (temporary)** | Prevents deletion until hold is removed | Yes (remove the hold) |

**WORM compliance (Write Once, Read Many):**
- Set a retention policy + **lock the bucket** = WORM storage
- Once locked, objects **cannot be deleted or overwritten** until retention period expires
- The retention policy itself **cannot be shortened or removed** after locking
- Meets regulatory requirements (SEC Rule 17a-4, CFTC Rule 1.31, FINRA)

```bash
# Set a retention policy (1 year = 31536000 seconds)
gcloud storage buckets update gs://my-bucket \
  --retention-period=31536000

# Lock the retention policy (IRREVERSIBLE)
gcloud storage buckets update gs://my-bucket \
  --lock-retention-policy

# Place a temporary hold on an object
gcloud storage objects update gs://my-bucket/important.pdf \
  --temporary-hold
```

**Soft Delete:**
- Objects are retained for a configurable period (7-90 days) after deletion
- Enables recovery of accidentally deleted objects
- Default: 7 days for new buckets
- Charged at the same rate as the object's storage class

**Exam tips:**
- **Bucket Lock is IRREVERSIBLE** -- this is a critical exam fact. Once locked, you cannot shorten or remove the retention policy.
- Lifecycle rules can transition classes (Standard -> Nearline -> Coldline -> Archive) but **never in reverse**
- Retention policies prevent deletion; lifecycle rules automate transitions and cleanup
- Soft delete is the first line of defense against accidental deletion

**Docs:**
- [Object lifecycle management](https://cloud.google.com/storage/docs/lifecycle)
- [Retention policies and Bucket Lock](https://cloud.google.com/storage/docs/bucket-lock)
- [Soft delete](https://cloud.google.com/storage/docs/soft-delete)

---

### Data Growth Planning

#### Database Scaling Patterns

| Database | Scaling mechanism | Key considerations |
|----------|------------------|-------------------|
| **Cloud Spanner** | Add/remove **processing units** (1000 PU = 1 node) | Horizontally scales reads AND writes; no downtime during scaling; min 100 PU for regional, 100 PU per region for multi-region |
| **BigQuery** | **Slots** (units of compute) | On-demand (per-query pricing, auto-allocated) or **editions** (Standard, Enterprise, Enterprise Plus) with autoscaling slot commitments |
| **Cloud Bigtable** | Add/remove **nodes** per cluster | Min 1 node; scales linearly (1 node ~ 10K rows/sec reads, 10K rows/sec writes); rebalancing takes time |
| **Cloud SQL** | Vertical scaling (machine type) + read replicas | Cannot scale writes horizontally; HA with regional failover; max instance size limits exist |
| **AlloyDB** | Primary instance (vertical) + read pool (horizontal) | PostgreSQL compatible; columnar engine for analytics; auto-scales read pool; cross-region replicas |
| **Firestore** | Automatic | Fully serverless; scales automatically; watch for hotspot patterns in key design |

**Spanner scaling details:**
```bash
# Scale Spanner instance
gcloud spanner instances update my-instance \
  --processing-units=3000

# Or use nodes (1 node = 1000 PU)
gcloud spanner instances update my-instance \
  --nodes=5

# Enable autoscaling
gcloud spanner instances update my-instance \
  --autoscaling-min-processing-units=1000 \
  --autoscaling-max-processing-units=5000 \
  --autoscaling-high-priority-cpu-target=65
```

**BigQuery editions and slots:**

| Edition | Commitment | Autoscaling | Cost model |
|---------|-----------|-------------|-----------|
| **On-demand** | None | Auto (no control) | Per TB scanned |
| **Standard** | 1 year or 3 year | Baseline + autoscale slots | Per-slot-hour |
| **Enterprise** | 1 year or 3 year | Baseline + autoscale slots | Per-slot-hour (lower) |
| **Enterprise Plus** | 1 year or 3 year | Baseline + autoscale slots | Per-slot-hour (lowest), CMEK, advanced security |

```bash
# Create a BigQuery reservation (Enterprise edition)
bq mk --reservation \
  --project_id=my-project \
  --location=us \
  --reservation_id=my-reservation \
  --slots=500 \
  --edition=ENTERPRISE
```

**Exam tips:**
- Spanner is the only relational DB that scales **writes horizontally** -- key differentiator
- BigQuery on-demand = simple (pay per query). Editions = predictable cost with autoscaling slots.
- Bigtable scaling is linear: 2x nodes = 2x throughput (approximately), but rebalancing takes time
- Cloud SQL **cannot** scale writes horizontally -- if you need write scaling for relational, go to **Spanner** or **AlloyDB**
- AlloyDB is the "best of both worlds" for PostgreSQL: auto-scaling reads + columnar analytics engine

**Docs:**
- [Spanner scaling](https://cloud.google.com/spanner/docs/instances)
- [BigQuery editions](https://cloud.google.com/bigquery/docs/editions-intro)
- [Bigtable scaling](https://cloud.google.com/bigtable/docs/scaling)
- [AlloyDB overview](https://cloud.google.com/alloydb/docs/overview)

---

### Data Protection

#### Cloud SQL Backup and HA

| Feature | Description |
|---------|-------------|
| **Automated backups** | Daily backups, retained for up to 365 days; configurable backup window |
| **On-demand backups** | Manual snapshots; retained until explicitly deleted |
| **PITR** | Point-in-time recovery using write-ahead logs; restore to any point within retention window |
| **HA configuration** | Regional HA with automatic failover to standby in another zone (synchronous replication) |
| **Read replicas** | Async replicas for read scaling; cross-region replicas for DR |
| **Cascading replicas** | Replica of a replica; reduces load on primary |

```bash
# Enable HA on Cloud SQL
gcloud sql instances patch my-instance \
  --availability-type=REGIONAL

# Create a read replica
gcloud sql instances create my-replica \
  --master-instance-name=my-instance \
  --region=us-east1

# Enable PITR
gcloud sql instances patch my-instance \
  --enable-point-in-time-recovery \
  --retained-transaction-log-days=7

# Restore to a specific point in time
gcloud sql instances restore-backup my-instance \
  --restore-instance=my-restored \
  --backup-run-id=12345

# Perform PITR clone
gcloud sql instances clone my-instance my-clone \
  --point-in-time="2026-02-15T10:00:00Z"
```

#### Spanner Backup and DR

| Feature | Description |
|---------|-------------|
| **Backup** | Full database backup; retained for configurable period (max 1 year) |
| **Restore** | Restore backup to a new database (same or different instance) |
| **PITR** | Version GC policy controls how far back you can read (default 1 hour, max 7 days) |
| **Export/Import** | Export to GCS (Avro/CSV); import from GCS |
| **Multi-region** | Built-in replication across regions with automatic failover |

```bash
# Create a Spanner backup
gcloud spanner backups create my-backup \
  --instance=my-instance \
  --database=my-db \
  --expiration-date="2026-03-15T00:00:00Z" \
  --async

# Restore from backup
gcloud spanner databases restore --async \
  --destination-instance=my-instance \
  --destination-database=my-restored-db \
  --source-instance=my-instance \
  --source-backup=my-backup
```

#### Firestore Backup and DR

| Feature | Description |
|---------|-------------|
| **PITR** | Enabled by default; recover to any point within the last 7 days (1-hour granularity) |
| **Export** | Export to GCS as LevelDB-format files |
| **Import** | Import from GCS export files |
| **Managed backup and restore** | Scheduled backups with retention policies |

```bash
# Export Firestore to GCS
gcloud firestore export gs://my-bucket/firestore-export \
  --collection-ids=users,orders

# Import Firestore from GCS
gcloud firestore import gs://my-bucket/firestore-export
```

#### BigQuery Data Protection

| Feature | Description |
|---------|-------------|
| **Time travel** | Query data as it existed at any point in the last **7 days** (configurable 2-7 days) |
| **Fail-safe** | Additional 7 days after time travel window; Google-managed, not queryable, for disaster recovery |
| **Snapshots** | Table snapshot at a point in time; incremental storage (only stores diffs) |
| **Dataset replicas** | Cross-region dataset replication for DR |
| **Scheduled queries** | Automate data exports/copies on a schedule |

```bash
# Query data as it existed 1 hour ago (time travel)
SELECT * FROM `project.dataset.table`
  FOR SYSTEM_TIME AS OF TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)

# Create a table snapshot
bq cp --snapshot project:dataset.table project:dataset.table_snapshot

# Restore from snapshot
bq cp --restore project:dataset.table_snapshot project:dataset.table_restored
```

#### Cloud Storage Data Protection

| Feature | Description |
|---------|-------------|
| **Object versioning** | Keeps all versions of an object; non-current versions billed at storage class rate |
| **Soft delete** | Retains deleted objects for 7-90 days (configurable) |
| **Dual-region** | Synchronous replication to two regions for durability |
| **Multi-region** | Geo-redundant storage across a continent |
| **Retention policy** | Prevents deletion before retention period expires |
| **Object hold** | Prevents deletion regardless of retention policy |

```bash
# Enable versioning
gcloud storage buckets update gs://my-bucket --versioning

# List object versions
gcloud storage ls --all-versions gs://my-bucket/my-file

# Configure soft delete (30 days)
gcloud storage buckets update gs://my-bucket \
  --soft-delete-duration=30d
```

**Exam tips:**
- Cloud SQL PITR uses **write-ahead logs** -- make sure to set an appropriate log retention period
- Cloud SQL HA uses **synchronous replication** to a standby in another zone -- automatic failover
- Spanner multi-region provides the **highest availability** (99.999% SLA) of any relational database on Google Cloud
- BigQuery time travel is **7 days** by default (configurable 2-7 days). Fail-safe adds another 7 days but is Google-managed.
- GCS versioning and soft delete are complementary -- versioning keeps old versions, soft delete allows recovery of deleted objects
- For DR: Cloud SQL uses read replicas + PITR. Spanner uses multi-region. BigQuery uses dataset replicas + snapshots.

**Docs:**
- [Cloud SQL backups](https://cloud.google.com/sql/docs/mysql/backup-recovery/backups)
- [Spanner backup and restore](https://cloud.google.com/spanner/docs/backup)
- [Firestore PITR](https://cloud.google.com/firestore/docs/pitr)
- [BigQuery time travel](https://cloud.google.com/bigquery/docs/time-travel)
- [GCS object versioning](https://cloud.google.com/storage/docs/object-versioning)

---

## 2.3 Configuring Compute Systems

### Compute Resource Provisioning

#### Machine Families

| Family | Series | vCPUs | Use case |
|--------|--------|-------|----------|
| **General-purpose** | E2 | 0.25-32 | Cost-optimized; dev/test, small-medium workloads |
| | N2 | 2-128 | Balanced price/performance; most workloads |
| | N2D | 2-224 | AMD EPYC; better price/performance for many workloads |
| | T2D | 1-60 | AMD; scale-out workloads (web servers, containerized) |
| | C4 | 2-192 | Latest gen; highest per-core performance in general-purpose |
| **Compute-optimized** | C2 | 4-60 | HPC, gaming, single-thread performance |
| | C2D | 2-112 | AMD; HPC, high-performance computing |
| | H3 | 88 | HPC with high-bandwidth networking (200 Gbps) |
| **Memory-optimized** | M2 | 12-416 | SAP HANA, large in-memory databases |
| | M3 | 32-128 | Next-gen memory-optimized; in-memory analytics |
| **Accelerator-optimized** | A2 | 12-96 | NVIDIA A100 GPUs; ML training and inference |
| | A3 | 26-208 | NVIDIA H100 GPUs; largest ML training workloads |
| | G2 | 4-96 | NVIDIA L4 GPUs; inference, graphics, video |

**Architect decision criteria:**

| Requirement | Recommended series |
|-------------|-------------------|
| Lowest cost, variable workloads | E2 (uses dynamic resource management) |
| Balanced production workloads | N2 or N2D |
| Web serving at scale | T2D or C4 |
| HPC / scientific computing | C2, C2D, or H3 |
| SAP HANA, large in-memory DBs | M2 or M3 |
| ML training with GPUs | A2 (A100) or A3 (H100) |
| ML inference, video transcoding | G2 (L4) |

#### Managed Instance Groups (MIGs)

| Feature | Stateless MIG | Stateful MIG |
|---------|--------------|-------------|
| **Instance identity** | Instances are interchangeable | Each instance has persistent identity |
| **Persistent disks** | Not preserved on recreation | Preserved across updates/repairs |
| **IP addresses** | May change | Can be preserved |
| **Instance names** | Auto-generated | Configurable, preserved |
| **Use case** | Web servers, stateless APIs | Databases, legacy apps, ML training |
| **Autoscaling** | Full support | Supported (but typically fixed size) |

**MIG scope:**

| Type | Scope | Benefit |
|------|-------|---------|
| **Zonal MIG** | Single zone | Simplest; use when zone affinity is needed |
| **Regional MIG** | Across zones in a region | Higher availability; instances distributed across zones |

**Instance templates:**
- Define the machine type, boot disk, network, metadata, labels, and startup script
- Immutable -- create a new template for changes
- Used by MIGs, and referenced by autoscaler configurations

```bash
# Create an instance template
gcloud compute instance-templates create my-template \
  --machine-type=n2-standard-4 \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=50GB \
  --tags=http-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update && apt-get install -y nginx'

# Create a regional MIG
gcloud compute instance-groups managed create my-mig \
  --template=my-template \
  --size=3 \
  --region=us-central1 \
  --target-distribution-shape=EVEN

# Set autoscaling
gcloud compute instance-groups managed set-autoscaling my-mig \
  --region=us-central1 \
  --min-num-replicas=2 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.70 \
  --cool-down-period=90
```

#### Sole-Tenant Nodes

Sole-tenant nodes provide **dedicated physical servers** for your VMs -- no other customers share the hardware.

**When to use sole-tenant nodes:**
- **Licensing:** Bring-your-own-license (BYOL) for software like Windows Server, SQL Server, Oracle that requires per-core or per-socket licensing
- **Compliance:** Regulatory requirements for physical isolation (HIPAA, PCI-DSS)
- **Performance:** Workloads sensitive to noisy-neighbor effects

```bash
# Create a sole-tenant node template
gcloud compute sole-tenancy node-templates create my-node-template \
  --node-type=n2-node-80-640 \
  --region=us-central1

# Create a sole-tenant node group
gcloud compute sole-tenancy node-groups create my-node-group \
  --node-template=my-node-template \
  --target-size=2 \
  --zone=us-central1-a

# Schedule a VM on a sole-tenant node group
gcloud compute instances create my-vm \
  --node-group=my-node-group \
  --zone=us-central1-a \
  --machine-type=n2-standard-8
```

**Exam tips:**
- E2 is the **cheapest** general-purpose family -- uses dynamic resource management
- For ML training: A2 (A100 GPUs) or A3 (H100 GPUs). For inference: G2 (L4 GPUs).
- Regional MIGs spread instances across zones for **higher availability** -- preferred for production
- Sole-tenant nodes are for **BYOL licensing** and **compliance/isolation** -- not for general cost savings
- Stateful MIGs preserve disk and IP across recreations -- use for apps that need persistent identity
- Instance templates are **immutable** -- you create a new one and update the MIG to use it

**Docs:**
- [Machine families](https://cloud.google.com/compute/docs/machine-resource)
- [Managed instance groups](https://cloud.google.com/compute/docs/instance-groups)
- [Sole-tenant nodes](https://cloud.google.com/compute/docs/nodes/sole-tenant-nodes)
- [Instance templates](https://cloud.google.com/compute/docs/instance-templates)

---

### Compute Volatility Configuration

#### Spot VMs

Spot VMs are spare Compute Engine capacity available at a **60-91% discount** off on-demand pricing.

| Feature | Spot VMs | Standard VMs |
|---------|---------|-------------|
| **Discount** | 60-91% | None (on-demand) / CUDs / SUDs |
| **Max runtime** | No hard limit (can be preempted anytime) | Unlimited |
| **Preemption notice** | 30-second warning | N/A |
| **Availability SLA** | None | Per service SLA |
| **Live migration** | No | Yes |
| **Auto-restart** | No | Yes |
| **Termination action** | STOP or DELETE | N/A |

**When to use Spot VMs:**
- Batch processing, data pipelines (Dataproc, Dataflow)
- CI/CD build agents
- Fault-tolerant distributed workloads
- ML training with checkpointing
- Rendering, transcoding, genomics
- Any workload that can handle interruption

**When NOT to use Spot VMs:**
- User-facing production services
- Stateful workloads without checkpointing
- Workloads with strict completion deadlines
- Single-instance databases

```bash
# Create a Spot VM
gcloud compute instances create my-spot-vm \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP \
  --machine-type=n2-standard-8 \
  --zone=us-central1-a

# Use Spot VMs in a MIG
gcloud compute instance-templates create spot-template \
  --provisioning-model=SPOT \
  --instance-termination-action=DELETE \
  --machine-type=n2-standard-4

gcloud compute instance-groups managed create spot-mig \
  --template=spot-template \
  --size=10 \
  --zone=us-central1-a
```

**Cost optimization strategy hierarchy:**

| Strategy | Discount | Commitment | Flexibility |
|----------|---------|------------|-------------|
| **Spot VMs** | 60-91% | None (can be preempted) | Highest |
| **3-year CUD** | Up to 57% | 3 years | Lowest |
| **1-year CUD** | Up to 37% | 1 year | Low |
| **Sustained use discount (SUD)** | Up to 30% | None (automatic) | High |
| **On-demand** | 0% | None | Highest |

**Exam tips:**
- Spot VMs replaced Preemptible VMs -- Spot has **no 24-hour limit** (Preemptible had a 24-hour cap)
- Spot VMs **cannot live migrate** and **cannot auto-restart** on host maintenance
- For batch on Dataproc, using **Spot VMs for workers** is a common cost optimization pattern
- The exam may describe a workload that "can tolerate interruption" -- this signals **Spot VMs**
- CUDs and SUDs apply to **on-demand** VMs, not Spot VMs

**Docs:**
- [Spot VMs](https://cloud.google.com/compute/docs/instances/spot)
- [Committed use discounts](https://cloud.google.com/compute/docs/instances/committed-use-discounts-overview)

---

### Cloud-native Network Config for Compute

#### Compute Engine Networking

| Feature | Description |
|---------|-------------|
| **Multiple NICs** | Up to 8 NICs per VM, each in a different VPC; enables multi-VPC routing |
| **Alias IP ranges** | Assign additional IP ranges to a NIC; used for containers on VMs |
| **Packet mirroring** | Mirror network traffic to a collector for inspection (IDS, forensics) |
| **Network tags** | Apply firewall rules to VMs by tag (not IP) |
| **Service accounts** | Firewall rules can target VMs by service account |

```bash
# Create a VM with multiple NICs
gcloud compute instances create multi-nic-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --network-interface=network=vpc-1,subnet=subnet-1 \
  --network-interface=network=vpc-2,subnet=subnet-2

# Create a packet mirroring policy
gcloud compute packet-mirrorings create my-mirror \
  --region=us-central1 \
  --network=my-vpc \
  --collector-ilb=my-ids-ilb \
  --mirrored-subnets=my-subnet
```

#### GKE Networking

**VPC-native clusters** (recommended, required for Autopilot):
- Pods and Services get IPs from VPC secondary ranges
- Enables direct pod-to-pod communication across VPCs (with peering)
- Required for Private Google Access from pods, VPC Service Controls, and most networking features

| GKE networking concept | Description |
|----------------------|-------------|
| **ClusterIP Service** | Internal-only; accessible within the cluster |
| **NodePort Service** | Exposes on each node's IP at a static port |
| **LoadBalancer Service** | Creates a Google Cloud load balancer (Network LB) |
| **Ingress** | L7 load balancing (creates Application LB) |
| **Gateway API** | Next-gen of Ingress; multi-cluster, traffic splitting |
| **Network Policies** | Kubernetes-native firewall rules for pods |

```bash
# Create a VPC-native GKE cluster
gcloud container clusters create my-cluster \
  --region=us-central1 \
  --enable-ip-alias \
  --cluster-ipv4-cidr=/16 \
  --services-ipv4-cidr=/22 \
  --network=my-vpc \
  --subnetwork=my-subnet

# Create a multi-cluster Gateway (GKE Enterprise)
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: external-http
  namespace: default
spec:
  gatewayClassName: gke-l7-global-external-managed-mc
  listeners:
  - name: http
    port: 80
    protocol: HTTP
EOF
```

#### Serverless VPC Access

Allows **Cloud Run, Cloud Functions, and App Engine** to connect to resources in your VPC.

**Two options:**
1. **Serverless VPC Access connector:** Dedicated connector (powered by e2-micro VMs); supports throughput up to 1 Gbps
2. **Direct VPC egress (Cloud Run only):** Routes traffic directly through VPC without a connector; simpler, supports higher throughput

```bash
# Create a Serverless VPC Access connector
gcloud compute networks vpc-access connectors create my-connector \
  --region=us-central1 \
  --network=my-vpc \
  --range=10.8.0.0/28 \
  --min-instances=2 \
  --max-instances=10

# Deploy Cloud Run with VPC connector
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image \
  --vpc-connector=my-connector \
  --vpc-egress=all-traffic \
  --region=us-central1

# Deploy Cloud Run with Direct VPC egress
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image \
  --network=my-vpc \
  --subnet=my-subnet \
  --vpc-egress=all-traffic \
  --region=us-central1
```

**Exam tips:**
- Multiple NICs on a VM = each NIC in a **different VPC** -- enables routing between VPCs without peering
- GKE **VPC-native clusters** are the default and recommended -- pods get routable VPC IPs
- **Gateway API** is replacing Ingress for advanced traffic management (multi-cluster, traffic splitting)
- Serverless VPC Access connector uses a **/28 subnet** -- plan your IP space accordingly
- **Direct VPC egress** (Cloud Run) is simpler and higher throughput than a connector -- prefer it for new deployments

**Docs:**
- [Compute Engine networking](https://cloud.google.com/compute/docs/networking)
- [GKE networking overview](https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview)
- [Serverless VPC Access](https://cloud.google.com/vpc/docs/configure-serverless-vpc-access)
- [Gateway API on GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/gateway-api)

---

### Infrastructure Orchestration and Patch Management

#### VM Manager

VM Manager provides **OS-level management** for Compute Engine fleets.

| Feature | Description |
|---------|-------------|
| **OS Patch Management** | Automate OS patching across VMs; schedule patch jobs; compliance reporting |
| **OS Configuration Management** | Enforce desired-state configurations using OS policies (install packages, manage files) |
| **OS Inventory** | Collect and query installed packages, OS versions, system updates across fleet |

**OS Patch Management:**
```bash
# Create a patch job (patch all VMs with a specific label)
gcloud compute os-config patch-jobs execute \
  --instance-filter-names="zones/us-central1-a/instances/vm-1" \
  --duration=3600s \
  --reboot-config=DEFAULT

# Schedule recurring patch deployment
gcloud compute os-config patch-deployments create monthly-patch \
  --instance-filter-all \
  --recurring-schedule-frequency=MONTHLY \
  --recurring-schedule-time-of-day-hours=2 \
  --recurring-schedule-time-of-day-minutes=0 \
  --recurring-schedule-time-zone="America/Chicago" \
  --recurring-schedule-monthly-week-day-of-month-day-of-week=SUNDAY \
  --recurring-schedule-monthly-week-day-of-month-week-offset=FIRST
```

#### MIG Update Policies

| Policy | Behavior | Use case |
|--------|----------|----------|
| **PROACTIVE** | Immediately replaces instances when template changes | Need immediate rollout |
| **OPPORTUNISTIC** | Replaces instances only when they are recreated (e.g., by autoscaler or manually) | Minimize disruption; gradual rollout |

**Rolling update parameters:**

| Parameter | Description |
|-----------|-------------|
| `max-surge` | Max number of new instances created above target size during update |
| `max-unavailable` | Max number of instances that can be unavailable during update |
| `min-ready-sec` | Time to wait after instance is created before marking it as updated |
| `replacement-method` | `substitute` (create new, delete old) or `recreate` (delete first, then create) |

```bash
# Update a MIG with rolling update
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=new-template \
  --region=us-central1 \
  --max-surge=3 \
  --max-unavailable=0 \
  --min-ready-sec=60

# Canary update (80% old, 20% new)
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=old-template \
  --canary-version=template=new-template,target-size=20% \
  --region=us-central1
```

**Exam tips:**
- **PROACTIVE** = immediate rollout. **OPPORTUNISTIC** = lazy rollout (when instances happen to be recreated).
- `max-surge=N, max-unavailable=0` = **zero-downtime** rolling update (always N extra instances running during update)
- VM Manager OS patch management is the answer for "how to ensure all VMs are patched" at scale
- Canary updates allow testing new templates on a percentage of instances before full rollout

**Docs:**
- [VM Manager overview](https://cloud.google.com/compute/docs/vm-manager)
- [OS patch management](https://cloud.google.com/compute/docs/os-patch-management)
- [MIG update policies](https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups)

---

### Container Orchestration

#### GKE Standard vs Autopilot

| Feature | GKE Standard | GKE Autopilot |
|---------|-------------|---------------|
| **Node management** | You manage nodes (create/configure node pools) | Google manages nodes fully |
| **Billing** | Pay for nodes (VMs) whether utilized or not | Pay per pod (CPU, memory, ephemeral storage) |
| **Node configuration** | Full control (machine type, GPU, taints) | Google selects optimal config |
| **GPU/TPU support** | Full control via node pools | Supported (request in pod spec) |
| **Security posture** | You harden nodes | Hardened by default (shielded nodes, COS, no SSH) |
| **Max pods per node** | Configurable (default 110) | Google-managed |
| **DaemonSets** | Full support | Limited (allowed list) |
| **Privileged containers** | Allowed | Not allowed |
| **Node auto-provisioning** | Optional | Built-in |
| **Recommended for** | Full control, specific node requirements | Most workloads; reduced ops overhead |

**Decision: Standard vs Autopilot**

| Scenario | Choose |
|----------|--------|
| Need privileged containers or custom kernel modules | Standard |
| Want lowest operational overhead | Autopilot |
| Specific GPU/TPU node pool configurations | Standard (more control) or Autopilot |
| Security-first with minimal configuration | Autopilot |
| Need DaemonSets for monitoring agents | Standard |
| Pay only for what pods use | Autopilot |
| Need Windows containers | Standard |

#### GKE Enterprise Features

| Feature | Description |
|---------|-------------|
| **Fleet management** | Manage multiple clusters as a single fleet with consistent policies |
| **Multi-cluster Services (MCS)** | Expose services across clusters in a fleet |
| **Config Sync** | GitOps: automatically sync Kubernetes configs from Git repos to clusters |
| **Policy Controller** | Enforce policies (based on OPA Gatekeeper) across fleet clusters |
| **Service Mesh (Anthos Service Mesh)** | Managed Istio-based service mesh for observability, security, traffic management |
| **Connect Gateway** | Access fleet clusters from anywhere via Google Cloud identity |
| **Binary Authorization** | Enforce that only trusted container images are deployed |

```bash
# Register a cluster to a fleet
gcloud container fleet memberships register my-cluster \
  --gke-cluster=us-central1/my-cluster \
  --enable-workload-identity

# Enable Config Sync
gcloud beta container fleet config-management apply \
  --membership=my-cluster \
  --config=config-sync.yaml

# Enable Policy Controller
gcloud container fleet policycontroller enable \
  --membership=my-cluster
```

#### Workload Identity

Workload Identity is the recommended way for GKE pods to authenticate to Google Cloud APIs.

**How it works:**
1. A Kubernetes service account (KSA) is linked to a Google service account (GSA)
2. Pods running as the KSA can authenticate as the GSA
3. No need for service account keys -- uses the GKE metadata server

```bash
# Enable Workload Identity on cluster
gcloud container clusters update my-cluster \
  --region=us-central1 \
  --workload-pool=my-project.svc.id.goog

# Create IAM binding (KSA -> GSA)
gcloud iam service-accounts add-iam-policy-binding my-gsa@my-project.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="serviceAccount:my-project.svc.id.goog[my-namespace/my-ksa]"

# Annotate the Kubernetes service account
kubectl annotate serviceaccount my-ksa \
  --namespace=my-namespace \
  iam.gke.io/gcp-service-account=my-gsa@my-project.iam.gserviceaccount.com
```

**Exam tips:**
- **Autopilot** is the default recommendation for new clusters -- choose Standard only when you need features Autopilot doesn't support
- GKE Enterprise features (fleet, Config Sync, Policy Controller) are for **multi-cluster governance** -- key for large organizations
- **Workload Identity** replaces service account key files in GKE -- always prefer it over mounting key files
- **Config Sync** = GitOps for GKE. **Policy Controller** = OPA Gatekeeper for GKE. Know the difference.
- **Binary Authorization** enforces that only signed/attested images can be deployed -- pairs with Artifact Registry vulnerability scanning
- Autopilot billing is per-pod, Standard billing is per-node -- this affects cost optimization strategies

**Docs:**
- [GKE Autopilot](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview)
- [GKE Enterprise](https://cloud.google.com/kubernetes-engine/enterprise/docs/concepts/overview)
- [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
- [Config Sync](https://cloud.google.com/kubernetes-engine/enterprise/docs/config-sync-overview)
- [Policy Controller](https://cloud.google.com/kubernetes-engine/enterprise/docs/concepts/policy-controller)

---

### Serverless Computing

#### Cloud Run

| Feature | Cloud Run Services | Cloud Run Jobs |
|---------|-------------------|----------------|
| **Purpose** | HTTP request handling | Batch/background tasks |
| **Trigger** | HTTP requests, Pub/Sub, Eventarc | Manual, scheduled, Workflows |
| **Scaling** | 0 to N instances (request-based) | 0 to N tasks (parallel execution) |
| **Max timeout** | 60 min (default), configurable | 24 hours |
| **Concurrency** | Up to 1000 requests/instance | 1 task per instance |
| **Billing** | CPU + memory per request duration | CPU + memory per task duration |

**Key Cloud Run configuration:**

| Setting | Description | Architect consideration |
|---------|-------------|----------------------|
| `min-instances` | Keep instances warm (avoid cold starts) | Set > 0 for latency-sensitive services; costs more |
| `max-instances` | Cap scaling | Prevent runaway costs; protect downstream services |
| `concurrency` | Requests per instance | Higher = more efficient; lower = more isolation |
| `cpu-throttling` | CPU allocated only during requests (default) vs always | Always-on for background processing |
| `execution-environment` | gen1 (gVisor) vs gen2 (Linux VM) | gen2 for full Linux compatibility |

```bash
# Deploy a Cloud Run service
gcloud run deploy my-service \
  --image=us-docker.pkg.dev/my-project/my-repo/my-image:latest \
  --region=us-central1 \
  --platform=managed \
  --min-instances=1 \
  --max-instances=100 \
  --concurrency=80 \
  --cpu=2 \
  --memory=1Gi \
  --no-cpu-throttling \
  --allow-unauthenticated

# Create a Cloud Run job
gcloud run jobs create my-job \
  --image=us-docker.pkg.dev/my-project/my-repo/my-job:latest \
  --region=us-central1 \
  --tasks=10 \
  --parallelism=5 \
  --max-retries=3 \
  --task-timeout=3600

# Execute the job
gcloud run jobs execute my-job --region=us-central1
```

#### Cloud Run Functions (2nd Gen)

Cloud Run functions (formerly Cloud Functions 2nd gen) are built on Cloud Run and provide an **event-driven** programming model.

**Key facts:**
- Built on Cloud Run infrastructure (inherits all Cloud Run features)
- Triggered by **Eventarc** events from 100+ Google Cloud sources
- Also supports HTTP triggers
- Supports **concurrency** (multiple requests per function instance) -- unlike 1st gen
- Max timeout: 60 minutes (vs 9 minutes for 1st gen)
- Supports **larger instances** (up to 16 GiB memory, 4 vCPUs)

#### Eventarc

Eventarc is the **unified eventing layer** for Google Cloud.

**Event sources:**
- Google Cloud services (via Cloud Audit Logs or direct)
- Pub/Sub messages
- Third-party sources (via Pub/Sub)
- Custom applications

**Event destinations:**
- Cloud Run services
- Cloud Run functions
- GKE services
- Workflows

```bash
# Create an Eventarc trigger (Cloud Storage -> Cloud Run)
gcloud eventarc triggers create my-trigger \
  --location=us-central1 \
  --destination-run-service=my-service \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.storage.object.v1.finalized" \
  --event-filters="bucket=my-bucket" \
  --service-account=my-sa@my-project.iam.gserviceaccount.com

# Create an Eventarc trigger (Audit Log -> Cloud Run)
gcloud eventarc triggers create audit-trigger \
  --location=us-central1 \
  --destination-run-service=my-service \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.audit.log.v1.written" \
  --event-filters="serviceName=bigquery.googleapis.com" \
  --event-filters="methodName=google.cloud.bigquery.v2.TableService.InsertTable" \
  --service-account=my-sa@my-project.iam.gserviceaccount.com
```

**Serverless decision matrix:**

| Criteria | Cloud Run (service) | Cloud Run (job) | Cloud Run functions |
|----------|-------------------|-----------------|-------------------|
| HTTP API serving | Best fit | No | Possible (simple APIs) |
| Event processing | Via Eventarc | Via Workflows | Best fit (event-native) |
| Batch processing | Possible | Best fit | No |
| Long-running tasks | Up to 60 min | Up to 24 hours | Up to 60 min |
| Container flexibility | Full Dockerfile | Full Dockerfile | Source-based (limited) |
| Concurrency | Up to 1000 | 1 per task | Configurable |

**Exam tips:**
- Cloud Run **services** = request-driven. Cloud Run **jobs** = task-driven (batch).
- `min-instances > 0` eliminates cold starts but increases cost -- architect trade-off between latency and cost
- Cloud Run functions 2nd gen is built on Cloud Run -- know that it inherits Cloud Run features (concurrency, longer timeouts)
- **Eventarc** is the answer when the exam says "trigger a service when X happens in Google Cloud"
- Cloud Run with `--no-cpu-throttling` is needed for background work (CPU is allocated even when not handling requests)
- For exam: "event-driven, single function, minimal code" = **Cloud Run functions**. "Container serving HTTP" = **Cloud Run service**. "Batch container tasks" = **Cloud Run job**.

**Docs:**
- [Cloud Run services](https://cloud.google.com/run/docs/overview/what-is-cloud-run)
- [Cloud Run jobs](https://cloud.google.com/run/docs/create-jobs)
- [Cloud Run functions](https://cloud.google.com/functions/docs/concepts/overview)
- [Eventarc](https://cloud.google.com/eventarc/docs/overview)

---

## 2.4 Leveraging Vertex AI for End-to-End ML Workflows

> **Note:** AI/ML content is significantly expanded in the 2025/2026 PCA exam. Expect 2-4 questions specifically on Vertex AI and ML infrastructure.

### Vertex AI Pipelines

Vertex AI Pipelines orchestrate ML workflows as **directed acyclic graphs (DAGs)** of containerized steps.

**Key concepts:**

| Concept | Description |
|---------|-------------|
| **Pipeline** | A DAG of components that defines an ML workflow |
| **Component** | A self-contained step (container) that performs one task (data prep, training, evaluation) |
| **Artifact** | Input/output data (datasets, models, metrics) tracked by Vertex ML Metadata |
| **Run** | A single execution of a pipeline |
| **Schedule** | Recurring pipeline execution on a cron schedule |

**Pipeline frameworks supported:**
- **Kubeflow Pipelines (KFP) v2:** Primary SDK for building pipelines on Vertex AI
- **TFX (TensorFlow Extended):** End-to-end TensorFlow ML pipelines

**Common pipeline pattern:**
```
Data ingestion -> Data validation -> Data transformation ->
Model training -> Model evaluation -> Model deployment (conditional)
```

```bash
# Submit a pipeline run
gcloud ai pipelines run create \
  --display-name=my-training-pipeline \
  --template-path=gs://my-bucket/pipeline.yaml \
  --region=us-central1 \
  --parameter-values='{"learning_rate": 0.01, "epochs": 100}'

# Schedule a pipeline
gcloud ai pipelines schedules create \
  --display-name=weekly-retrain \
  --template-path=gs://my-bucket/pipeline.yaml \
  --region=us-central1 \
  --cron="0 2 * * 0"
```

**Python SDK example (conceptual -- know the pattern for the exam):**
```python
from kfp import dsl
from google.cloud import aiplatform

@dsl.pipeline(name="training-pipeline")
def my_pipeline(project: str, region: str):
    # Step 1: Data preparation
    data_op = prepare_data(project=project)
    # Step 2: Training
    train_op = train_model(data=data_op.output)
    # Step 3: Conditional deployment
    with dsl.Condition(train_op.outputs["accuracy"] > 0.9):
        deploy_model(model=train_op.outputs["model"])

# Compile and submit
aiplatform.PipelineJob(
    display_name="my-pipeline",
    template_path="pipeline.yaml",
).run()
```

**Exam tips:**
- Vertex AI Pipelines = **orchestration** of ML lifecycle steps (not training itself)
- Pipelines use **Kubeflow Pipelines v2** SDK -- know this is the primary framework
- Each pipeline component is a **container** -- components are reusable across pipelines
- ML Metadata automatically tracks artifacts, lineage, and metrics

**Docs:**
- [Vertex AI Pipelines](https://cloud.google.com/vertex-ai/docs/pipelines/introduction)
- [Kubeflow Pipelines on Vertex AI](https://cloud.google.com/vertex-ai/docs/pipelines/build-pipeline)

---

### Vertex AI Data Integration

#### Feature Store

Vertex AI Feature Store provides a **centralized repository** for ML features with low-latency serving.

**Key concepts:**
- **Feature:** A measurable property used in ML models (e.g., `customer_age`, `transaction_count`)
- **Entity type:** A group of related features (e.g., `Customer`, `Product`)
- **Feature view:** A selection of features for a specific model
- **Online serving:** Low-latency reads for real-time prediction (backed by Bigtable)
- **Offline serving (batch):** Large-scale reads for training (backed by BigQuery)

**Why Feature Store matters for architects:**
- **Consistency:** Same features for training and serving (avoids training-serving skew)
- **Reuse:** Features computed once, shared across teams and models
- **Point-in-time correctness:** For training, retrieve features as they existed at a specific time
- **Freshness:** Feature values updated via streaming or batch ingestion

```bash
# Create a Feature Store (online store)
gcloud ai feature-online-stores create my-online-store \
  --region=us-central1 \
  --bigtable-auto-scaling-min-node-count=1 \
  --bigtable-auto-scaling-max-node-count=3

# Create a feature view
gcloud ai feature-views create my-feature-view \
  --feature-online-store=my-online-store \
  --region=us-central1 \
  --big-query-source-uri="bq://my-project.my-dataset.my-features_table" \
  --entity-id-columns="customer_id"
```

#### Datasets and Data Labeling

- **Managed datasets:** Upload data to Vertex AI for use in training (tabular, image, text, video)
- **Data labeling:** Human labeling service for unstructured data
- **BigQuery integration:** Directly reference BigQuery tables as training data sources
- **GCS integration:** Reference GCS files for image, text, and video datasets

**Exam tips:**
- Feature Store prevents **training-serving skew** -- a common ML problem where features differ between training and production
- Feature Store online serving uses **Bigtable** under the hood for low-latency lookups
- BigQuery is the primary data source for tabular ML workflows on Vertex AI

**Docs:**
- [Vertex AI Feature Store](https://cloud.google.com/vertex-ai/docs/featurestore/overview)
- [Managed datasets](https://cloud.google.com/vertex-ai/docs/training/using-managed-datasets)

---

### AI Hypercomputer and Accelerators

#### Cloud TPUs

| TPU version | Chip | TF-ops | HBM | Use case |
|------------|------|--------|-----|----------|
| **TPU v5e** | Custom | Cost-efficient | 16 GB/chip | Training and inference for medium models |
| **TPU v5p** | Custom | Highest performance | 95 GB/chip | Largest model training (LLMs, foundation models) |
| **TPU v4** | Custom | Previous gen | 32 GB/chip | Large-scale training |

**TPU key concepts:**
- **TPU Pod:** A group of TPU chips connected by high-speed interconnect (ICI)
- **TPU slice:** A subset of a TPU Pod allocated to a single workload
- **Multislice training:** Distribute training across multiple TPU slices connected via data center network (DCN) for the largest models
- **Queued resources:** Request TPU capacity that may not be immediately available; gets provisioned when capacity frees up

#### GPU Options

| GPU | Series | Use case | Key specs |
|-----|--------|----------|-----------|
| **NVIDIA H100** | A3 | Largest LLM training, HPC | 80 GB HBM3, NVLink |
| **NVIDIA A100** | A2 | ML training, HPC, inference | 40/80 GB HBM2e |
| **NVIDIA L4** | G2 | Inference, video, graphics | 24 GB GDDR6, power-efficient |
| **NVIDIA T4** | N1 + T4 | Cost-effective inference | 16 GB GDDR6 |

#### GPUs vs TPUs Decision

| Criteria | GPUs | TPUs |
|----------|------|------|
| **Framework support** | PyTorch, TensorFlow, JAX, all frameworks | TensorFlow, JAX, PyTorch/XLA |
| **Model type** | Any model architecture | Best for dense matrix operations (transformers, CNNs) |
| **Custom ops** | Full CUDA support | Limited custom operation support |
| **Availability** | More regions, on-demand + Spot | Limited regions, queued resources |
| **Cost at scale** | Higher per-FLOP | Lower per-FLOP for compatible workloads |
| **Ecosystem** | Broadest ecosystem (CUDA, cuDNN) | Google-specific ecosystem (XLA, JAX) |
| **Inference** | L4/T4 for cost-effective inference | v5e for high-throughput inference |

```bash
# Create a TPU node
gcloud compute tpus tpu-vm create my-tpu \
  --zone=us-central2-b \
  --accelerator-type=v5litepod-4 \
  --version=tpu-ubuntu2204-base

# Create a VM with GPUs
gcloud compute instances create gpu-vm \
  --zone=us-central1-c \
  --machine-type=a2-highgpu-1g \
  --accelerator=type=nvidia-tesla-a100,count=1 \
  --maintenance-policy=TERMINATE

# Create GKE node pool with GPUs
gcloud container node-pools create gpu-pool \
  --cluster=my-cluster \
  --region=us-central1 \
  --machine-type=g2-standard-4 \
  --accelerator=type=nvidia-l4,count=1 \
  --num-nodes=2
```

**Exam tips:**
- **TPUs** = best for large-scale training of transformer-based models (LLMs) using TensorFlow or JAX
- **GPUs** = more flexible, support all frameworks, better for custom model architectures
- **L4 GPUs (G2)** are the go-to for cost-effective **inference**
- **H100 GPUs (A3)** are for the **largest training** jobs when you need GPU (not TPU)
- **Multislice TPU training** spans multiple pods -- the answer when "training a very large model that doesn't fit on a single TPU slice"
- `--maintenance-policy=TERMINATE` is required for GPU VMs (they cannot live migrate)
- **Queued resources** for TPUs = capacity isn't guaranteed immediately, but you get in line

**Docs:**
- [Cloud TPU overview](https://cloud.google.com/tpu/docs/intro-to-tpu)
- [GPU platforms](https://cloud.google.com/compute/docs/gpus)
- [AI Hypercomputer](https://cloud.google.com/blog/products/compute/introducing-cloud-tpu-v5p-and-ai-hypercomputer)

---

### Optimizing for Different Consumption Models

#### Prediction Serving Options

| Option | Latency | Cost model | Use case |
|--------|---------|-----------|----------|
| **Online prediction (dedicated endpoint)** | Low (ms) | Pay for always-on compute (min 1 replica) | Production real-time serving |
| **Online prediction (serverless endpoint)** | Low-medium | Pay per prediction | Variable traffic, cost-sensitive |
| **Batch prediction** | High (minutes-hours) | Pay for batch job compute | Bulk scoring, periodic predictions |

**Dedicated vs serverless endpoints:**

| Feature | Dedicated endpoint | Serverless endpoint |
|---------|-------------------|-------------------|
| **Min replicas** | 1 (always running) | 0 (scales to zero) |
| **Cold start** | None (always warm) | Possible (when scaling from 0) |
| **GPU support** | Yes | Limited |
| **Custom containers** | Yes | Yes |
| **Cost at low traffic** | Higher (always-on) | Lower (pay per prediction) |
| **Cost at high traffic** | Lower (predictable) | Higher (per-prediction adds up) |

```bash
# Deploy a model to a dedicated endpoint
gcloud ai endpoints deploy-model my-endpoint \
  --model=my-model \
  --display-name=my-deployment \
  --machine-type=n1-standard-4 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --min-replica-count=1 \
  --max-replica-count=5 \
  --region=us-central1

# Run batch prediction
gcloud ai batch-prediction-jobs create \
  --model=my-model \
  --input-path=gs://my-bucket/input/ \
  --output-path=gs://my-bucket/output/ \
  --region=us-central1 \
  --machine-type=n1-standard-4
```

#### Model Optimization Techniques

| Technique | Description | Trade-off |
|-----------|-------------|-----------|
| **Quantization** | Reduce precision (FP32 -> INT8/FP16) | Smaller model, faster inference; slight accuracy loss |
| **Distillation** | Train a smaller "student" model from a larger "teacher" | Much smaller model; may lose nuanced capabilities |
| **Pruning** | Remove low-importance weights | Smaller model; may need retraining |
| **Caching** | Cache frequent predictions | Reduces compute; stale results for dynamic inputs |

**Exam tips:**
- **Dedicated endpoints** for production real-time serving with predictable traffic
- **Serverless endpoints** for variable/unpredictable traffic or cost-sensitive serving
- **Batch prediction** when you don't need real-time results (overnight scoring, weekly reports)
- Quantization and distillation are key techniques for **reducing serving costs** without retraining from scratch
- The exam may present a scenario with "variable traffic patterns" -- the answer is often **serverless endpoint** or **autoscaling dedicated endpoint**

**Docs:**
- [Vertex AI Prediction](https://cloud.google.com/vertex-ai/docs/predictions/overview)
- [Batch prediction](https://cloud.google.com/vertex-ai/docs/predictions/get-batch-predictions)
- [Model optimization](https://cloud.google.com/vertex-ai/docs/training/model-optimization)

---

### Running Large-Scale AI Model Trainings

#### Custom Training Jobs

Vertex AI supports multiple approaches to training:

| Approach | Description | Use case |
|----------|-------------|----------|
| **AutoML** | Automated model training (no code) | Tabular, image, text, video; quick baselines |
| **Custom training** | Your own training code in a container | Full control over model architecture and training loop |
| **Custom training (pre-built container)** | Your code + Google-provided container (TF, PyTorch, etc.) | Common frameworks without Dockerfile management |
| **Hyperparameter tuning** | Automated search over hyperparameter space | Optimize model performance without manual experimentation |

**Distributed training strategies:**

| Strategy | Description | When to use |
|----------|-------------|-------------|
| **Data parallelism** | Same model replicated across workers; data split among them | Most common; works well when model fits on one GPU/TPU |
| **Model parallelism** | Model split across workers | Very large models that don't fit on one device |
| **Pipeline parallelism** | Model layers split across devices in a pipeline | Very deep models with many layers |

```bash
# Submit a custom training job
gcloud ai custom-jobs create \
  --display-name=my-training-job \
  --region=us-central1 \
  --worker-pool-spec=machine-type=a2-highgpu-1g,replica-count=4,accelerator-type=NVIDIA_TESLA_A100,accelerator-count=1,container-image-uri=us-docker.pkg.dev/my-project/my-repo/train:latest

# Submit a hyperparameter tuning job
gcloud ai hp-tuning-jobs create \
  --display-name=hp-tuning \
  --region=us-central1 \
  --max-trial-count=20 \
  --parallel-trial-count=5 \
  --config=hp_config.yaml
```

#### Vertex AI Experiments

Track and compare training runs for reproducibility and model selection.

**Key features:**
- Log metrics, parameters, and artifacts for each training run
- Compare runs side-by-side in the console
- Integrates with TensorBoard for visualization
- Tracks lineage: which data, code, and parameters produced which model

**Exam tips:**
- **AutoML** = fastest path to a model, no ML expertise required. **Custom training** = full control.
- For the exam, if the scenario mentions "data scientists who want full control" -> **custom training**. If "business analysts who need a model quickly" -> **AutoML**.
- **Data parallelism** is the default distributed strategy -- model is replicated, data is split
- **Model parallelism** is needed when a model is too large for a single GPU -- think LLMs with billions of parameters
- Vertex AI Experiments = **experiment tracking** -- compare different training runs to find the best model

**Docs:**
- [Custom training](https://cloud.google.com/vertex-ai/docs/training/create-custom-job)
- [Hyperparameter tuning](https://cloud.google.com/vertex-ai/docs/training/hyperparameter-tuning-overview)
- [Distributed training](https://cloud.google.com/vertex-ai/docs/training/distributed-training)
- [Vertex AI Experiments](https://cloud.google.com/vertex-ai/docs/experiments/intro-vertex-ai-experiments)

---

## 2.5 Configuring Prebuilt Solutions or APIs with Vertex AI

### Google AI APIs (Pre-trained Models)

Google provides production-ready AI APIs that require **no ML expertise** -- just send data and get predictions.

| API | Input | Output | Use case |
|-----|-------|--------|----------|
| **Vision AI** | Images | Labels, objects, faces, text (OCR), logos, landmarks | Image classification, content moderation, OCR |
| **Video Intelligence** | Videos | Labels, shot changes, object tracking, text, person detection | Video analysis, content moderation |
| **Natural Language** | Text | Entities, sentiment, syntax, categories, moderation | Text analysis, content classification |
| **Speech-to-Text** | Audio | Transcribed text | Voice interfaces, transcription, subtitles |
| **Text-to-Speech** | Text | Audio | Voice assistants, accessibility, IVR |
| **Translation** | Text | Translated text | Localization, multilingual apps |
| **Document AI** | Documents (PDF, images) | Structured data (entities, tables, forms) | Invoice processing, form extraction, document classification |

**When to use pre-trained APIs vs custom models:**

| Scenario | Choose |
|----------|--------|
| Standard use case (OCR, sentiment, translation) | Pre-trained API |
| Domain-specific vocabulary or entities | **AutoML** custom model or fine-tuned API |
| Unique model architecture needed | **Custom training** on Vertex AI |
| Need to process sensitive data on-premises | Not APIs (use custom model + on-prem serving) |

```bash
# Detect text in an image (Vision AI)
gcloud ml vision detect-text gs://my-bucket/receipt.jpg

# Transcribe audio (Speech-to-Text)
gcloud ml speech recognize gs://my-bucket/audio.flac \
  --language-code=en-US

# Analyze sentiment (Natural Language)
gcloud ml language analyze-sentiment --content="This product is amazing!"

# Translate text
gcloud ml translate translate-text \
  --content="Hello world" \
  --target-language=es
```

**Document AI processors:**
- **General processors:** Form parser, OCR, document splitter
- **Specialized processors:** Invoice parser, receipt parser, ID parser, W-2 parser, contract parser
- **Custom processors:** Train on your own document types
- **Human-in-the-loop:** Review and correct Document AI output

**Exam tips:**
- Pre-trained APIs are the **fastest and cheapest** path when your use case matches -- always consider them first
- **Document AI** is the answer for "extract structured data from invoices/forms/receipts"
- Vision AI supports **batch processing** for large volumes of images
- For domain-specific language (medical, legal), consider **AutoML** custom models on top of pre-trained APIs
- Speech-to-Text supports **streaming recognition** for real-time transcription

**Docs:**
- [Vision AI](https://cloud.google.com/vision/docs)
- [Video Intelligence](https://cloud.google.com/video-intelligence/docs)
- [Natural Language](https://cloud.google.com/natural-language/docs)
- [Speech-to-Text](https://cloud.google.com/speech-to-text/docs)
- [Text-to-Speech](https://cloud.google.com/text-to-speech/docs)
- [Translation](https://cloud.google.com/translate/docs)
- [Document AI](https://cloud.google.com/document-ai/docs)

---

### Gemini Enterprise Features

#### Gemini Models in Vertex AI

| Model | Strengths | Use case |
|-------|----------|----------|
| **Gemini 1.5 Pro** | Long context (up to 2M tokens), multimodal | Document analysis, code generation, complex reasoning |
| **Gemini 1.5 Flash** | Fast, cost-effective | High-volume, lower-complexity tasks |
| **Gemini Ultra** | Highest capability | Most demanding reasoning, creative, and multimodal tasks |

#### Vertex AI Agent Builder

Build **AI agents** and **search applications** powered by generative AI.

**Key components:**

| Component | Description |
|-----------|-------------|
| **Search** | Enterprise search over your data (structured + unstructured) |
| **Conversation** | Chatbots and conversational agents grounded in your data |
| **Grounding** | Connect Gemini to your data sources (GCS, BigQuery, websites) for factual responses |
| **Extensions** | Connect agents to external APIs and tools |
| **Data connectors** | Index data from Cloud Storage, BigQuery, websites, third-party sources |

**Architecture pattern: Retrieval Augmented Generation (RAG)**
```
User query -> Agent Builder retrieves relevant documents from your data store ->
Gemini generates response grounded in those documents -> Response with citations
```

#### Gemini in Google Cloud Products

| Product | Gemini integration | Use case |
|---------|-------------------|----------|
| **Gemini Code Assist** | AI code completion and generation in IDEs | Developer productivity |
| **Gemini in BigQuery** | SQL generation, data insights, Python notebooks | Data analysis |
| **Gemini in Looker** | Natural language to dashboard queries | Business intelligence |
| **Gemini Cloud Assist** | Infrastructure recommendations, troubleshooting | Cloud operations |
| **NotebookLM** | AI research assistant grounded in your documents | Research, document analysis |

**Exam tips:**
- **Vertex AI Agent Builder** = the answer when the exam says "build a chatbot/search app grounded in company data"
- **Grounding** prevents hallucination by connecting Gemini to your actual data sources
- **RAG (Retrieval Augmented Generation)** is the architectural pattern -- retrieve context, then generate
- Know the difference: **Gemini Code Assist** = IDE code generation. **Gemini Cloud Assist** = cloud operations help. **Gemini in BigQuery** = SQL/data help.
- Agent Builder **Extensions** connect agents to external APIs (e.g., call a REST API, query a database)

**Docs:**
- [Vertex AI Gemini API](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/gemini)
- [Vertex AI Agent Builder](https://cloud.google.com/generative-ai-app-builder/docs/introduction)
- [Grounding](https://cloud.google.com/vertex-ai/docs/generative-ai/grounding/overview)

---

### Integrating AI Models from Model Garden

#### Model Garden Overview

Vertex AI Model Garden is a **curated catalog** of models available for deployment on Google Cloud.

**Model categories:**

| Category | Examples | Deployment options |
|----------|---------|-------------------|
| **Google models** | Gemini, PaLM 2, Imagen, Codey | API (managed), Vertex AI endpoints |
| **Open-source models** | Llama 3, Mistral, Falcon, Stable Diffusion | One-click deploy to Vertex AI endpoint, GKE |
| **Partner models** | Anthropic Claude, AI21, Cohere | API through Model Garden |

#### Deployment Options

| Option | Description | When to use |
|--------|-------------|-------------|
| **Model-as-a-Service (MaaS)** | Call the model via API; no infrastructure management | Fastest path; pay per token/request |
| **One-click deploy** | Deploy to a Vertex AI endpoint (dedicated VM with GPU) | Need dedicated capacity; predictable performance |
| **Self-deploy on GKE** | Deploy on your GKE cluster with GPUs/TPUs | Maximum control; existing GKE infrastructure; custom serving stack |
| **Download weights** | Download model weights for local/on-prem use | Air-gapped environments; custom serving infrastructure |

#### Fine-tuning Options

| Method | Description | Data needed | Use case |
|--------|-------------|-------------|----------|
| **Supervised fine-tuning** | Train on labeled examples (input-output pairs) | Hundreds-thousands of examples | Adapt model to specific task format |
| **RLHF (Reinforcement Learning from Human Feedback)** | Human preferences guide model behavior | Human preference data | Align model outputs with human values |
| **Adapter tuning (LoRA)** | Train small adapter layers, keep base model frozen | Fewer examples needed | Efficient fine-tuning; multiple adapters per model |
| **Prompt tuning** | Optimize soft prompts prepended to input | Minimal data | Lightest fine-tuning; task specialization |
| **Distillation** | Train smaller model from larger model outputs | Generated from teacher model | Reduce serving cost while maintaining quality |

```bash
# Deploy an open-source model from Model Garden (via console or SDK)
# Example: Deploy a model to a Vertex AI endpoint
gcloud ai endpoints create \
  --display-name=llama-endpoint \
  --region=us-central1

# Fine-tune a Gemini model (via SDK)
# gcloud does not fully support all fine-tuning params; use Python SDK
from vertexai.preview.tuning import sft
sft_tuning_job = sft.train(
    source_model="gemini-1.5-pro-002",
    train_dataset="gs://my-bucket/train.jsonl",
    validation_dataset="gs://my-bucket/val.jsonl",
    epochs=3,
    learning_rate_multiplier=1.0,
)
```

**Exam tips:**
- **Model Garden** = single place to discover, test, and deploy models (Google + open-source + partner)
- **MaaS** = simplest deployment, no infrastructure. Choose this when you don't need customization.
- **Fine-tuning** options range from lightweight (prompt tuning) to heavy (supervised fine-tuning + RLHF)
- **LoRA/adapter tuning** is the sweet spot for most fine-tuning -- efficient, doesn't modify base weights, can have multiple adapters
- If the exam says "deploy an open-source LLM on Google Cloud with minimal effort" -> **Model Garden one-click deploy**
- If the exam says "customize a foundation model for domain-specific tasks" -> **fine-tuning** (supervised or adapter)
- **Distillation** = make a smaller, cheaper model that mimics a larger one -- key cost optimization strategy for serving

**Docs:**
- [Model Garden](https://cloud.google.com/vertex-ai/docs/start/explore-models)
- [Fine-tuning models](https://cloud.google.com/vertex-ai/docs/generative-ai/models/tune-models)
- [Deploy models from Model Garden](https://cloud.google.com/vertex-ai/docs/start/deploy-model)

---

## Section 2 Summary: Key Architect Decisions

| Decision | Key factors | Primary options |
|----------|-------------|-----------------|
| **Hybrid connectivity** | Bandwidth, latency, encryption, cost | HA VPN vs Dedicated vs Partner Interconnect |
| **VPC architecture** | Org structure, routing needs, scale | Shared VPC vs VPC Peering vs NCC Hub-and-spoke |
| **Load balancer** | L4 vs L7, internal vs external, proxy vs passthrough | Application LB, Network LB (proxy/passthrough) |
| **Storage class** | Access frequency, cost, retrieval needs | Standard / Nearline / Coldline / Archive / Autoclass |
| **Database scaling** | Read vs write scaling, relational vs NoSQL | Spanner (write scale) vs Cloud SQL (read replicas) vs AlloyDB |
| **Compute platform** | Control, scaling, operational overhead | Compute Engine vs GKE vs Cloud Run vs Functions |
| **GKE mode** | Control vs simplicity | Standard (full control) vs Autopilot (managed) |
| **ML infrastructure** | Scale, framework, cost | GPUs (flexible) vs TPUs (scale, TF/JAX) |
| **AI approach** | Expertise, customization, speed | Pre-trained APIs vs AutoML vs Custom training |
| **Model serving** | Latency, traffic pattern, cost | Dedicated endpoint vs Serverless vs Batch |
| **Foundation model use** | Customization, cost, data sensitivity | MaaS vs Fine-tuning vs Self-hosted |

---

## Cross-Reference: Related Sections

| Topic | Also covered in |
|-------|----------------|
| IAM and access control | Section 3 (Security and Compliance) |
| Terraform for infrastructure | Section 5 (Managing Implementations) |
| Cost optimization | Section 4 (Optimizing Processes) |
| Monitoring and observability | Section 6 (Operations Excellence) |
| DR and HA architecture | Section 1 (Designing and Planning) |
| VPC Service Controls | Section 3 (Security and Compliance) |
| Well-Architected Framework | Section 6 (Operations Excellence) |

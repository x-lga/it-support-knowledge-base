# NET-002 - Asymmetric Routing: Diagnosis and Impact

**Article ID:** NET-002
**Category:** Networking - Routing and WAN
**Severity:** P2 (intermittent connectivity failures)
**Cert alignment:** CompTIA Network+
**Last verified:** 2026-07

---

## What Asymmetric Routing Is and Why It Causes Intermittent Failures

Asymmetric routing occurs when the outbound path from A to B and the return
path from B to A use different routes. This is invisible in many environments
and completely normal in others - but it breaks stateful firewalls and can
cause bizarre intermittent connectivity that is extremely difficult to diagnose
without understanding the routing.

**Why stateful firewalls break with asymmetric routing:**
A stateful firewall tracks connections: "Source A opened a connection to
Destination B via this interface." The firewall expects the return traffic
to come in on the same interface. If the return traffic arrives on a different
interface (because it took a different path), the firewall does not recognise
it as part of an established connection and drops it. The result is TCP sessions
that establish partially but then drop, or HTTP requests that time out after the
initial SYN-ACK.

---

## Step 1 - Identify Asymmetric Routing with Traceroute

```powershell
# Forward path: from the client to the destination
tracert -d 192.168.5.10   # Replace with the destination IP
# Note the hops, especially the first 3

# Reverse path: requires running traceroute FROM the destination back
# SSH or RDP to the destination machine and run:
tracert -d [client IP]
# Compare the hops — if they are different routes, routing is asymmetric
```

**Reading the output:**
If forward path goes: Client → Router A → ISP1 → Destination
And return path goes: Destination → Router B → ISP2 → Client
Then any stateful firewall in Router A or Router B's path will see
only half of the connection.

---

## Step 2 - Identify Asymmetric Routing in an Azure Hybrid Environment

In Azure with forced tunnelling (user-defined routes sending all traffic to
an NVA or Azure Firewall), asymmetric routing is a common misconfiguration:

```bash
# Check effective routes for a VM's NIC
az network nic show-effective-route-table \
    --resource-group rg-prod \
    --name nic-app-vm-01 \
    --output table

# Look for:
#   0.0.0.0/0 → VirtualAppliance (Azure Firewall IP) — expected if forced tunnelling
#   But also check: does the return path from the Firewall go back through the same path?
#   If the Firewall has a route sending return traffic directly (bypassing the NVA):
#   asymmetric routing is occurring
```

---

## Resolving Asymmetric Routing

The resolution is always the same: ensure outbound and return traffic use the
same path through the stateful device. This means either:
1. Routing both flows through the same firewall (preferred)
2. Disabling stateful inspection on the affected interface
3. Adding static routes to force symmetric traffic

In Azure: ensure UDRs are applied to ALL subnets that communicate, not just
some of them. A missing UDR on the return-path subnet is the most common cause.


---


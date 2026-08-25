# Phase 04 — Hybrid Connectivity Troubleshooting Case Study

## Why this document exists

This is the troubleshooting record for the hardest networking checkpoint in Phase 04.

The goal is not to preserve every command. The goal is to preserve the **reasoning model** that turned a vague AWS Directory Service failure into a specific, testable diagnosis.

## The symptom

AWS AD Connector repeatedly failed with:

```text
DNS unavailable (TCP port 53) for IP 192.168.14.10
```

At first glance this could mean many things:

- DNS service down,
- Windows firewall,
- bad VPC route,
- wrong subnet association,
- EC2 routing appliance misconfiguration,
- WireGuard routing/AllowedIPs,
- Linux forwarding,
- Security Group/NACL,
- return-path failure.

Changing all of these at once would make the result impossible to explain.

## The diagnostic model

Think of the packet like a parcel moving through checkpoints:

```text
AD Connector ENI
      ↓
Private subnet
      ↓
Route table
      ↓
EC2 Security Group
      ↓
WG-HUB ens5
      ↓
Linux forwarding / NAT
      ↓
WG-HUB wg0
      ↓
WireGuard tunnel
      ↓
MADAR-WG01
      ↓
Local LAN
      ↓
MADAR-DC01:53
      ↓
Return path
```

The question at each step is simple:

> Did the packet reach this checkpoint?

## Step 1 — Prove the tunnel itself

WireGuard showed:

- a recent handshake,
- bidirectional transfer counters,
- successful tunnel-side ping.

This proved peer connectivity, but **not** end-to-end routing to Active Directory.

## Step 2 — Prove the local destination

`MADAR-WG01` could reach `MADAR-DC01` and query `madar.local` DNS. TCP checks to core AD services succeeded.

This proved the local Linux router and domain controller could communicate.

## Step 3 — Prove AWS route-table intent

Both private subnets used by Directory Service were verified against the intended private route table.

The route was active:

```text
192.168.14.0/24 -> MADAR-P04-WG-HUB
```

This removed "wrong private subnet route table" from the suspect list.

## Step 4 — Use packet capture as a boundary detector

`tcpdump` was run first on `wg0` and then on `ens5` of the EC2 appliance.

During a failing Connector attempt, neither interface initially saw the expected DNS traffic.

That observation was important: there was no reason to keep modifying WireGuard if the packet had not even reached the Linux instance.

## Step 5 — Inspect the AWS security boundary

The AWS-created Directory Service Security Group allowed outbound traffic.

The `WG-HUB` Security Group, however, initially allowed only:

```text
UDP 51820 from 0.0.0.0/0
```

That was sufficient for WireGuard peer establishment, but the instance was also being used as a **transit router** for VPC traffic.

The required VPC-side transit allowance was added for the lab network `10.50.0.0/16`.

This is the key lesson:

> A working VPN handshake does not prove that routed workload traffic is allowed to enter the network appliance.

## Step 6 — Verify Linux forwarding / NAT

The EC2 appliance had explicit forwarding rules for AWS-to-on-prem traffic and the return path. The rules were persisted with `netfilter-persistent`.

The local gateway also used the required routing/NAT behavior for the lab topology.

## Step 7 — Re-test and observe real DNS traffic

After the AWS-side transit path was corrected, packet capture showed AWS-originated TCP/53 traffic reach `192.168.14.10` and receive a reply.

A complete TCP handshake was observed.

That was the decisive network proof.

## Step 8 — Read the new failure correctly

The next AD Connector attempt changed from:

```text
DNS unavailable
```

to:

```text
Invalid credentials (bad username/password)
```

This is not "another networking failure." It is evidence that troubleshooting successfully moved the integration to the next layer.

The remaining action is credential validation for the dedicated `svc-adconnector` account.

## What not to do

During this incident, the preferred approach became:

```text
Observe
  ↓
Form one hypothesis
  ↓
Test one boundary
  ↓
Change one thing
  ↓
Re-test
```

Avoid:

```text
Change route + firewall + WireGuard + password + connector
                         ↓
                 "it works now"
                         ↓
                 no idea why
```

A portfolio project is stronger when the engineer can explain **why** the system failed and **which evidence** proved the fix.

## Final lesson

The hybrid path is a chain. A green status at one layer does not guarantee the next layer works.

```text
WireGuard handshake           ≠ routed application connectivity
Ping                          ≠ DNS/Kerberos/LDAP readiness
Route table entry             ≠ packet reached EC2
EC2 received packet           ≠ packet entered WireGuard
DNS connectivity              ≠ valid AD credentials
AD Connector Active           ≠ least-privilege AWS access
```

Each layer requires its own proof.
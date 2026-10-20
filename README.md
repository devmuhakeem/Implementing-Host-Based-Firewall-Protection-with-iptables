# Implementing Host-Based Firewall Protection with iptables

A hands-on lab using iptables, the standard Linux firewall utility, to block internet access for a single user on a shared Ubuntu machine without affecting any other account on the same system, then reversing the change and confirming access was restored.

## Scenario
A shared Linux machine needed internet access restricted for one specific user (`bob`) without affecting anyone else on that system. This required a rule scoped not by IP or port, but by the Linux user ID that owns the outgoing traffic.

## What I did

### 1. Confirmed the starting state
- Logged in as `bob` and loaded a website in Firefox — confirmed working internet access before any rule existed

### 2. Identified the target user's UID
```bash
id bob
```
- Returned Bob's UID (1000), the identifier the firewall rule would actually match against

### 3. Checked the existing rule set
```bash
iptables -L
```
- Confirmed no rules existed yet — consistent with Bob's unrestricted access

### 4. Added a user-scoped DROP rule
```bash
iptables -A OUTPUT -o eth0 -m owner --uid-owner 1000 -j DROP
```
- `-A OUTPUT`: appends to the OUTPUT chain (traffic originating from this machine)
- `-o eth0`: scoped to the `eth0` interface
- `-m owner --uid-owner 1000`: the key piece — matches traffic by the UID of the local process owner, not by IP/port
- `-j DROP`: silently discards matching traffic (no response sent back, unlike REJECT)

### 5. Verified the block
- Reloaded the website as Bob — it failed to load, confirming the rule was in effect

### 6. Removed the rule and confirmed restoration
```bash
iptables -t filter --delete OUTPUT 1
iptables -L
```
- Confirmed the rule set was empty again
- Reloaded the website as Bob — access was fully restored

## Key takeaways
- The `owner` match module lets a host-based firewall filter by *which local user* generated the traffic, not just by address or port — something IP-based filtering alone can't do on a machine shared by multiple users
- DROP vs. REJECT is a deliberate choice: DROP discards silently (looks like a timeout), REJECT sends an explicit response — DROP is often preferred when you don't want to confirm a block is intentional
- `iptables --delete <chain> <number>` is positional — in a rule set with more than one entry, list rules with `--line-numbers` first to avoid deleting the wrong one
- Testing restoration (not just the restriction) is what confirms a change is fully reversible with no side effects left behind
- This is a host-enforced control: anyone with root on the same machine can remove the rule — useful for scoped, temporary restriction, not a substitute for real network segmentation

## Tools
iptables, Ubuntu (Linux), Bash

---
*Completed in an isolated training lab environment on a provisioned Linux machine.*

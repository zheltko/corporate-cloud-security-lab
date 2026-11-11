# Network Scanning: Nmap Analysis
# Network Scanning & Firewall Behavior Analysis

## Objective
To verify the effectiveness of the host-based firewall (UFW) on the secondary VM (`kate-labb2`, IP `10.20.0.5`) and understand how it handles different types of traffic (ICMP, open ports, closed ports).

## Methodology & Tools
Scanning was performed from the primary VM (`kate-labb`, IP `10.20.0.4`) using `nmap` and `netcat` (nc).

## Results & Analysis

### 1. Port Scanning (Nmap)
**Command:** `sudo nmap -sT -Pn -p- 10.20.0.5`
**Result:**
*   **Host is up** (detected despite ICMP block due to `-Pn` flag).
*   **22/tcp (SSH):** `open`
*   **All other 65534 ports:** `filtered`

**Analysis:** The `-Pn` flag skips host discovery (ping), forcing nmap to scan ports directly. The `filtered` state for all non-SSH ports proves that UFW is configured with a default `DROP` policy. Instead of replying with a "port closed" message (RST), the firewall silently discards the packets, providing no information to the attacker.

### 2. ICMP Ping Test
**Command:** `ping -c 3 10.20.0.5`
**Result:** `100% packet loss`
**Analysis:** UFW drops incoming ICMP echo requests. This prevents simple network discovery and makes the host less visible to automated scanners.

### 3. Closed Port Connection Test
**Command:** `nc -zv -w 3 10.20.0.5 80`
**Result:** `Connection timed out`
**Analysis:** Port 80 is not open in UFW. The connection attempt times out rather than being explicitly refused. This is the expected behavior for `default deny incoming` (DROP policy), as opposed to `REJECT` which would yield "Connection refused".

### 4. Open Port Connection Test
**Command:** `nc -zv -w 3 10.20.0.5 22`
**Result:** `succeeded!`
**Analysis:** Port 22 is explicitly allowed by UFW rules. The firewall permits the connection, confirming that the intended access control works correctly.

## Key Takeaways
1. **Defense in Depth:** Relying on a cloud Security Group is not enough; a host-based firewall (UFW) provides a crucial second layer of defense.
2. **DROP vs REJECT:** Using `DROP` (default in UFW deny) hides the existence of closed ports, whereas `REJECT` reveals them. `DROP` is preferred for security.
3. **Nmap Adaptability:** Attackers will use flags like `-Pn` to bypass ICMP blocks. Security measures must not rely solely on hiding from ping.
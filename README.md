# Network Traffic Analysis using Wireshark, tcpdump and Snort

## Objective
Capture normal and suspicious network traffic, analyze it with Wireshark and tcpdump, and detect a port scan using a custom Snort rule.

## Environment
- OS: Kali Linux (VirtualBox)
- Tools: tcpdump, Wireshark, Nmap, Snort 3 (3.12.2.0)
- Scan target: scanme.nmap.org (public server provided by Nmap for practice scanning)

## Methodology
1. Verified the network interface (eth0) and installed the tools.
2. Captured baseline traffic (ping, curl, nslookup) with tcpdump: pcaps/baseline.pcap (95 packets).
3. Captured a TCP SYN scan (nmap -sS -p 1-100) with tcpdump: pcaps/scan.pcap (409 packets).
4. Analyzed both files in Wireshark using protocol hierarchy, conversations and display filters.
5. Analyzed both files from the command line with tcpdump filters.
6. Wrote custom Snort rules and replayed both captures through Snort.

## Findings

### Baseline traffic
- Low-volume, mixed traffic: DNS, ICMP, HTTP/TCP.
- No repeated connection attempts to multiple ports.

### Port scan traffic
- One source IP (10.0.2.15) sent 199 SYN packets to 101 different ports on 45.33.32.156 (scanme.nmap.org) within a few seconds.
- Open ports replied with SYN-ACK; closed ports replied with RST.
- Wireshark filter used: tcp.flags.syn==1 && tcp.flags.ack==0

### Snort detection
- Rule: alert tcp any any -> any any (msg:"Possible SYN Port Scan"; flags:S; detection_filter:track by_src, count 20, seconds 5; sid:1000002; rev:1;)
- Scan capture: 179 "Possible SYN Port Scan" alerts.
- Baseline capture: 0 scan alerts, only ICMP ping alerts, so the rule does not fire on normal traffic.

## Indicators of a port scan
- Many SYN packets from a single source IP.
- Many different destination ports in a short time.
- Very few completed connections, many RST replies.

## Recommendations
- Rate-limit and block repeated SYN packets at the firewall.
- Deploy an IDS/IPS such as Snort with scan-detection rules.
- Monitor logs for one source touching many ports.
- Segment the network and close unused ports.

## Project structure
- pcaps/ - baseline.pcap, scan.pcap
- snort-rules/ - local.rules, snort-lab.conf, scan-alerts.txt, baseline-alerts.txt
- screenshots/ - proof of each step

## Ethical note
Scanning was done only against scanme.nmap.org, which permits it, with a small port range (1-100).

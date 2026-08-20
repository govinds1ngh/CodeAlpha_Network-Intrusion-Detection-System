# Simple Intrusion Detection System (IDS)

A lightweight, rule-based Python IDS built on [Scapy](https://scapy.net/). It captures live IP traffic and raises alerts for:

- **Port scans**: a source IP sends TCP SYN packets to more than `PORT_SCAN_THRESHOLD` distinct destination ports during one five-second capture interval.
- **ICMP floods**: a source IP sends more than `ICMP_FLOOD_THRESHOLD` ICMP packets during one five-second capture interval.
- **Suspicious payloads**: TCP or UDP packets contain configured keywords such as `union select`, `<script>`, or `etc/passwd`.

This is an educational / demonstration tool, not a production-grade IDS. It's meant for learning how signature- and threshold-based detection works, testing on your own lab traffic, or as a starting point for something more robust (e.g. Suricata, Zeek, Snort).

## Requirements

- Python 3.8 or newer
- [Scapy](https://scapy.net/): `pip install scapy`
- **Administrator/root privileges** because raw packet capture requires elevated permissions
- **Linux/macOS**: `libpcap` (usually preinstalled or available through the system package manager)
- **Windows**: [Npcap](https://npcap.com/) must be installed separately

Install Scapy with:

```bash
python -m pip install scapy
```

## Usage

```bash
# Linux/macOS: run on all available interfaces for 30 seconds
sudo python3 network.py

# Linux/macOS: run on a specific interface for 60 seconds
sudo python3 network.py --interface eth0 --duration 60

# Windows PowerShell: run the terminal as Administrator first
python network.py --duration 60
```

| Flag | Description | Default |
|---|---|---|
| `-i`, `--interface` | Network interface to monitor (for example, `eth0`, `en0`, or `Wi-Fi`) | all interfaces |
| `-d`, `--duration` | How long to run, in seconds | `30` |

Press `Ctrl+C` to stop early. A summary of the last 5 alerts is printed on exit.

## Configuration

The thresholds live near the top of `network.py` and can be tuned:

```python
PORT_SCAN_THRESHOLD = 15   # distinct destination ports before flagging a scan
ICMP_FLOOD_THRESHOLD = 20  # ICMP packets before flagging a flood
TIME_WINDOW = 5            # capture interval, in seconds
```

When the program stops, it prints the total number of alerts and the last five alert messages.

## Known limitations

- **Fixed capture intervals.** State is cleared every `TIME_WINDOW` seconds rather than using a true sliding window, so an attack that straddles an interval boundary may be split into two sub-threshold bursts and missed. A timestamp-based approach would be more robust.
- **UDP port scans are not detected** — only TCP SYN-based scanning is tracked. UDP scanning (e.g. `nmap -sU`) will not trigger an alert.
- **No capture filter (BPF)** is applied, so all traffic on the interface is processed, which can be costly on busy networks.
- **`alert_log` grows unbounded** for the life of the process; consider capping or rotating it for long-running deployments.
- **Payload inspection is limited.** Encrypted traffic, fragmented payloads, and payloads split across packets may not match. Simple substring matching can also produce false positives.

## Disclaimer

This tool captures and inspects network traffic. Only run it on networks and systems you own or are explicitly authorized to monitor. Unauthorized packet sniffing may violate local laws and organizational policies.

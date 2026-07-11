# Python Port Scanner

A multi-threaded Python command-line tool that scans a target host for open
TCP ports and identifies the service running on each one.

## Why I built this

This is the natural follow-up to my [failed-login-detector](https://github.com/Cwubs28/failed-login-detector)
project. That tool analyzes logs after an attack has already happened.
This one answers the question, "What's actually exposed on a system in the first place?"
Port scanning is the first step of both real penetration tests and real
reconnaissance by attackers, so understanding how it works from the ground
up (instead of just running Nmap) was the goal here.

## Legal note

Only scan hosts you own or have explicit written permission to scan.
Scanning systems you don't control without permission is illegal in most
places (in the US, this can fall under the Computer Fraud and Abuse Act).
This script defaults to safe local testing against `127.0.0.1` (your own
machine).

## What it does

- Scans a target IP/hostname across a custom port range or list
- Uses multi-threading so large port ranges scan in seconds instead of minutes
- Identifies common services by port number (SSH, HTTP, FTP, RDP, etc.)
- Grabs service banners when available for extra detail (e.g. identifying
  the exact software/version running on an open port)
- Outputs a clean report to screen or to a file

## How to run it

```bash
# Scan the default range (1-1024) on your own machine
python3 port_scanner.py 127.0.0.1

# Scan a specific range
python3 port_scanner.py 127.0.0.1 --ports 1-1000

# Scan specific ports
python3 port_scanner.py 127.0.0.1 --ports 22,80,443,3389

# Mix ranges and specific ports
python3 port_scanner.py 127.0.0.1 --ports 1-100,443,8080

# Control thread count (higher = faster, but more aggressive)
python3 port_scanner.py 127.0.0.1 --ports 1-1000 --threads 200

# Save the report to a file
python3 port_scanner.py 127.0.0.1 --ports 1-1000 --output report.txt
```

## Example output

Ran against localhost, scanning ports 1-1024:

```
OPEN PORTS FOUND: 2
[OPEN] Port 135   unknown
[OPEN] Port 445   SMB
```

See screenshot attached for the full scan output.

## How it works

1. **Connection attempt** — for each port, the script tries to open a real
   TCP connection using Python's `socket` library. A successful connection
   means something is listening on that port.
2. **Threading** — scanning happens across multiple threads pulling from a
   shared queue, so hundreds of ports can be checked in parallel instead of
   one at a time (a 1000-port scan done sequentially could take ~17 minutes
   with a 1-second timeout per port; threaded, it takes seconds).
3. **Banner grabbing** — once a port is confirmed open, the script tries to
   read whatever the service sends back first, which often reveals the
   exact software and version running there.

## Skills demonstrated

- Socket programming (TCP connections at the network level)
- Command-line tool design 
- Core security concept: reconnaissance / attack surface discovery
- Responsible use disclosure and legal awareness around scanning tools

## Possible next steps

- Add UDP scanning (this version only covers TCP)
- Export results as JSON for feeding into other tools
- Feed discovered open ports into the failed-login-detector as a combined
  recon-to-detection pipeline

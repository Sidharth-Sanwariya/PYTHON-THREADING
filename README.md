# PYTHON-THREADING

How I Built a Port Scanner That's 50x Faster Using Python Threading.

# How I Built a Port Scanner That's 50x Faster Using Python Threading

## The Problem

Port scanning is essential for security testing. But scanning 1000 ports one by one takes time.

**Single-thread scanning:**
- Scans 1 port at a time
- 100 ports = 100 seconds
- Too slow for real-world use

## The Solution: Threading

Threading allows multiple operations to run simultaneously. Instead of scanning one port, you scan 10, 50, or 100 ports at once.

**Multi-thread scanning:**
- Scans 50 ports simultaneously
- 100 ports = 2 seconds
- **50x faster**

## Single-Thread Version (Slow)

```python
import socket
import time

def scan_port(host, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(1)
    result = sock.connect_ex((host, port))
    sock.close()
    return result == 0

target = "scanme.nmap.org"
start = time.time()

for port in range(1, 101):
    if scan_port(target, port):
        print(f"Port {port}: OPEN")

print(f"Completed in {time.time() - start:.2f} seconds")

Takes 100 sec

##Multi-Thread Version (Fast)

import socket
import threading
import time
from queue import Queue

def scan_port(host, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(1)
    result = sock.connect_ex((host, port))
    sock.close()
    return result == 0

def worker(host, queue, results):
    while not queue.empty():
        try:
            port = queue.get_nowait()
            if scan_port(host, port):
                results.append(port)
        except:
            break
        finally:
            queue.task_done()

def scan_ports(host, start_port, end_port, num_threads=50):
    queue = Queue()
    results = []
    
    for port in range(start_port, end_port + 1):
        queue.put(port)
    
    threads = []
    for _ in range(num_threads):
        t = threading.Thread(target=worker, args=(host, queue, results))
        t.start()
        threads.append(t)
    
    queue.join()
    for t in threads:
        t.join()
    
    return results

target = "scanme.nmap.org"
start = time.time()
open_ports = scan_ports(target, 1, 100)
print(f"Open ports: {open_ports}")
print(f"Completed in {time.time() - start:.2f} seconds")

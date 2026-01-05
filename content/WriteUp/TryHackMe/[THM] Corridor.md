# solve

- nmap 시 80 하나가 끝
```
─$ nmap -sV -sC --open 10.49.135.241 -oA tcpdetailed
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-05 13:27 EST
Nmap scan report for 10.49.135.241
Host is up (0.16s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Werkzeug httpd 2.0.3 (Python 3.10.2)
|_http-title: Corridor
|_http-server-header: Werkzeug/2.0.3 Python/3.10.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.84 seconds
```





---
title:  "Optimum - Hack The Box"
classes: wide
---

![styled-image](/assets/images/Optimum.png){: .align-center style="width: 50%;"}

Optimum es una máquina sencilla basada en vulnerar una versión obsoleta de Windows. Para obtener la shell inicial hemos vulnerado la versión 2.3 de HTTP File Server. Desde aqui hemos ejecutado herramientas automatizadas adaptandonos al enotno del sistema operativo y buscando soluciones para poder escalar privilegios en esta versión concreta.

**Portscan**
```
sudo nmap -sC -sV -p- 10.10.10.8   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-16 11:57 CEST
Nmap scan report for 10.10.10.8
Host is up (0.054s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.59 seconds
```



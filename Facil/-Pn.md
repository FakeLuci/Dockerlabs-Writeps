Máquina -Pn

Nmap de reconocimiento:

```
# Nmap 7.94SVN scan initiated Thu May 16 08:38:43 2024 as: nmap -p- --open -sS -sCV --min-rate 5000 -vvv -n -Pn -oN Target 172.17.0.2
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000060s latency).
Scanned at 2024-05-16 08:38:43 CEST for 8s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 64 vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
8080/tcp open  http    syn-ack ttl 64 Apache Tomcat 9.0.88
|_http-title: Apache Tomcat/9.0.88
|_http-favicon: Apache Tomcat
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May 16 08:38:51 2024 -- 1 IP address (1 host up) scanned in 7.94 seconds
```


Entramos en el ftp de forma anonyma, y extraemos el archivo:

```
Hello tomcat, can you configure the tomcat server? I lost the password...
```

El usuario tomcat existe, en el botón manage app iniciamos sesion con las credenciales por defecto:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/4a9f5f46-7832-41d6-9479-dbb572932cab)

Dentro vemos que la versión de tomcat es la 9.0.88

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/b074d23e-79af-426b-9e36-9461a09129da)

Vemos que hay un sitio para subir archivos y cuando subo algo me pide .war:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/7e32f7dd-7dc5-405d-9187-39a4390b256d)

Creamos el payload con msfvenom y lo subimos

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/1dba2026-b5ab-47b5-b2d9-493113da24ac)

Ejecutamos la carpeta cmd

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/c1a20949-da15-48e1-aac5-871289c25dc7)

y entablamos la reverse shell:

![image](https://github.com/M4nuTCP/Dockerlabs-Writups/assets/96147300/27623cda-4d44-4a2b-b1eb-385920afd7e9)

la entablamos y además como root








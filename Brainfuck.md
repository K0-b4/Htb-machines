
Empezamos listando todos los puertos abiertos del equipo: 

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
25/tcp  open  smtp     Postfix smtpd
110/tcp open  pop3     Dovecot pop3d
143/tcp open  imap     Dovecot imapd
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb

Al ver los puertos abiertos buscamos más información de Dovecot y Postfix, en los cuales podemos ver que hay exploits: 

![image](https://user-images.githubusercontent.com/122020487/227682597-a34de686-79cb-46cf-ae11-3608c4b23103.png)

Antes de proceder con los intentos de explotación de estos servicios, procedemos a obtener información de la web para más información: 

![image](https://user-images.githubusercontent.com/122020487/227682588-a4b01f4f-9dcb-4789-978a-72ffe673ad7b.png)

Realizamos una petición con curl para determianar si podemos conseguir más información pero no encontramos nada: 

![image](https://user-images.githubusercontent.com/122020487/227682575-4d6f756b-35cb-4673-bbb4-c88f3ee93947.png)

Al ver que whatweb no nos da mucha información, procedemos a conectarnos al servidor mediante ssl y así poder ver tanto el usuario como el dominio al que debemos conectarnos: 

openssl s_client -connect 10.10.10.17:443

![image](https://user-images.githubusercontent.com/122020487/227682571-53b5bf53-857b-44b8-896c-a19329112b06.png)

También procedemos a realizar un pequeño reconocimiento con sslscan: 

sslscan https://10.10.10.17

![image](https://user-images.githubusercontent.com/122020487/227682550-450054bd-4937-426f-82d6-da6be898ad86.png)

Al obtener tanto el dominio como un subdominio procedemos a incorporarlos en el etc/hosts para poder ver el contenido: 

![image](https://user-images.githubusercontent.com/122020487/227682537-58368e04-17a6-45c5-be21-7e76e44ee899.png)

En este caso vemos dos usuarios, recordamos que el protocolo ssh que estaba utilizando es de una versión antigua, por lo que quizás podamos enumerar usuarios, por lo que probamos el usuario orestis viendo que si existe en el servidor: 

![image](https://user-images.githubusercontent.com/122020487/227682528-50c8cb05-b02f-454a-ab24-18fd1ad671bc.png)

Al no poder hacer mucho más con el subdominio, procedemos a analizar los plugins instalados en la web, podemos obtener los plugins de tres maneras diferentes: 

1. haciendo un curl -k al dominio y filtrando con grep los plugins lo cual nos daría el siguiente resultado: 

![image](https://user-images.githubusercontent.com/122020487/227682520-13ce52c3-b793-4843-8541-49cce63f3e9b.png)

2. Al saber el path de los plugins en wordpress podemos intentar entrar al path y ver si hay directory listing, como en este caso: 

![image](https://user-images.githubusercontent.com/122020487/227682501-099080c1-2644-42b2-a63c-4f13a2af1fab.png)

En este caso podemos obtener más información de los plugins easy-wp-smtp y wp-support-plus-responsive-ticket-system con el archivo readme.txt que hay en ambos plugins permitiendo ver la versión para buscar un posible exploit:

![image](https://user-images.githubusercontent.com/122020487/227682481-14cd85c8-8f74-4bd7-b4c3-fa18bc25ab7d.png)
![image](https://user-images.githubusercontent.com/122020487/227682496-ea1ffb46-cbe7-4404-abb5-167f78619b46.png)

3. Usando wpscan el cual podemos por ejemplo filtrar por usuario y plugins con el siguiente comando: 

 wpscan --url https://brainfuck.htb --enumerate u,p --disable-tls-checks

![image](https://user-images.githubusercontent.com/122020487/227682437-40f1c5e5-2804-4234-8da6-6b63b7ea9bf9.png)

Una vez hemos obtenido toda la información de los plugins podemos mirar a ver si hay algún exploit: 

![image](https://user-images.githubusercontent.com/122020487/227682417-c7cb9bda-d8f2-4cd1-8c88-d091fb052dea.png)

Como podemos ver, hay al menos dos exploits que coinciden con la versión que tenemos, por lo que después de mirar ambos, intentamos explotar la de privilege escalation ya que a priori es la que más privilegios conseguimos y más facil de explotar nos resulta: 

![image](https://user-images.githubusercontent.com/122020487/227682402-7f850d80-01fa-441d-af12-7419e58c4683.png)

Como vemos que tenemos que hacer una petición con una estructura html, primero nos levantamos un servidor usando el siguiente comando:

php -S 0.0.0.0:80 

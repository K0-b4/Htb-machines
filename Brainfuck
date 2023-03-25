
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



Antes de proceder con los intentos de explotación de estos servicios, procedemos a obtener información de la web para más información: 



Realizamos una petición con curl para determianar si podemos conseguir más información pero no encontramos nada: 



Al ver que whatweb no nos da mucha información, procedemos a conectarnos al servidor mediante ssl y así poder ver tanto el usuario como el dominio al que debemos conectarnos: 

openssl s_client -connect 10.10.10.17:443



También procedemos a realizar un pequeño reconocimiento con sslscan: 

sslscan https://10.10.10.17



Al obtener tanto el dominio como un subdominio procedemos a incorporarlos en el etc/hosts para poder ver el contenido: 



En este caso vemos dos usuarios, recordamos que el protocolo ssh que estaba utilizando es de una versión antigua, por lo que quizás podamos enumerar usuarios, por lo que probamos el usuario orestis viendo que si existe en el servidor: 



Al no poder hacer mucho más con el subdominio, procedemos a analizar los plugins instalados en la web, podemos obtener los plugins de tres maneras diferentes: 

	1. haciendo un curl -k al dominio y filtrando con grep los plugins lo cual nos daría el siguiente resultado: 



	2. Al saber el path de los plugins en wordpress podemos intentar entrar al path y ver si hay directory listing, como en este caso: 



En este caso podemos obtener más información de los plugins easy-wp-smtp y wp-support-plus-responsive-ticket-system con el archivo readme.txt que hay en ambos plugins permitiendo ver la versión para buscar un posible exploit:





	3. Usando wpscan el cual podemos por ejemplo filtrar por usuario y plugins con el siguiente comando: 

 wpscan --url https://brainfuck.htb --enumerate u,p --disable-tls-checks



Una vez hemos obtenido toda la información de los plugins podemos mirar a ver si hay algún exploit: 



Como podemos ver, hay al menos dos exploits que coinciden con la versión que tenemos, por lo que después de mirar ambos, intentamos explotar la de privilege escalation ya que a priori es la que más privilegios conseguimos y más facil de explotar nos resulta: 



Como vemos que tenemos que hacer una petición con una estructura html, primero nos levantamos un servidor usando el siguiente comando:

php -S 0.0.0.0:80 

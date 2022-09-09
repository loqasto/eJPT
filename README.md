# eJPT
 
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

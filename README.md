# eJPT
 
   ## Network mapping

   Para hacer ping a un rango de IP's, utilizamos fping:

      fping -g -a IPRANGE

   -a nos muestra los hosts activos

   -g indica que hacemos un ping de un rango de IP's
   
   Para evitar errores, añadimos la salida a /dev/null:
   
     fping -g -a IPRANGE 2>/dev/null
     
   ## nmap
   
   Ping scan (nos dice que hosts están activos):
   
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

 Determina qué puertos TCP y UDP están abiertos y qué demonio está escuchando en esos puertos (software y versión), enviando señales al target y analizando la respuesta.
 
 ![image](https://user-images.githubusercontent.com/111526713/189317504-3e2cc9a2-0ced-47ee-a3c4-e4e271c9ddca.png)

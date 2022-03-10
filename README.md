## Explición fichero docker-compose.yml

```
version: "3.8"
services:
  apache:       #creación del contenedor apache server 
    container_name: apache_server     #asignación del nombre del contedor 
    image: httpd:latest   #imagen que se va a utilizar 
    networks:   #hace referencia a la red que se van a utilizar en este contenedor
      nt01:     #nombre de la red creada
        ipv4_address: 100.0.0.10    #ip estática para el contenedor
    ports:
      - 80:80   #mapeo de puertos [maquina anfitrión]:[contenedor]
    volumes:
      - apache2:/usr/local/apache2    #referenciación del volumnen a utilizar, indicandole la ruta
  bind9:    #creación del contenedor DNS
    container_name: bind9_server      #asignación de un nombre
    image: internetsystemsconsortium/bind9:9.16     #la imagen a utilizar
    networks:
      nt01:     #referenciación de la red a utilizar
        ipv4_address: 100.0.0.254   #asiganación de una IP estática
    ports:
      - 53:53   #mapeo de puertos [maquina anfitrión]:[contenedor]
    volumes:
      - bind:/etc/bind      #referenciación del volumnen a utilizar, indicandole la ruta
  cliente:      #creación del contenedor cliente
    container_name: apache_cliente    #asignación de un nombre al contenedor
    image: kasmweb/desktop-deluxe:1.9.0-rolling
    networks:
      nt01:   #referenciación de la red a utilizar
        ipv4_address: 100.0.0.2   #asignación de una IP estática 
    ports:
      - 6901:6901    #mapeo de puertos [maquina anfitrión]:[contenedor]
    dns:
      - 100.0.0.254  #hace referencia al contenedor DNS (para que sea el servidor DNS del cliente)
    environment:
      - VNC_PW=abc123.    #se trata de un parámetro propio (de la propia imagen) para cambiar la contraseña de acceso por VNC
volumes:      #creación de los volumenes
  bind:     #voluemn del contenedor de DNS
    external: true    #con esta opción no se crearán los volumenes si ya existen
  apache2:    #volumen del contenedor de apache
    external: true
networks:     #creación de la red asignada anteriormnete en los contenedores
  nt01:     #nombre de la red 
    ipam:     #IP Address Management
      config:
        - subnet: 100.0.0.0/24    #subred en la que se encontrará
```

## Configuración de Apache Web Server
_Toda la configuración del servicio está alojada en el directorio /usr/local/apache2/conf/_

En primer lugar se deberá editar el fichero httpd-vhosts.conf (directorio: /extra) en el que se configurará el VirtualHost que dará servicio a "paxina1" 
la cual funcionará sin seguridad (http).
Para la configuración del VirtualHost los parámetros más importantes a configurar serán "DocumentRoot" y "ServerName" donde el primero,
será la ruta absoluta de donde se encuentra el directorio que contiene el "index.html" y el segundo será la dirección DNS que identifica la web.
```
<VirtualHost *:80>
    DocumentRoot "/usr/local/apache2/htdocs/paxina1"
    ServerName paxina1.oadri.gal
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>
```
### Configuración de seguridad "https"


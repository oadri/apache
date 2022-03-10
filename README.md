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

En este caso se dotará de acceso seguro a través del protocolo https a la página "páxina2" para esto, en primer lugar, se deberá habilitar en el fichero de configuración principal "httpd.conf" el siguiente módulo: (descomentar) 
```
LoadModule ssl_module modules/mod_ssl.so
```
y a continuación, se modificará el VirtualHost dedicado a la configuración SSL. En este archivo se aplicará la misma configuración que en el apartado anterior añadiando la ruta del de la web y la dirección DNS que apuntará a esta.
En este fichero tabién habrá que descomentar la linea "SSLEngine on".
```
<VirtualHost _default_:443>

#   General setup for the virtual host
DocumentRoot "/usr/local/apache2/htdocs/paxina2"
ServerName paxina2.oadri.gal:443
ServerAdmin aalfonsosimon@danielcastelao.org
ErrorLog /proc/self/fd/2
TransferLog /proc/self/fd/1

#   SSL Engine Switch:
#   Enable/Disable SSL for this virtual host.
SSLEngine on

#   Server Certificate:
#   Point SSLCertificateFile at a PEM encoded certificate.  If
#   the certificate is encrypted, then you will be prompted for a
#   pass phrase.  Note that a kill -HUP will prompt again.  Keep
#   in mind that if you have both an RSA and a DSA certificate you
#   can configure both in parallel (to also allow the use of DSA
#   ciphers, etc.)
#   Some ECC cipher suites (http://www.ietf.org/rfc/rfc4492.txt)
#   require an ECC certificate which can also be configured in
#   parallel.
SSLCertificateFile "/usr/local/apache2/conf/server.crt"
#SSLCertificateFile "/usr/local/apache2/conf/server-dsa.crt"
#SSLCertificateFile "/usr/local/apache2/conf/server-ecc.crt"

#   Server Private Key:
#   If the key is not combined with the certificate, use this
#   directive to point at the key file.  Keep in mind that if
#   you've both a RSA and a DSA private key you can configure
#   both in parallel (to also allow the use of DSA ciphers, etc.)
#   ECC keys, when in use, can also be configured in parallel
SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"
```
Por último se debe crear el crtificado digital con openssl, en esta caso desde la terminal del mismo contenedor. 
Para terminar, se deberán añadir en el fichero anterior las rutas de "server.key" y "server.crt" los cuales son la clave privada y certificado respectivamente. 
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout usr/local/apache2/conf/server.key -out /usr/local/apache2/conf/server.crt
```
La configuración completa del servicio de DNS y del servicio Web Apache se adjuntarán a continuación:
[DNS](https://github.com/oadri/apache-dns) y [DNS](https://github.com/oadri/apache-config)

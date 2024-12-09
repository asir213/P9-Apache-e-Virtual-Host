configuración de dos páginas web

En esta práctica se configuraron dos páginas web, fabulasmaravillosas y fabulasoscuras, asociándolas a una misma dirección IP en el servidor DNS. Aquí se explica cómo se realizó:
Archivo docker-compose.yml

Este archivo define dos servicios principales y la red común:

    Servicio web:
        Usa la imagen php:7.4-apache.
        Nombre del contenedor: apaserver.
        Expone el puerto 80.
        Define los volúmenes (carpetas compartidas) para los archivos web y la configuración de Apache.
        Configura la red apared con la dirección IP 172.39.4.2.

web:
    image: php:7.4-apache
    container_name: apaserver
    ports:
      - "80:80"
    volumes:
      - ./www:/var/www
      - ./confApache:/etc/apache2
    networks:
      apared:
        ipv4_address: 172.39.4.2

Servicio DNS:

    Usa la imagen ubuntu/bind9.
    Nombre del contenedor: dnsserver.
    Expone el puerto 51 para consultas DNS.
    Define volúmenes para la configuración del servidor DNS.
    Configura la red apared con la dirección IP 172.39.4.3.

dns:
    container_name: dnsserver
    image: ubuntu/bind9
    ports:
      - "51:53"
    volumes:
      - ./confDNS/conf:/etc/bind
      - ./confDNS/zonas:/var/lib/bind
    networks:
      apared:
        ipv4_address: 172.39.4.3

Red:

    Red llamada apared en modo bridge con una subred definida.

    networks:
      apared:
        driver: bridge
        ipam:
          driver: default
          config:
            - subnet: 172.39.0.0/16

Carpeta confApache

Contiene la configuración de Apache para las páginas web. Incluye dos archivos principales:

    fabulasmaravillosas.conf: Configura el dominio fabulasmaravillosas.asircastelao.int y su ruta al archivo index.html.

<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasmaravillosas.asircastelao.int
    ServerAlias www.fabulasmaravillosas.asircastelao.int
    DocumentRoot /var/www/fabulasmaravillosas
</VirtualHost>

fabulasoscuras.conf: Similar al anterior, pero adaptado al dominio fabulasoscuras.asircastelao.int.

    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName fabulasoscuras.asircastelao.int
        ServerAlias www.fabulasoscuras.asircastelao.int
        DocumentRoot /var/www/fabulasoscuras
    </VirtualHost>

Carpeta confDNS

Contiene configuraciones del servidor DNS:

    Archivo de zona (db.asircastelao.int):
        Define el servidor de nombres principal, las direcciones IP asociadas y las páginas web.

$TTL    604800  
@       IN      SOA     ns.asircastelao.int. contacto.email.com. (
                           2         ; Serial
                      604800         ; Refresh
                       86400         ; Retry
                     2419200         ; Expire
                      604800 )       ; Negative 
@       IN      NS      ns.asircastelao.int.
ns      IN      A       172.39.4.3
fabulasoscuras       IN      A       172.39.4.2
fabulasmaravillosas     IN      A       172.39.4.2

Configuración local (named.conf.local): Indica la ubicación del archivo de zona.

zone "asircastelao.int" {
    type master;
    file "/var/lib/bind/db.asircastelao.int";
    allow-query { any; };
};

Opciones DNS (named.conf.options): Configura los forwarders y permite consultas desde cualquier IP.

    options {
        directory "/var/cache/bind";
        recursion yes;
        allow-query { any; };
        dnssec-validation no;
        forwarders {
            8.8.8.8;
            1.1.1.1;
        };
        listen-on { any; };
    };

Carpeta www

Incluye dos subcarpetas para cada página web (fabulasmaravillosas y fabulasoscuras), con sus respectivos archivos index.html.
Configuraciones adicionales

    Modificar el archivo /etc/systemd/resolved.conf para incluir la IP y puerto del servidor DNS:

DNS=172.39.4.3#51

Reiniciar el servicio:

    sudo systemctl restart systemd-resolved

    Desactivar el DNS automático en la configuración de red de la máquina y reiniciarla.

Comprobación

Ejecutar docker-compose up y visitar en el navegador las páginas configuradas, como:

    www.fabulasmaravillosas.asircastelao.int
    www.fabulasoscuras.asircastelao.int

Si hay errores, detener Apache en la máquina local:

sudo systemctl stop apache2>
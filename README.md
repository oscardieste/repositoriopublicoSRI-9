# repositoriopublicoSRI-9
Nesta práctica configurei dúas páxinas web, chamadas fabulasbrillantes e fabulasnegras, asociadas a un mesmo servidor DNS. Ambas as páxinas comparten a mesma dirección IP, configurada previamente no ficheiro docker-compose.yml. A continuación, detallo todos os pasos realizados:
## Arquivo docker-compose.yml
Neste arquivo, configúranse dous servizos: o servidor web e o servidor DNS. Ademais, defínese unha rede personalizada chamada redweb.
### web
```
web:
    image: php:7.4-apache
    container_name: servidorweb
    ports:
      - "80:80"
    volumes:
      - ./www:/var/www
      - ./confApache:/etc/apache2
    networks:
      redweb:
        ipv4_address: 192.168.100.10
```

### dns 
``'
dns:
    container_name: servidorDNS
    image: ubuntu/bind9
    ports:
      - "53:53"
    volumes:
      - ./confDNS/conf:/etc/bind
      - ./confDNS/zonas:/var/lib/bind
    networks:
      redweb:
        ipv4_address: 192.168.100.20
``'
### red 
```
networks:
  redweb:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 192.168.100.0/24
```
## Conf Apache
Para configurar Apache, creei dous ficheiros .conf , un para cada páxina web. Estes ficheiros sitúanse nas carpetas sites-available e sites-enabled.

### fabulasbrillantes.conf
```
<VirtualHost *:80>
    ServerAdmin admin@exemplo.com
    ServerName fabulasbrillantes.dominio.int
    ServerAlias www.fabulasbrillantes.dominio.int
    DocumentRoot /var/www/fabulasbrillantes
</VirtualHost>
```
### fabulasnegras.conf
```
<VirtualHost *:80>
    ServerAdmin admin@exemplo.com
    ServerName fabulasnegras.dominio.int
    ServerAlias www.fabulasnegras.dominio.int
    DocumentRoot /var/www/fabulasnegras
</VirtualHost>
```
## Configuración do Servidor DNS
Os ficheiros necesarios para configurar Bind9 están organizados nas carpetas conf e zonas.
### Ficheiro da zona (db.dominio.int)
```
$TTL    604800
@       IN      SOA     ns.dominio.int. contacto.dominio.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
@       IN      NS      ns.dominio.int.
ns      IN      A       192.168.100.20
fabulasbrillantes  IN      A       192.168.100.10
fabulasnegras      IN      A       192.168.100.10
```
### named.conf.local
```
zone "dominio.int" {
    type master;
    file "/var/lib/bind/db.dominio.int";
    allow-query { any; };
};
```
### named.conf.options
```
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    dnssec-validation no;
    forwarders {
        8.8.8.8;  # Google DNS
        1.1.1.1;  # Cloudflare DNS
    };
    listen-on { any; };
};
```
## www
Editei o ficheiro /etc/systemd/resolved.conf para engadir o DNS configurado:
```
DNS=192.168.100.20
```
Desactivei o DNS automático na configuración de rede.
Reiniciei o servizo de resolución:
```
sudo systemctl restart systemd-resolved
```
### Comprobación 
Iniciei os contedores co comando:
```
docker compose up
```

No navegador, probei os dominios configurados:
        www.fabulasbrillantes.dominio.int
        www.fabulasnegras.dominio.int
Ambas as páxinas cargáronse correctamente co seu contido HTML.

Se aparece algún erro, usaríamos o servizo Apache no host coa seguinte orde:
```
sudo systemctl stop apache2
```
#FIN

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

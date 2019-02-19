# apache

## Configuracion HTTPS

Crear claves RSA de 2048 bit

```
sudo openssl genrsa -out seguro.key 2048
```
Generar una solicitud de certificado CSR, rellenando los datos
```
sudo openssl req -new -key seguro.key -out seguro.csr
```
Esta solicitud de certificado creada, la podría enviar a una autoridad de certificación para generase el
certificado crt. En este caso lo firmamos nosotros, es decir creamos un certificado auto firmado usando
la clave privada.
```
sudo openssl x509 -req -days 365 -in seguro.csr -signkey seguro.key -out seguro.crt
```
Lo siguiente que haremos es mover la clave y el certificado creados a los directorios que utiliza por
defecto apache y configurar los permisos adecuados (se habrán creado en el directorio en el que
estuviéramos al ejecutar los comandos anteriores).
```
sudo mv seguro.key /etc/ssl/private/
sudo mv seguro.crt /etc/ssl/certs/
sudo chown root:ssl-cert /etc/ssl/private/seguro.key
sudo chmod 640 /etc/ssl/private/seguro.key
sudo chown root:root /etc/ssl/certs/seguro.crt 
```
Creamos el PEM
```
cat seguro.key seguro.crt > nombre-sitio.pem
cp nombre-sitio.pem /etc/ssl/private/
```
Modificaremos el site, comprobando que dentro de virtualhost tenemos las siguientes
directivas:
```
SSLEngine on
SSLCertificateFile /etc/ssl/private/nombre-sitio.pem
```
Hay que comentar la opción SSLCertificateKeyFile

En la directiva Directory /var/www/html debe de aparecer la siguiente configuracion:
```
<Directory /var/www/html>
SSLRequireSSL
...
</Directory>
```

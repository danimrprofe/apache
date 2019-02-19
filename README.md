# Configuracion HTTPS

La idea es configurar un servidor web que soporte SSL. Para realizar la configuración necesitaremos tener instalado el paquete OpenSSL. Openssl es un kit de herramientas para trabajar con TLS y SSL.

Para ello seguiremos los siguientes pasos:
* Crear una clave privada lo suficientemente fuerte
* Crear una solicitud CSR y enviarla a una CA
* Instalar el certificado que nos devuelva la CA en nuestro servidor web


## Creacion de certificado digital para nuestro site

Crear claves RSA de 2048 bit. La clave se guarde en formato PEM, que es un formato de solo texto,
para poder ser enviada a través de web.

```
sudo openssl genrsa -out clave.key 2048
```
Generar una solicitud de certificado CSR, rellenando los datos. Es un petición formal pidiendo
a la CA que nos firme el certificado.
```
sudo openssl req -new -key clave.key -out solicitud.csr
```
Esta solicitud de certificado creada, la podría enviar a una autoridad de certificación para generase el
certificado crt. 

Como lo que vamos a hacer es instalar un servidor para nuestro propio uso, no hace falta ir a una CA
para que nos certifique.

En este caso lo firmamos nosotros, es decir creamos un certificado auto firmado usando
la clave privada.
```
sudo openssl x509 -req -days 365 -in solicitud.csr -signkey clave.key -out certificado.crt
```
Lo siguiente que haremos es mover la clave y el certificado creados a los directorios que utiliza por
defecto apache y configurar los permisos adecuados (se habrán creado en el directorio en el que
estuviéramos al ejecutar los comandos anteriores).
```
sudo cp clave.key /etc/ssl/private/
sudo cp certificado.crt /etc/ssl/certs/
sudo chown root:ssl-cert /etc/ssl/private/clave.key
sudo chmod 640 /etc/ssl/private/clave.key
sudo chown root:root /etc/ssl/certs/certificado.crt 
```
## Conversión del certificado

Las claves y certificados se pueden guardar en varios formatos diferentes, por lo que podemos 
convertir entre uno y otro formato. Agrupamos en un mismo archivo la clave y el certificado:

```
cat clave.key certificado.crt > certificadoyclave.pem
cp certificadoyclave.pem /etc/ssl/private/
```
## Modificacion del archivo de configuracion del site

Modificaremos el site, comprobando que dentro de virtualhost tenemos las siguientes
directivas:
```
SSLEngine on
SSLCertificateFile /etc/ssl/private/certificadoyclave.pem
```
Hay que comentar la opción SSLCertificateKeyFile para que no afecte.

En la directiva Directory /var/www/html debe de aparecer la siguiente configuracion (apache2.conf)
```
<Directory /var/www/>
SSLRequireSSL
...
</Directory>
```
## Habilitar mod Apache SSL

Habilitamos el mod 
```
sudo a2enmod ssl
```
## Habilitar el site SSL
```
sudo a2ensite default-ssl
```
## Reiniciamos servidor
```
sudo systemctl restart apache2
```

# apache

## Configuracion HTTPS

```
sudo openssl genrsa -out seguro.key 2048
```
sudo openssl req -new -key seguro.key -out seguro.csr
sudo openssl x509 -req -days 365 -in seguro.csr -signkey seguro.key -out seguro.crt

sudo mv seguro.key /etc/ssl/private/
sudo mv seguro.crt /etc/ssl/certs/
sudo chown root:ssl-cert /etc/ssl/private/seguro.key
sudo chmod 640 /etc/ssl/private/seguro.key
sudo chown root:root /etc/ssl/certs/seguro.crt 

cat seguro.key seguro.crt > midominio.org.pem

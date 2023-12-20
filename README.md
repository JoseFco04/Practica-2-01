# Practica-2-01 IAW Jose Francisco León López
## En está práctica vamos a hacerla muy parecida a la práctica 01-07 pero esta vez con el servidor web nginx en vez de apache.
## Para ello debemos de crear una instancia en AWS de ubuntu con los puertos HTTP,HTTPS y SSH abiertos.
### Primero debemos de crear el script del install_lemp con el que vamos a instalar el servidor de nginx entre otras cosas.El script paso por paso se vería así:
#### Mostramos los comandos ejecutados y detenemos si falla el script
~~~
set -ex
~~~
#### Añadimos las variables.
~~~
source .env
~~~
#### Actualizamos los repositorios.
~~~
apt update
~~~
#### Instalamos nginx
~~~
apt install nginx -y
~~~
#### Instalar mysqlserver
~~~
apt install mysql-server -y
~~~
#### Instalacion de paquetes: paquete php-fpm y php-mysql
~~~
apt install php-fpm php-mysql -y
~~~
#### Copiamos el contenido de default de default a /etc/nginx/sites-available/
~~~
cp ../conf/000-default.conf /etc/nginx/sites-available/000-default.conf
~~~
#### Reiniciamos el servicio de nginx
~~~
systemctl restart nginx
~~~
#### Damos permisos al usuario de nginx
~~~
chown -R www-data:www-data /var/www/html
~~~
#### Copiamos el contenido de php a html para comprobar que funciona.
~~~
cp ../php/info.php /var/www/html
~~~
### Una vez hecho este script el siguiente paso es ejecutar el script del deploy del wordpress que tampoco tiene muchos cambios con respeecto al de la práctica 1.7.
### Paso por paso el script se vería así 


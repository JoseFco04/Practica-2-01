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
#### Muestra todos los comandos que se han ejeutado.
set -ex

#### Actualización de repositorios
~~~
sudo apt update
~~~
#### Actualización de paquetes
~~~
# sudo apt upgrade  
~~~
#### Incluimos las variables del archivo .env.
~~~
source .env
~~~
#### Borramos los archivos previos.
~~~
rm -rf /tmp/wp-cli.phar
~~~
#### Descargamos La utilidad wp-cli
~~~
wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -P /tmp
~~~
#### Asignamos permisos de ejecución al archivo wp-cli.phar
~~~
chmod +x /tmp/wp-cli.phar
~~~
#### Movemos los el fichero wp-cli.phar a bin para incluirlo en la lista de comandos.
~~~
mv /tmp/wp-cli.phar /usr/local/bin/wp
~~~
#### Eliminamos instalaciones previas de wordpress
~~~
rm -rf /var/www/html/*
~~~
#### Descargarmos el codigo fuente de wordpress en /var/www/html
~~~
wp core download --path=/var/www/html --locale=es_ES --allow-root
~~~
#### Creamos la base de la bbase de datos y el usuario de la base de datos.
~~~
mysql -u root <<< "DROP DATABASE IF EXISTS $WORDPRESS_DB_NAME"
mysql -u root <<< "CREATE DATABASE $WORDPRESS_DB_NAME"
mysql -u root <<< "DROP USER IF EXISTS $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"
mysql -u root <<< "CREATE USER $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL IDENTIFIED BY '$WORDPRESS_DB_PASSWORD'"
mysql -u root <<< "GRANT ALL PRIVILEGES ON $WORDPRESS_DB_NAME.* TO $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"
~~~
#### Creación del archivo wp-config 
~~~
wp config create \
  --dbname=$WORDPRESS_DB_NAME \
  --dbuser=$WORDPRESS_DB_USER \
  --dbpass=$WORDPRESS_DB_PASSWORD \
  --path=/var/www/html \
  --allow-root
~~~
#### Instalar wordpress.
~~~
wp core install \
  --url=$CERTIFICATE_DOMAIN \
  --title="$wordpress_title" \
  --admin_user=$wordpress_admin_user \
  --admin_password=$wordpress_admin_pass \
  --admin_email=$wordpress_admin_email \
  --path=/var/www/html \
  --allow-root
~~~
#### Actualizamos el core
~~~
wp core update --path=/var/www/html --allow-root
~~~
#### Instalamos un tema:
~~~
wp theme install sydney --activate --path=/var/www/html --allow-root
~~~
#### Instalamos el plugin bbpress:
~~~
wp plugin install bbpress --activate --path=/var/www/html --allow-root
~~~
#### Instalamos el plugin para ocultar wp-admin
~~~
wp plugin install wps-hide-login --activate --path=/var/www/html --allow-root
~~~
#### Habilitar permalinks
~~~
wp rewrite structure '/%postname%/' \
  --path=/var/www/html \
  --allow-root
~~~
#### Modificamos automaticamente el nombre que establece por defecto el plugin wpd-hide-login
~~~
wp option update whl_page $WORDPRESS_HIDE_LOGIN --path=/var/www/html --allow-root
~~~
#### Copiamos el htaccess a /var/www/html
~~~
cp ../conf/.htaccess /var/www/html
~~~
#### Cambiamos al propietario de /var/www/html como www-data
~~~
chown -R www-data:www-data /var/www/html
~~~

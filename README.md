# Tarea 4

En este documento se especifican los pasos a seguir para instalar Wordpress en un contenedor docker con Ubuntu.

## 1. Utiliza la imagen de Ubuntu, version 22

<details>
    <summary>Descargar la imagen de Ubuntu</summary>
</br>
    
```bash
# De no especificar versión, se descargara la más nueva
docker pull ubuntu:22.04
```
    
![Comando Paso1](/img/paso1_1.png)
> Salida por consola ↑

</details>
<details>   
    <summary>Crear un contenedor Docker</summary>
</br>
    
```bash
docker run -d -p 7000:80 --name cnt_ubuntu ubuntu:22.04 tail -f /dev/null
```
![Comando Paso1](/img/paso1_2.png)
> Salida por consola ↑

---
</details>

> [!NOTE]
> Dado que vamos a utilizar el contenedor para Wordpress es necesario asignarle un puerto

## 2. Instalar LAMP en dicho contenedor

LAMP es una pila de software de código abierto que se utiliza como infraestructura de un servidor web.

- Linux
- Apache
- MySQL/MariaDB
- PHP

---

<details>
    <summary>Acceder al contenedor</summary>
</br>
    
```bash
docker exec -it cnt_ubuntu sh
```

</details>

<details>
    <summary>Actualizar lista de paquetes</summary>
</br>
    
```bash
apt update
```

</details>

<details>
    <summary>Instalación</summary>
<br>
<details>
    <summary>Servicio a Servicio</summary>
<br>
    
- Instalar Apache:
  
```bash
apt install -y apache2 apache2-utils
```

- Instalar MariaDB: 

```bash
apt install -y mariadb-server mariadb-client
```

- Iniciar base de datos:

```bash
service mariadb start
```

![Comando Paso2](/img/paso2_12.png)

- Configuración de seguridad de MariaDB:

```bash
mysql_secure_installation
```

- Instalar PHP:
  
```bash
apt install -y php php-mysql libapache2-mod-php
```

- Reiniciar Apache:
  
```bash
service apache2 restart
```

![Comando Paso2](/img/paso2_4.png)

</details>

<details>
    <summary>All in One</summary>
<br>
    
- Instalar pila LAMP:
  
```bash
apt install -y lamp-server^
```

![Comando Paso2](/img/paso2_2.png)
> Salida por consola ↑

- Iniciar Apache:
- 
```bash
service apache2 start
```

- Iniciar base de datos:
  
```bash
service mysql start
```

- Configuración de seguridad de MySQL:
  
```bash
mysql_secure_installation
```

![Comando Paso2](/img/paso2_6.png)
> Salida por consola ↑

- Desabilitar autentificación por Unix Socket:
  
```bash
#Iniciar sesión  
mysql -u root -p

use mysql;
update user set plugin='mysql_native_password' where user='root';
flush privileges;
quit;
```
</details>

---
</details>

> [!IMPORTANT]
> Si durante la configuración de la base de datos ocurre: ERROR 2002 (HY000), se debe a que el servicio de la BD no esta iniciado

<details>
    <summary>Comprobación</summary>
<br>
    
Apache:

```bash
service apache2 status
```

![Comando Paso2](/img/paso2_8.png)
> Salida por consola ↑

MariaDB/MySQL: 

```bash
#MariaDB
service mariadb status

#MySQL
service mysql status
```
![Comando Paso2](/img/paso2_7.png)
> Salida por consola ↑ (MariaDB)

PHP:

```bash
echo "<?php phpinfo(); ?>" | tee /var/www/html/info.php
# Crea una página en el servidor web con la configuración actual de PHP
```

- Comprobación en navegador: ```http://<ip>:<puerto>/info.php```

![Comando Paso2](/img/paso2_3.png)
> Deberia aparecer esta pagina ↑

- Borrar la página:

```bash
rm /var/www/html/info.php
# Es recomendable eliminar la pagina del servidor, pues puede exponer información sensible
```

</details>

## 3. Instalar wordpress en el contenedor

<details>
    <summary>Instalar dependencias</summary>
<br>

```bash
apt install ghostscript \
            php-bcmath \
            php-curl \
            php-imagick \
            php-intl \
            php-json \
            php-mbstring \
            php-mysql \
            php-xml \
            php-zip
```
    
</details>

<details>
    <summary>Preparar el entorno para una instalación de WordPress</summary>
<br>

```bash
#Crear un directorio
mkdir -p /srv/www

#Cambiar la propiedad al usuario www-data
sudo chown www-data: /srv/www

#Descarga la última versión de WordPress y extraerla en /srv/www
curl https://wordpress.org/latest.tar.gz | tar zx -C /srv/www
```

![Comando Paso3](/img/paso3_curl.png)
> Salida por consola ↑ (curl)

--- 
</details>

> [!NOTE]
> Puede que para utilizar el comando curl sea necesario instalar cURL ```apt install curl```

<details>
    <summary>Configurar Apache</summary>
<br>
    
- Crear una página

```bash
nano etc/apache2/sites-available/wordpress.conf
```

Que contenga las siguientes lineas:

```bash
<VirtualHost *:80>
    DocumentRoot /srv/www/wordpress
    <Directory /srv/www/wordpress>
        Options FollowSymLinks
        AllowOverride Limit Options FileInfo
        DirectoryIndex index.php
        Require all granted
    </Directory>
    <Directory /srv/www/wordpress/wp-content>
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>

# Para guardar: Ctrl + X
```

- Habilitar la página:

```bash
# Habilitar el sitio de WP
a2ensite wordpress
# Habilitar el módulo de reescritura
a2enmod rewrite
# Deshabilitar el sitio predeterminado
a2dissite 000-default

# Recargar Apache para aplicar los cambios
service apache2 reload
```
---
</details>

> [!NOTE]
> Para utilizar nano es necesario instalarlo ```apt install nano```

<details>
    <summary>Conprobar acceso a WordPress</summary>
<br>
    
```bash
# (en navegador)
http://<ip>:<puerto>/wp-admin/setup-config.php
```
    
![Comando Paso3](/img/paso3_pruebaWp.png)
>El resultado debería ser el de la imagen

</details>

<details>
    <summary>Configurar la base de datos</summary>
<br>

```bash
mysql -u root

# Crea una nueva base de datos
create database <nombre_BD>;

# Crea un nuevo usuario y aplica una contraseña
create user <nombre_usuario>@localhost identified by '<contraseña>';

# Otorga todos los privilegios al usuario especificado
grant all privileges on <nombre_BD>.* TO <nombre_usuario>@localhost;

# Actualiza los privilegios
fush privileges;

exit;
```
    
</details>

<details>
    <summary>Configurar conexion de Wordpress a la BD</summary>
<br>

```bash
# Copia el archivo de configuración 
cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
```
> Los siguientes comandos ↓ se pueden realizar modificado el archivo de configuracion directamente (ver punto siguiente)

```bash
# Reemplaza el nombre de la base de datos en el archivo de configuración
sed -i 's/database_name_here/<nombre_BD>/' /srv/www/wordpress/wp-config.php

# Reemplaza el nombre del usuario en el archivo de configuración
sudo -u www-data sed -i 's/username_here/<nombre_usuario>/' /srv/www/wordpress/wp-config.php

# Reemplaza la contraseña del usuario en el archivo de configuración
sed -i 's/password_here/<contraseña>/' /srv/www/wordpress/wp-config.php
```
    
</details>

<details>
    <summary>Acceder al archivo configuración de Wordpress</summary>
<br>

```bash
nano /srv/www/wordpress/wp-config.php
```

Cambiamos las claves por las generadas aleatoriamente en:
[https://api.wordpress.org/secret-key/1.1/salt/]( https://api.wordpress.org/secret-key/1.1/salt/ )

![Comando Paso3](/img/punto3_conf.png)
    
</details>

<details>
    <summary>Configurar WordPress</summary>
<br>

```bash
# (en navegador)
http://<ip>:<puerto>/wp-admin
```

En Wordpress tendremos que que configurar la página, dandole titulo, nombre de usuario y contraseña (diferentes a las especificadas en las BD)
 
![Comando Paso3](/img/paso3_confWp_1.png)
> Al final de la configuración saldra la siguiente página ↓

![Comando Paso3](/img/paso3_confWp_2.png)
</details>

## (Opcional) Instalacón phpMyAdmin

PhpMyAdmin es una aplicación web que sirve para administrar bases de datos MySQL mediante una interfaz gráfica.

<details>
    <summary>Instalar phpMyAdmin</summary>
<br>

```bash
apt install phpmyadmin
```
    
</details>

<details>
    <summary>Comprobar funcionamiento</summary>
<br>

```bash
# (en navegador)
http://<ip>:<puerto>/phpmyadmin
```

![Comando Paso3](/img/paso4_compPHP2.png)
>El resultado debería ser el de la imagen

![Comando Paso3](/img/paso4_compPHP.png)
>Un vez iniciado sesión apareceran las diferentes BD del servidor

</details>

## Apuntes finales

>[!NOTE]
>Los servicios de Apache y MariaDB/MySQL se paran si se detiene el contenedor en el que están alojados. Para poder utilizar Wordpress o phpMyAdmin has de iniciarlos antes. 

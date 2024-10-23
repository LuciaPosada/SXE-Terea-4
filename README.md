# Tarea 4
---

## 1. Utiliza la imagen de Ubuntu, tag 22

- Descargamos la imagen: ```docker pull ubuntu:22.04```

    ![Comando Paso1](/img/paso1_1.png)

- Creamos un contenedor: ```docker run -d -p 7000:80 --name cnt_ubuntu ubuntu:22.04 tail -f /dev/null```

    ![Comando Paso1](/img/paso1_2.png)

## 2. Instalar LAMP en dicho contenedor

- Acedemos al contenedor: ```docker exec -it cnt_ubuntu sh```

- Actualizamos los repositorios: ```apt update```

    ![Comando Paso2](/img/paso2_1.png)

- Instalación:

    - Servicio a Servicio:

        - Instalamos Apache: ```apt install -y apache2 apache2-utils```

            ![Comando Paso2](/img/paso2_11.png)

        - Instalamos MariaDB: ```apt install -y mariadb-server mariadb-client```

            ![Comando Paso2](/img/paso2_10.png)

            - Iniciar base de datos: ```service mariadb start```

                ![Comando Paso2](/img/paso2_12.png)

            - Asegurar la instalación de MySQL: ```mysql_secure_installation```

        - Instalamos PHP: ```apt install -y php php-mysql libapache2-mod-php```

        - Reiniciamos Apache: ```service apache2 restart```

            ![Comando Paso2](/img/paso2_4.png)

    - All in One:

        - Instalamos LAMP stack: ```apt install -y lamp-server^```

            ![Comando Paso2](/img/paso2_2.png)

        - Iniciar base de datos: ```service mysql start```

        - Asegurar la instalación de MySQL: ```mysql_secure_installation```

            ![Comando Paso2](/img/paso2_6.png)

        - Nos conectamos a la BD como root: ```mysql -u root -p```

        - Desabilitamos autentificacion por  Unix Socket:
            ```
            use mysql;
            update user set plugin='mysql_native_password' where user='root';
            flush privileges;
            quit;
            ```

> [!IMPORTANT]
> No iniciar la base de datos puede causar ERROR 2002 (HY000)

- Comprobacion:

    - Apache: ```service apache2 status```

        ![Comando Paso2](/img/paso2_8.png)

    - MariaDB/MySQL: ```service mariadb status```

        ![Comando Paso2](/img/paso2_7.png)

    - PHP:

        - Colocamos una pagina PHP en la carpeta del apache: ```echo "<?php phpinfo(); ?>" | tee /var/www/html/info.php```

            ![Comando Paso2](/img/paso2_9.png)

        - Comprobamos en navegador:```http://10.0.9.147:7000/info.php```

            ![Comando Paso2](/img/paso2_3.png)

## 3. Instalar wordpress en el contenedor

- Instalamos depecndencias:
```
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

- Creamos un directorio de instalacion:
```
mkdir -p /srv/www
sudo chown www-data: /srv/www
curl https://wordpress.org/latest.tar.gz | tar zx -C /srv/www

```
> [!NOTE]
> para usar el comando curl es necesario instalar cURL ```apt install curl```

- Configuramos Apache:

    - Creamos una pagina: ``nano etc/apache2/sites-available/wordpress.conf``

    Que contenga las siguientes lineas

![Comando Paso3](/img/paso3_.png)

> [!NOTE]
> Para utilizar nano hay es necesario instalarlo ```apt install nano```
    - Habilitamos la pagina:

    ```
    a2ensite wordpress
    a2enmod rewrite
    a2dissite 000-default

    service apache2 reload
    ```
- Configuramos la base de datos: ```mysql -u root```
    ```
    create database <nombre_BD>;
    create user <nombre_usuario>@localhost identified by '<contraseña>';
    grant all privileges on <nombre_BD>.* TO <nombre_usuario>@localhost;
    fush privileges;
    exit;
    ```
- Configuramos Wordpress para que se conecte a la BD:

    ```
    cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php

    sed -i 's/database_name_here/<nombre_BD>/' /srv/www/wordpress/wp-config.php
    sudo -u www-data sed -i 's/username_here/<nombre_usuario>/' /srv/www/wordpress/wp-config.php
    sed -i 's/password_here/<contraseña>/' /srv/www/wordpress/wp-config.php
    ```

    - Accedemos a la configuracion de wordpress:```nano /srv/www/wordpress/wp-config.php ```

    Cambiamos las claves por las generadas aleatoriamente en:
    [https://api.wordpress.org/secret-key/1.1/salt/]( https://api.wordpress.org/secret-key/1.1/salt/ )

![Comando Paso3](/img/paso3_.png)

- Configurar WordPress:

![Comando Paso3](/img/paso3_confWp_1.png)

![Comando Paso3](/img/paso3_confWp_2.png)

# Tarea 4
---

## 1. Utiliza la imagen de Ubuntu, tag 22

- Descargamos la imagen: ```docker pull ubuntu:22.04```

    ![Comando Paso1](/img/paso1_1.png)

- Creamos un contenedor: ```docker run -d -p 7000:80 --name cnt_ubuntu ubuntu:22.04 tail -f /dev/null```

    ![Comando Paso1](/img/paso1_2.png)

## 2. Instalar LAMP en dicho contenedor

- Acedemos al contenedor: ```docker exec -it cnt_ubuntu sh```

- Actualizamos el repositorio: ```apt update```

    ![Comando Paso2](/img/paso2_1.png)

- Instalación:

    - Servicio a Servicio:

        - Instalamos Apache: ```apt install -y apache2 apache2-utils```

        - Instalamos MariaDB: ```apt install -y mariadb-server mariadb-client```

            - Iniciar base de datos: ```service mariadb start```

> [!IMPORTANT]
> No iniciar la base de datos puede causar ERROR 2002 (HY000)
            - Asegurar la instalación de MySQL: ```mysql_secure_installation```

        - Instalamos PHP: ```apt install -y php php-mysql libapache2-mod-php```

        - Reiniciamos Apache: ```service apache2 restart```

    - All in One:

        - Instalamos LAMP stack: ```apt install -y lamp-server^```

            ![Comando Paso2](/img/paso2_2.png)

        - Iniciar base de datos: ```service mysql start```

        > [!IMPORTANT]
        > No iniciar la base de datos puede causar ERROR 2002 (HY000)
        - Asegurar la instalación de MySQL: ```mysql_secure_installation```

        - Nos conectamos a la BD como root: ```mysql -u root -p```

            > use mysql;
            > update user set plugin='mysql_native_password' where user='root';
            > flush privileges;
            > quit;

- Comprobacion:

    - Apache: ```service apache2 status```

    - MariaDB/MySQL: ```service mariadb status```

    - PHP:

        - Colocamos una pagina PHP en la carpeta del apache: ```echo "<?php phpinfo(); ?>" | tee /var/www/html/info.php```

        - Comprobamos en navegador:```http://10.0.9.147:7000/info.php```


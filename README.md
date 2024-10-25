# TAREA04_SXE

### 1. Utiliza la imagen de Ubuntu tag 22, e instala LAMP en dicho contenedor.

1. Para descargar la imagen de **Ubuntu 22.04**:
```bash
 docker image pull ubuntu:22.04
```
2. Se crea un *contenedor* con la imagen de Ubuntu y se accede a el:
```bash
 docker run -it --name ubuntu22 -p 8000:80 ubuntu:22.04 /bin/bash
 docker start -ai ubuntu22
```

<details>
<summary> 3. Ahora es necesario descargar Apache y MariaDB:</summary>

Actualizar el repositorio de paquetes
    
```bash
sudo apt update
```

Instalar **Apache**:
```bash
sudo apt install -y apache2 apache2-utils
```
Instalar **MariaDB**:
```bash
sudo apt install -y mariadb-server mariadb-client
```

> [!IMPORTANT]
> La opción **-y** responde "sí" automáticamente a cualquier solicitud de confirmación durante la instalación.

Se inicializa MariaDB para poder continuar y usar comandos PHP:
```bash
service mariadb start
```
</details>
    
<details>
<summary> 4. Lo siguiente es descargar PHP</summary>

Se asegura la instalación de MySQL utilizando:
```bash
mysql_secure_installation
```
Y se siguen los pasos indicados en la guía.

Ahora se instala **PHP**:
```bash
apt install -y php php-mysql libapache2-mod-php
```

Y finalmente se instala systemctl y se reinicia el servicio de Apache:

```bash
apt install systemctl
systemctl restart apache2
```
</details>

### Comprobaciones
Para comprobar el correcto funcionamiento hay que entrar en la ip del equipo y poner el siguiente comando:
```bash
echo "<?php phpinfo(); ?>" | tee /var/www/html/info.php
```

<details>
<summary> Una vez hecho esto, accedemos a la web y comprobamos que funciona:</summary>

![imagen](https://github.com/user-attachments/assets/2e9ad564-10ff-4939-8cff-f02fad72791b)
</details>

---
### 2. Instalar wordpress en el contenedor.
<details>
<summary> 1. Se instalan todas las dependencias necesarias:</summary>

```bash
sudo apt install apache2 \
                 ghostscript \
                 libapache2-mod-php \
                 mysql-server \
                 php \
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
<summary> 2. Se instala WordPress utilizando: </summary>

```bash
mkdir -p /srv/www
sudo chown www-data: /srv/www
curl https://wordpress.org/latest.tar.gz | tar zx -C /srv/www
```

> El primer comando crea un directorio, el segundo cambia la propiedad y el ultimo descarga la última versión de WordPress y la extrae

![imagen](https://github.com/user-attachments/assets/b0ec9913-007b-4821-b5cb-b2fbdd01619e)

</details> 

<details>
<summary> 3. Creación y configuración de una página WordPress en Apache: </summary>

- Creación de la página con la siguiente configuración:
    
    Se crea y abre el fichero de configuración
    ```bash
    nano etc/apache2/sites-available/wordpress.conf
    ```
    
    Se pega la siguiente configuración
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
    ```
    
- Habilitar el funcionamiento de la página:
    ```bash
    a2ensite wordpress
    a2enmod rewrite
    a2dissite 000-default
    service apache2 reload
    ```


> El primer comando habilita la web de wordpress, el segundo habilita la reescritura, el tercero desactiva la web predeterminada y el último reinicia el servicio de Apache.

- Para comprobar el correcto funcionamiento de WordPress se accede al link mediante la IP:
    ```bash
    http://10.0.9.153:8000/wp-admin/setup-config.php
    ```

    ![imagen](https://github.com/user-attachments/assets/e6df6b9c-ba44-446f-913e-4bf7b4969fb6)
</details>

<details>
<summary> 4. Configuración de la base de datos MySQL: </summary>


- Creación de una base de datos
    ```bash
    mysql -u root
    CREATE DATABASE wordpress;
    CREATE USER pablo IDENTIFIED BY 'admin';
    GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO pablo;
    FLUSH PRIVILEGES;
    quit
    
    # Iniciar la base de datos
    service mysql start
    ```
- Configuración de la base de datos
    
    Se crea el fichero de configuración y se escribe el texto generado desde wordpress:
    
    ```bash
    touch /srv/www/wordpress/wp-config.php
    nano /srv/www/wordpress/wp-config.php
    ```

    <details>
    <summary> Este es el texto generado en mi caso: </summary>
    
    ```bash
    <?php
    /**
     * The base configuration for WordPress
     *
     * The wp-config.php creation script uses this file during the installation.
     * You don't have to use the website, you can copy this file to "wp-config.php"
     * and fill in the values.
     *
     * This file contains the following configurations:
     *
     * * Database settings
     * * Secret keys
     * * Database table prefix
     * * ABSPATH
     *
     * @link https://developer.wordpress.org/advanced-administration/wordpress/wp-config/
     *
     * @package WordPress
     */
    
    // ** Database settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define( 'DB_NAME', 'wordpress' );
    
    /** Database username */
    define( 'DB_USER', 'pablo' );
    
    /** Database password */
    define( 'DB_PASSWORD', 'admin' );
    
    /** Database hostname */
    define( 'DB_HOST', 'localhost' );
    
    /** Database charset to use in creating database tables. */
    define( 'DB_CHARSET', 'utf8mb4' );
    
    /** The database collate type. Don't change this if in doubt. */
    define( 'DB_COLLATE', '' );
    
    /**#@+
     * Authentication unique keys and salts.
     *
     * Change these to different unique phrases! You can generate these using
     * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
     *
     * You can change these at any point in time to invalidate all existing cookies.
     * This will force all users to have to log in again.
     *
     * @since 2.6.0
     */
    define( 'AUTH_KEY',         '+hE!5`FfPz]k%g/w&m?pn}*AF-LA]?X@EZ[ryclrBzCCh7[ m:28Ms1G%~Fio$px' );
    define( 'SECURE_AUTH_KEY',  'hi{CHVccjo#5FO/tJRMR:#nka^r@~{}jl4g#7+-+>AT_]@Di1{&O7+zhoJ?MyUL:' );
    define( 'LOGGED_IN_KEY',    'i+A3T{_atqWd57:lgow^.08,s3OqYCKEY^]Ds*](m0m@.uR>`mqIQTfDQp}RH $@' );
    define( 'NONCE_KEY',        'K6v0b%/#ud&yOb7)4KRjVJqzM2|be,GaJRp mtet<hxS0MvDRWPZo)la^NaYr<i5' );
    define( 'AUTH_SALT',        '+SYcBw.Z$%%4nwnt=[ B?p1LDO[P2=JO-.P..B=@{&?3Fjy%mVp|@+kWUYFwPN]6' );
    define( 'SECURE_AUTH_SALT', '1Ujkx_/OU1G&</M/%IfD(m^qQ_}:TIvzo<vH{,jBf:N${Ql$cJ0M<%N=O-W.vqeV' );
    define( 'LOGGED_IN_SALT',   'bwjk&)y`y;x$Ouvx1;BcnTz`qDm>?mH80{lLiFmODYxnF(A,vwoPFGoR+^,K]52>' );
    define( 'NONCE_SALT',       '^tm};KO%EnXZHt-&E0v0fA/`mi=|QjH5(nBd?p9bcA2cieWUYqQuY,O+X/_.f% F' );
    
    /**#@-*/
    
    /**
     * WordPress database table prefix.
     *
     * You can have multiple installations in one database if you give each
     * a unique prefix. Only numbers, letters, and underscores please!
     */
    $table_prefix = 'wp_';
    
    /**
     * For developers: WordPress debugging mode.
     *
     * Change this to true to enable the display of notices during development.
     * It is strongly recommended that plugin and theme developers use WP_DEBUG
     * in their development environments.
     *
     * For information on other constants that can be used for debugging,
     * visit the documentation.
     *
     * @link https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/
     */
    define( 'WP_DEBUG', false );
    
    /* Add any custom values between this line and the "stop editing" line. */
    
    
    
    /* That's all, stop editing! Happy publishing. */
    
    /** Absolute path to the WordPress directory. */
    if ( ! defined( 'ABSPATH' ) ) {
    	define( 'ABSPATH', __DIR__ . '/' );
    }
    
    /** Sets up WordPress vars and included files. */
    require_once ABSPATH . 'wp-settings.php';
    ```
    </details>
</details>

<details>
<summary> 5. Creación finalización de wordpress </summary>
    
- Se indica la información que nos pide:
    
![imagen](https://github.com/user-attachments/assets/7ca81cba-5af1-4d6a-aa23-8dac2c88c44b)

- Instalamos WordPress y continuamos:

![imagen](https://github.com/user-attachments/assets/ad29296b-4c58-4907-8de2-e5052624c415)

- Nos logeamos y comprobamos que todo funciona bien:

![imagen](https://github.com/user-attachments/assets/21bebdc3-f6a1-40b3-b715-0bb6e8fc30ad)

En mi caso todo funciona a la perfección.
</details>

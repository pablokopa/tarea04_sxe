# TAREA04_SXE

### 1. Utiliza la imagen de Ubuntu tag 22, y apoyandote en esta guía sigue sus instrucciones para instalar LAMP en dicho contenedor.

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
### 2. Utiliza esta guía para instalar wordpress en el contenedor.
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
> [!NOTE]
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
    
> [!NOTE]
> El primer comando habilita la web de wordpress, el segundo habilita la reescritura, el tercero desactiva la web predeterminada y el último reinicia el servicio de Apache.

- Para comprobar el correcto funcionamiento de WordPress se accede al link mediante la IP:
    ```bash
    http://10.0.9.153:8000/wp-admin/setup-config.php
    ```

    ![imagen](https://github.com/user-attachments/assets/e6df6b9c-ba44-446f-913e-4bf7b4969fb6)
</details> 

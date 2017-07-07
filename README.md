# guia-django-apache

# Poner en producción un proyecto Django utilizando Apache

## 1.- Creamos la estructura para nuestro proyecto
- Como usuario `root` creamos un nuevo usuario en linux llamado `webuser` 
    ```sh
    adduser webuser
    ```

- después de esto nos hará una serie de preguntas este es un ejemplo del output, en donde nos pide la contraseña escribirla y anotarla ya que será la contraseña del usuario que usaremos
    ```sh
    Adding user `webuser' ...
    Adding new group `webuser' (1001) ...
    Adding new user `webuser' (1001) with group `webuser' ...
    Creating home directory `/home/webuser' ...
    Copying files from `/etc/skel' ...
    Enter new UNIX password:
    Retype new UNIX password:
    passwd: password updated successfully
    Changing the user information for urban
    Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
    Is the information correct? [Y/n]
    ```

- Agregar el usuario a la lista de sudoers para poder ejecutar tareas como sudo
    ```sh
    gpasswd -a webuser sudo
    ```
    
- Cambiar al nuevo usuario creado `webuser`
    ```sh
    su - webuser
    ```

- Cambiar al `home` de `webuser`
    ```sh
    cd ~
    ```

- Crear un nuevo directorio para las webs
    ```sh
    mkdir www
    ```
    
## 2.- Instalar virtualenv

- Instalar el paquete `pip` en nuestro sistema.
 
    ```sh
    sudo apt-get install python-pip
    ```
   
- Instalar `virtualenv` en nuestro python
   ```sh
    sudo pip install virtualenv
    ```   
    
## 3.- Clonar nuestra web y echarla a andar

- Acceder a nuestro usuario `webuser` si aún no estamos en él o salimos por alguna razón
   ```sh
    su webuser
    ```
    
- Ir al directorio de `www` en el usuario `webuser`
   
   ```sh
    cd ~/www
    ```

- Clonar nuestro repositorio
   ```sh
    git clone <<url_del_repo>>
    ```

- Acceder al directorio creado por tu repositorio `<<folder_repo>>`, creado al hacer el clone dentro de `www`
   ```sh
    cd <<folder_repo>> #Ej. uscit
    ```
    
- Creamos un entorno virtual con `virtualenv`
   ```sh
    virtualenv -p python3 venv
    ```
 
- Activamos nuestro entorno virtual.
   ```sh
    source venv/bin/activate
    ```
    
- Instalamos los paquetes necesarios para nuestro programa en nuestro entorno virtual
   ```sh
    pip install -r requirements.txt
    ```  

## 4.- Instalar y configurar apache

- Instalar `apache`

    ```sh
    sudo apt-get install apache2
    ```

- Instalar `lib-wsgi` para usar los proyectos de django
    ```sh
    # si usas python 3
    sudo apt-get install libapache2-mod-wsgi-py3
    
    # si usas python 2
    sudo apt-get install libapache2-mod-wsgi
    ```    
    
## 5.- Habilitar nuestro sitio en apache

- Borramos el archivo por default de apache en `/etc/apache2/sites-available`
    ```sh
    sudo rm /etc/apache2/sites-available/000-default.conf
    ```

- Creamos un archivo de configuración para nuestra web en  `/etc/apache2/sites-available`
    ```sh
    sudo nano /etc/apache2/sites-available/<<nombre_web>>.conf #Ej. uscit.conf
    ```
- Aqui un ejemplo del contenido del archivo:
    ```sh
        # Para este archivo consideramos un proyecto django llamado uscit con la siguiente estructura de directorios
        # así que deberias modificarlo según los nombres de los directorios de tu proyecto
        # home
        #   ---> webuser
        #          ---> www
        #                 --->uscit
        #                       ---> src
        #                           ---> manage.py
        #                           ---> uscit
        #                                  ---> settings.py
        #                                  ---> urls.py
        #                                  ---> wsgi.py
        #                           ---> una-app
        #                                  ---> views.py       
        #                                  ---> admin.py
        #                                  ---> urls.py
        #                                  ---> models.py
        #                                  ---> ....
        #                 --->static
        #                       ---> bootstrap.min.css
        #                       ---> jquery.min.css
        #                       ---> styles.css
        #                       ---> scripts.js
        #                       ---> ....
        #                 --->media
        #                       ---> ....
       <VirtualHost *:80>
            ServerName uscit.me
            ServerAdmin ing.cuahutliulloa@gmail.com

            Alias /static /home/webuser/www/uscit/static/
            <Directory /home/webuser/www/uscit/static/>
               Require all granted
             </Directory>

            Alias /media /home/webuser/www/uscit/media
            <Directory /home/webuser/www/uscit/media>
               Require all granted
             </Directory>

            <Directory /home/webuser/www/uscit/src/uscit>
                <Files wsgi.py>
                    Require all granted
                </Files>
            </Directory>

            WSGIDaemonProcess uscitproc python-path=/home/webuser/www/uscit/src/ python-home=/home/webuser/www/uscit/venv/lib/python3.5/site-packages
            WSGIProcessGroup uscitproc
            WSGIScriptAlias / /home/webuser/www/uscit/src/uscit/wsgi.py


            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined

        </VirtualHost>
    ```

    
- Habilitamos nuestro sitio en apache
    ```sh
    sudo a2ensite <<nombre_web>>.conf #ej. uscit.conf
    ```
- Recargamos el servidor apache
    ```sh
    sudo service apache2 reload
    ```    

## 6.- Ajustar permisos en las carpetas para que todo funcione correctamente

- Agregar al usuario `webuser` al grupo `www-data` de apache
    ```sh
    sudo adduser webuser www-data
    ```

- Modificar los permisos en la carpeta `www`
    ```sh
    sudo chown -R webuser:www-data /home/webuser/www
    ```

## 7.- Actualizar el proyecto con los cambios realizados.
    
Una vez implementado el proyecto para actualizarlo con los cambios que se hayan realizado y que todo funcione correctamente se deben realizar los siguientes pasos.

- Ingresar a la ruta del proyecto
    ```sh
    cd ~/www/<<folder_repo>> #Ej cd ~/www/uscit
    ```
    
- Bajar las actualizaciones del proyecto del repositorio git.
    ```sh
    git pull
    ```

- Activar el entorno virtual
   ```sh
    source venv/bin/activate
    ```
    
- Actualizar las librerias del entorno virtual por si se incluyó alguna nueva.
    ```sh
    pip install -r requirements.txt
    ```
    
- Actualizar la Base de Datos por si se realizaron cambios:
    ```sh
    cd src &&  python manage.py migrate && python manage.py makemigrations
    ```

- Recolectar estáticos
    ```sh
    python manage.py collectstatic
    ```

- Volver a establecer los permisos correctos en Linux
    ```sh
    sudo chown -R webuser:www-data /home/webuser/www
    ```

- Recargar apache
    ```sh
    sudo service apache2 reload
    ```

- Validar que apache arrancó correctamente
   ```sh
    sudo service apache2 status
    ```
    
     
### XX.- Solución de problemas

----   

- Si hay un problema al correr la app en apache y dentro de los logs de error aparece un error similar a :
    ```sh
    [error] mod_wsgi (pid=31000): Call to 'site.addsitedir()' failed for '(null)', stopping.
    ```

    entonces habría que cambiar en el archivo de configuración la linea:
    
    ```sh
    WSGIDaemonProcess uscitproc python-path=/home/webuser/www/uscit/src/:/home/webuser/www/uscit/venv/lib/python3.5/site-packages
    ```
    
    por la siguiente:

    ```sh
    WSGIDaemonProcess uscitproc python-path=/home/webuser/www/uscit/src/ python-home=/home/webuser/www/uscit/venv/lib/python3.5/site-packages
    ```

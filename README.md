###Arquitectura:
- Ubuntu 14.04.2 LTS
- Python 2.7.6 

###Softarwe:
- Servidor web: Nginx
- Base de datos: MySQL

1. Ingresar como root (todo los comandos necesitan de root)
sudo su

2. Instalar dependencias necesarias para desarrollar con python en ubuntu:
	
3. Tener ubuntu y dependencias actualizadas:
	apt-get update
	apt-get upgrade
4. Instalar build-essential
	apt-get install -y build-essential 

6. Instalar paquetes de desarrollo de python

	apt-get install python-setuptools python-dev python2.7-dev python-software-properties libpq-dev

7. Instalar librerias necesarias útiles en cierto paquetes

	apt-get install libtiff4-dev libjpeg8-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.5-dev tk8.5-dev

###Instalar software necesario:

1. Instalar Mysql y dependencias para desarrollo
	apt-get install mysql-server mysql-client
	apt-get install libmysqlclient-dev
2. Instalar Nginx
	apt-get install nginx
3. Instalar supervisor (ejecutar y monitorear la tarea de gunicorm siempre)
	apt-get install supervisor
	
###Instalar software para desarrollo en python
1. Instalar PIP
	apt-get install python-pip
2. Instalar vistualenv desde PIP 
	pip install virtualenv

###Iniciar proyecto en Django

1. Elegimos una carpeta de trabajo en mi caso /home/django
	cd /home/django
2. Creamos el Entorno virtual y activamos
	virtualenv proyectoenv
	source proyectoenv/bin/activate
3. A nivel de proyectoenv  el archivo requirements.txt que contiene las dependencias del proyecto es buena practica esto
	touch requirements.txt 

4. Agregamos los primero requerimientos e instalamos. Agregar: 
	Django==1.9.2
	MySQL-python
	Pillow

5. Instalar paquetes: 
	pip install -r requirements.txt

6. Iniciar en django: En la carpeta raiz ejecutamos 
	django-admin startproject proyecto
	
Aquí django crea la carpeta “proyecto” esto porque le pusimos ese nombre tu carpeta base debe quedar así:

	

###Configuración Iniciar y de base de datos:

1. En la primera línea agregamos: 
	```python
	# -*- coding: utf-8 -*-
	```

2. Configuración de Idioma y tiempo:
	```python
	LANGUAGE_CODE = 'es-PE'
	TIME_ZONE = 'America/Lima'
	USE_TZ = False
	Configurar la base de datos Mysql: 
	DATABASES = {
	
	   'default': {
	       'ENGINE': 'django.db.backends.mysql',
	       'HOST' : '127.0.0.1',
	       'USER' : 'usauriobd',
	       'PASSWORD' : 'passwordbd',
	       'NAME': 'nombrebd',
	   }
	}
	```

######Nota: para demo puede dejarlo como sqlite si no has creado una base de datos en mysql



###Configuración de contenido estadísticos (Template, static file y media)

Template (HTML): Yo recomiendo que se guarden todos en una misma carpeta “templates” en el proyecto y recién dentro de template se crea una carpeta para cada app. 

######Nota: otros recomiendan crearlo dentro de cada app pero a mi me parece desordenado y confuso cuando tienes muchas app es cuestión de gustos.

Para configurar la carpeta “templates” buscamos la variable:
	```python
	TEMPLATES = []
	```

dentro de la lista encontramos a ‘DIRS’ : [] ahi agregamos nuestra ruta quedando asi
	```python
	‘DIRS’ : [os.path.join(BASE_DIR ,'templates')]
	```

static file: Para los archivos estáticos CSS/JS/IMG del proyecto es recomendable que esten ubicados en la raiz del proyecto en la carpeta “static” aqui necesitamos configurar 2 variables si no están las agregamos si estan la modificamos si es necesario:
	```python
	STATIC_URL = '/static/'
	#Esta línea de abajo solo en desarrollo  (como ahora estamos en producción no la usamos y la comentamos)
	STATICFILES_DIRS = (os.path.join(BASE_DIR, "static"),) 
	
	#esta línea solo en producción (Como ahora estamos en producción usamos esta)
	STATIC_ROOT = os.path.join(BASE_DIR, "static")
	```

Así debe quedar esa sección:

Archivos multimedia (los cargador en el sistema “upload”): Es recomendable que esto estén en otro server si o si pero si lo queremos en el mismo server debe tener esta configuración:
esto para crear la carpeta media hay otros que le llaman upload:

	```python
	MEDIA_ROOT = os.path.join(BASE_DIR, "media")
	
	MEDIA_URL = '/media/’
	```

El archivo settings.py para esta sección debe quedar asi:

Ahora en el archivo url.py dentro de la carpeta de configuración debe quedar asi (lo que esta en negrita he argegado):
	
	```python
	from django.conf.urls import url
	from django.contrib import admin
	from django.conf import settings
	from django.conf.urls.static import static
	urlpatterns = [
	    url(r'^admin/', admin.site.urls),
	] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
	```

Demo en producción : Antes de configurar nginx vamo a levantar el admin de django para luego manejar sus archivos estáticos: 

En la carpeta de proyecto donde esta el manage.py corremos el servidor:
	```python
	python manage.py runserver 0.0.0.0:80
	```

######Nota: como verás se abre el server de desarrollo en el puerto 80 si tuvieras algo corriendo en el puerto ochenta ejemplo apache lo apagas y también valida que el puerto 80 está abierto en amazon y por ultimo ten a la mano la ip de amazon en mi caso es http://52.35.27.241/ ahi tendrias que ver la pagina de django:


Cancela el comando anterior con CTRL-C y ejecuta este comando para que copie todo los archivos estatico a la carpeta static
	```python
	python manage.py collectstatic
	```
Ojo todo los comandos que corrar manage.py o pip debes estar dentro del entorno virtual 

Configurar Gunicon: gunircon se instala en el entorno virtual
	```python
	pip install gunicorn
	```


por buena practica tambien agregalo al requirements.txt

En nuestra carpeta del entorno virtual “proyectoenv” ingresamos a bin y creamos el archivo gunicorn_start este archivo va a tener un script de configuración que tu tienes que remplazar por tu carpeta 

	https://gist.github.com/neotrons/25875dd7e82967d549a2

Listo una ves creado lo que nos importa ahora es la ruta de nuestro archivo gunicorn_start para este ejemplo seria 			/home/django/proyectoenv/bin/gunicorn_start 

le damos permisos de ejecución: 
	chmod u+x /home/django/proyectoenv/bin/gunicorn_start

Supervisor: Lo que hará supervisor es solo ejecutar gunicorn_start como un servicio 

creamos el archivo de configuración del proyecto 

	touch /etc/supervisor/conf.d/proyecto.conf

lo abrimos y le agregamos el contenido de supervisor_proyecto.conf de esta url:

	https://gist.github.com/neotrons/25875dd7e82967d549a2

Luego actualizamos supervisor 
	supervisorctl reread
	supervisorctl update
	Configuración de nginx 

Primero valida que nginx esté corriendo en el server entrando a la ip si sale un mensaje de bienvenida está activo si no sale eso hay que reiniciar nginx

	service nginx restart 

y verificamos si arranca:

para poder configurar el proyecto es simple creamos un archivo en los site-avaibles de nginx y un enlace simbólico a los site-enables
 
	touch /etc/nginx/sites-available/proyecto
	ln -s /etc/nginx/sites-available/proyecto /etc/nginx/sites-enabled/proyecto

Abrimos el archivo /etc/nginx/sites-available/proyecto  y agreamos el contenido de nginx_proyecto.conf de esta url

	https://gist.github.com/neotrons/25875dd7e82967d549a2

como verás para los archivos estáticos  y media en producción se agrego en el archivo las rutas estáticas :

	location /static/ {
		alias /home/django/proyecto/static/;
	}
	
	location /media/ {
		alias /home/django/proyecto/media/;
	}


reiniciamos nginx e ingresamos a la IP o dominio y te debe funcionar bien entra al admin si te cargan los estilos es que has echo un buen trabajo
http://52.35.27.241/admin

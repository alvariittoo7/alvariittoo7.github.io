---
title: "Servidor virtual con Base de Datos"
date: 2025-06-30  # ¡Asegúrate de que la fecha coincida con el nombre del archivo!
layout: single    # o "post" si prefieres el layout de entrada de blog
---

> **Despliegue** **en** **Servidor** **Virtual** **con** **Base** **de**
> **Datos**
>
> 17-11-2024

<img src="./zbxb0ln2.png"
style="width:5.70125in;height:3.71806in" />

> **Diagrama** **de** **despliegue** **de** **la** **arquitectura**
> **en** **AWS**

Lo primero que hacemos es crear la instancia de EC2; hemos utilizado
Amazon Linux 2023 AMI con la configuración pordefecto, pares
declavevockey, t2.micro…El nombre denuestra instanciaes
**ec2-practica**.

Adicionalmente, hemos decidido usar el servicio **Elastic** **IP** para
tener una IP fija para nuestra EC2, de modo que no cambie cada vez que
se cierra la sesión. La IP es **54.84.202.12**.

Ahora creamos la Base de Datos, para locual usamos**AmazonRDS**
yelegimos **PostgreSQL**. El nombre de la DB es **rds-practica**, el
usuario maestro es **postgres** y su contraseña es **postgres-user**, y
el nombre inicial de la DB es **sample**. En cuanto a la
**configuración,** hemos usado la versión 15.7, con db.t3.micro, la
imagen free tier y conectada a una instancia de EC2, donde
seleccionaremos la instancia que acabamos de crear. El resto es la
configuración por defecto.

Ahora desde la terminal de la instancia de ec2 podemos comprobar el
acceso a la Base de Datos, para ello exportamos la variable local
\$DATABASE_URI:

> export
> DATABASE_URI=postgresql://postgres:postgres-user@rdspractica.cfmn4tysf6zh.us-east-1.rds.amazonaws.com/sample

Ahora ejecutaremos el comando:

> psql \$DATABASE_URI

Si lo hemos hecho bien, nos saldría el **prompt** de la DB que en este
caso sería sample\>.

Ahora tenemos que instalar **pip** e ir a la carpeta del game-store e
instalar los requerimientos (para ello antes hay que pasar la carpeta al
EC2 con scp), y en caso de que no esté instalado **gunicorn**, también
habría que hacerlo para poder lanzar el demonio de la API REST:

sudo dnf install python3-pip pip install -r requeriments.txt pip install
gunicorn

<img src="./yycf4asv.png"
style="width:6.13611in;height:2.21653in" />Ahora tenemos que modificar
el grupo de seguridad de la INSTANCIA para habilitar la escucha en el
puerto 5000, en nuestro caso es el **launch-wizard-5**:

Ahora ejecutamos el daemon con el siguiente mandato:

gunicorn --bind 0.0.0.0:5000 initAlchemy:api --daemon

<img src="./5410hxno.png"
style="width:2.34305in;height:0.33889in" />Con el comando sudo netstat
-tln \| grep 5000 podemos comprobar que el puerto 5000 está en escucha y
con ps aux \| grep gunicorn podemos comprobar que el daemon funciona.

Ahora creamos un bucket con S3 para el swagger con la configuracion por
defecto: **s3-practica**

Una vez creado le pasamos los archivos del swagger, pero antes hay que
cambiar la URL en el **swaggerconfig.yaml.**

> <img src="./px4q03e4.png"
> style="width:4.36417in;height:2.81111in" />Pondremos
> http://ec2-54-84-202-12.compute-1.amazonaws.com:5000/api

En este caso, la API está en el puerto 5000 de nuestra instancia EC2.
Para pasar los archivos simplemente clickamos en el bucket y le damos a
upload.

Si no lo hemos hecho antes, tenemos que permitir que el bucket aloje una
web. Para ello, vamos a **static** **website** **hosting**, le damos a
enable y ponemos el fichero **index.html** como index document.

Una vez hecho esto, tenemos que configurar el acceso a la web. Para
ello, desactivamos el bloqueo de todo el acceso público, y en la
política del bucket ponemos:

> {
>
> "Version": "2012-10-17", "Statement": \[ {
>
> "Sid": "PublicReadGetObject", "Effect": "Allow", "Principal": "\*",
>
> "Action": "s3:GetObject",
>
> "Resource": "arn:aws:s3:::s3-practica/\*" }
>
> \]
>
> }

Esto permite el acceso a cualquier usuario leer el contenido del bucket.
Ahora ponemos la siguiente URL en el navegador y estamos dentro:

http://s3-practica.s3-website.us-east-1.amazonaws.com

<img src="./rd1dz510.png" style="width:6.13333in;height:3.45in" />

Ahora desde aquí podemos hacer las pruebas, donde pone **servers**
eligiremos el que tiene el mismo URL que pusimos anteriormente en el
**swagger-config.yaml**.

> **Casos** **de** **prueba:**
>
> **1.** Buscar el juego con id=2:

<img src="./jzkm50qi.png"
style="width:6.13472in;height:6.71639in" />

> **2.** Buscamos el juego Final Fantasy VII Remake:

<img src="./bzce3zeg.png"
style="width:6.12639in;height:6.63806in" />

> **3.** Añadimos el juego Grand Theft Auto VI y lo buscamos:

<img src="./sjqedvqy.png"
style="width:5.49444in;height:8.3625in" />

<img src="./dejq1knz.png"
style="width:6.12986in;height:6.65819in" />

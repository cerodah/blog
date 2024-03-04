---
title: Accesos no autorizados en Linux - Info
published: true
---

# [](#header-1)Introducción
Este post está dedicado para aprender técnicas comunes para puertas traseras en linux. Basicamente un backdoor es algo que podemos hacer para garatizarnos de poder acceder al sistema de manera constante. Aunque la máquina se apague o se reinicie podremos seguir teniendo acceso.

Obviamente eso no es una vulnerabilidad, si no que son técnicas para mantenernos con acceso al objetivo.
# [](#header-1) Puertas traseras en SSH
Esta es nuestra primera técnica que veremos, ssh backdoors.
Principalmente, esto consiste en dejar nuestras llaves ssh en el directorio de inicio de un usuario. En la mayoría de casos, el directorio sería en el de root. ¿Por qué? Porque es el usuario con mayores privilegios. Para poder hacer eso, tenemos que crear un conjunto de llaves ssh con el comando `ssh-keygen`.

> ![image](https://github.com/cerodah/blog/assets/82907557/2dfda856-50c8-4627-a567-7928c02a7abe)

Ahora que tenemos la llave publica y la privada podemos dejarla en /root/.ssh. No olvides cambiar el nombre de la llave publica, normalmente tiene como extension `.pub` a authorized_keys. Si el directorio no existe, simplemente crealo con `mkdir -p /root/.ssh`, el parametro `-p` asegurará que se cree el directorio .ssh en root y si existe no generará ningún error por pantalla.

Lo más seguro es que tenga que otorgar los permisos necesarios a la llave privada para que la llave sea accesible solamente para el propietario y nadie más (además ssh se quejará si no lo hacemos...). Con el comando `chmod 600 id_rsa` se hace.
Hay que tener algo que en cuenta. Este "backdoor" no está oculto en absoluto, ya que cualquier usuario con permisos necesarios podrá eliminar tanto nuestra llave pública (authorized_keys) como nuestra llave privada!

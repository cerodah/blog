---
title: Atacando Shadow - Info
published: true
---
# [](#header-1) Introducción a /etc/shadow 
El archivo shadow es un componente crucial en sistemas operativos basados en Unix y Linux. En este archivo es donde se almacenan las contraseñas de los usuarios. Este fichero es propiedad del usuario root y contiene el permiso 640.

6 (root) =  permiso de lectura y escritura para el usuario root 
4 (grupo) = Permisos solamente de lectura. Los usarios pertanecientes al grupo podrán leer el archivo
0 (otros) = Ningun permisos para usuarios que no sean el propietario o el grupo.

No voy a enfocarme profundamente en explicar la estructura de este archivo, si no que voy a profundizar más en lo que nos interesa, la explotación (en entornos controlados siempre).

# [](#header-1) Explotación con permiso de escritura
Teniendo la habilidad de poder escribir sobre este archivo, podemos copiar el contenido y posteriormente obtener los hashes y romperlos con hashcat o john.

Imaginamos que siendo el usuario "unix" queremos acceder a bob sin saber su contraseña y nos damos cuenta que podemos leer sobre shadow. 
![Captura de pantalla 2024-03-12 033052](https://github.com/cerodah/blog/assets/82907557/66a56b94-7f61-4efe-b64f-3ba853c59473)

Nos lo copiamos y lo traemos a nuestra máquina. En ella usaremos hashcat para poder crackear la contraseña. Si nos fijamos, el hash empieza por 

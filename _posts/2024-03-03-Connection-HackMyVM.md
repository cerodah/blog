---
title: Connection WriteUp - HackMyVM
published: true
---
[HackMyVM - Connection](https://hackmyvm.eu/machines/machine.php?vm=Connection)

Este es un writeup que hice en 2021 de nivel fácil. En esta máquina enumeramos un servicio SMB en el que nos aprovecharemos para subir nuestra propia shell. Para la escalada utilizaremos `gdb` con permisos SUID para escalar a root.


# [](#header-1)Enumeración
Con el comando `sudo  arp-scan -l | grep -i "PCS" | awk '{print $1}'` obtenemos directamente la ip haciendo un escaneo ARP en nuestra red local filtrando por la nomenclatura PCS utilizada por VirtualBox para definir en red sus dispositivos virtualizados.
> ![image](https://github.com/cerodah/blog/assets/82907557/e9750184-0568-409e-9899-4e1edb19c3c5)




## [](#header-2)Nmap
A través de `nmap -p22,80,139,445 -sV -sC $IP -oN nmap` vemos 4 puertos en los que hay 3 servicios corriendo. 
> ![image](https://github.com/cerodah/blog/assets/82907557/aac8dc27-16df-4f3a-bc4c-c90212968599)


Pasamos directamente a mirar el servidor samba para investigar en el.
> ![image](https://github.com/cerodah/blog/assets/82907557/433574b8-0f2c-47a1-9d40-34bd54be5906)

Listamos los recursos compartidos en el servidor y vemos un recurso llamado share.
> ![image](https://github.com/cerodah/blog/assets/82907557/272800d0-6900-416d-9a96-306759ed9d7f)


# [](#header-1)Explotación
Hay un archivo index.html, por intuición el servidor web se está cargando desde el propio servidor SMB. Por tanto podemos subir nuestra PHP web shell y conseguir acceso.
> ![image](https://github.com/cerodah/blog/assets/82907557/3bfe36a9-2232-4733-ad36-27f6ec9adfc1)

# [](#header-1)Escalada de privilegios 
Una vez obtenida nuestra sesión, siguiendo la metodologia en CTF's podemos enumerar binarios SUID. ¿Qué son los permisos SUID? Básicamente, es un permiso que se puede ejecutar como si tu fueras el propietario del archivo.
Con `find / -perm -4000 2>/dev/null` le estamos diciendo al sistema que quiero que me busques en la carpeta raíz archivos con el bit SUID activado. El numero 4000 es el valor octal correspondiente al bit SUID.
> ![image](https://github.com/cerodah/blog/assets/82907557/f75b4151-e18b-4d57-b4f2-7dad9d89c0aa)

Como buenos analistas que somos, sabemos que existe una [web](https://gtfobins.github.io) en la que hay público articulos de técnicas de escalada de privilegios y uso malicioso de comandos encontrados en binarios del sistema operativo. Principalmente está web se basa en binarios con el bit SUID. Pues bien, en ella encontramos la forma de explotarlo.

![image](https://github.com/cerodah/blog/assets/82907557/864596c5-d8df-40bc-b986-0dc3d7a682de)
```bash
gdb -nx -ex 'python import os; os.execl("/bin/bash", "bash", "-p")' -ex quit
```
y... ¡BOOM! Acceso a root.

![image](https://github.com/cerodah/blog/assets/82907557/3a3d1eb7-a0bd-4ab9-810e-512044acfc10)



```
Happy Hacking ^^
```

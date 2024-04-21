---
layout: single
title: Bypass Windows Defender con Nim.
excerpt: "En este artículo, os muestro como es posible eludir el AV de Windows para obtener un shell inverso estable con nim."
date: 2024-04-21
classes: wide
categories:
  - Research
tags:
  - AV
  - Reverse Shell
  - Bypass
---

## ¿Qué es Nim?
Según el sitio web oficial de NIM, lo definen como un lenguaje de programación que tiene el poder de poder generar archivos ejecutables sin la necesidad de dependencias. Es utilizado para desarrollar aplicaciones para sistemas Windows, BSD, Linux, etc.

## Requisitos previos.
Como es de entender, se necesita tener instalado previamente nim y mingw. Esto se puede completar mediante la comanda:

```bash
Debian: sudo apt install nim mingw-w64
Fedora, CentOS, RHEL: sudo dnf install nim mingw64-gcc
Arch Linux: sudo pacman -S nim mingw-w64-gcc
OpenSUSE: sudo zypper install nim mingw64-gcc
```
Una vez acabada con la instalación debemos de clonar un repositorio de GitHub, el cual es el encargado de hacer nuestra tarea.
```
https://github.com/Sn1r/Nim-Reverse-Shell
```
## Tutorial.
Una vez clonado, debemos de editar las variables v1 y v2, reemplanzadolas por nuestra IP y el puerto que queramos usar.

> ![imagen](https://github.com/cerodah/blog/assets/82907557/79c01442-3105-406a-bb5d-0c68c31c4aa9)

Una vez hecho, debemos de escribir este comando
```bash
nim c -d:mingw --app:gui --opt:speed -o:Prueba.exe rev_shell.nim
```
Desglosamiento:
* **c:** Indicamos al compilador que complile con c.
* **-d:mingw:** Indicamos al compilador que use MinGW como cadena de strings para compilar. Permite ejecutar código nativo de Windows en sistemas Unix y Linux.
* **-app:gui:** Mientras se ejecuta el shell inverso en la máquina víctima (siempre éticos) el proceso se crea en segundo plano y la CLI no se muestra al usuario.
* **--opt:Speed:** Optimizamos el código para agilizar la velocidad de ejecución.
  Bueno... y los otros parámetros ya se puede intuir...
  > ![imagen](https://github.com/cerodah/blog/assets/82907557/c255ed96-3723-47e1-a550-e3861268f1bc)

Activamos un servidor HTTP en nuestra máquina local para poder nosotros descargarlo desde la máquina víctima y a la vez nos ponemos en escucha por el puerto establecido anteriormente (mi caso el 1337, jejeje).

> ![imagen](https://github.com/cerodah/blog/assets/82907557/bce96b4c-5bc1-4bc8-92ca-16c9d09cc772)

El Windows Defender no me ha avisado absolutamente de nada, pero hablaremos de una cosa más adelante.
> ![imagen](https://github.com/cerodah/blog/assets/82907557/e0dd89b7-7d95-4ff3-b2d2-86dc90e7abdb)
Y finalmente entablamos una conexión con la máquina víctima :). Si da el caso en el que se nos cierra la shell debido a restricciones simplemente creamos un nuevo usuario rápidamente con privilegios de administrador y posteriormente iniciar sesión a través de RDP o SSH si estuvieran abiertos los puertos.
```bash
 net user cerodah cerodah123 /add
 net localgroup administrators cerodah /add
```

## No todo es lo que parece.
Si nuestra víctima tuviera otro AV, hay una pequeña parte que podría ser detectado. Nos podemos pasar este archivo a VirusTotal y vemos que 30 AV lo han detectado que es un 43,66%.

> ![imagen](https://github.com/cerodah/blog/assets/82907557/f3fd88b2-4d3c-4cb6-9f8f-27e92aefe5a1)

Y con esto me despido...

```bash
Happy Hacking ^^
```

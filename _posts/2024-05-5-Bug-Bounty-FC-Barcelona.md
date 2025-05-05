---
title: Bug Bounty FC Barcelona
published: true
---
# Informe de Vulnerabilidad: Reflected XSS en fundacion.fcbarcelona.es

## Subdominio afectado

`**https://fundacion.fcbarcelona.es**`

## Descripción de la vulnerabilidad

Se ha identificado una vulnerabilidad de tipo **Reflected Cross-Site Scripting (XSS)** en el subdominio indicado. 

Esta vulnerabilidad permite a un atacante inyectar código JavaScript malicioso en una página web, que se refleja y se ejecuta en el navegador del usuario cuando visita el enlace preparado.

## Funcionamiento de la vulnerabilidad

El XSS se produce cuando una aplicación web recibe datos de un usuario y los muestra posteriormente en una página sin validar o limpiar correctamente ese contenido. En este caso concreto, el parámetro vulnerable refleja el contenido sin ningún tipo de saneamiento, permitiendo así la ejecución del código inyectado.

Cuando un usuario accede a un enlace que contiene el payload malicioso, el JavaScript incluido se ejecuta en el contexto de la sesión de ese usuario. Esto puede permitir acciones como:

- Robo de cookies de sesión
- Modificación del contenido de la página
- Redirección del usuario a un sitio externo controlado por el atacante

## Payload utilizado

A continuación, se muestra el payload utilizado para demostrar la explotación de la vulnerabilidad:

`‹script src=data:text/j\0061v\0061&#115&#99&#114&#105&#112&#116,\u0061%6C%65%72%74(/XSS/)›‹/script›`

### Cadena desofuscada

`<script src="data:text/javascript,alert(/XSS/)"></script>`

Este payload hace uso de codificaciones como hexadecimal y Unicode para evadir filtros de seguridad comunes.

## Prueba de concepto (PoC)

Se ha creado un vídeo de prueba de concepto donde se demuestra la ejecución del XSS a través del enlace malicioso. Este vídeo muestra el comportamiento de la página cuando se carga el código inyectado y la ejecución del `alert()` como prueba.

**[PoC: XSS_PoC_FCBarcelona.mp4]**
<video controls width="600" autoplay="">
  <source src="https://github.com/cerodah/blog/raw/refs/heads/master/assets/XSS_PoC_FCBarcelona(1).mp4">
  Tu navegador no soporta videos HTML5.
</video>

## Recomendaciones

Para mitigar esta vulnerabilidad, se recomienda al equipo de desarrollo aplicar las siguientes medidas:

- **Validación y limpieza de entrada:** Todos los datos recibidos por parte del usuario deben ser validados y limpiados antes de ser mostrados.
- **Encabezados de seguridad HTTP:** Implementar políticas CSP (Content Security Policy) para limitar las fuentes de scripts y prevenir la ejecución de código no autorizado.
- **Revisión del código:** Realizar auditorías regulares del código para detectar y corregir vectores de inyección de código.

---

*Este informe ha sido generado con fines educativos y de investigación en ciberseguridad.*

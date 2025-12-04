---
title: Bypassing UAC Windows
published: true
---

Después de tanto tiempo sin publicar nada por aquí, hoy me decido a publicar una nueva entrada en este blog. 
Esta vez se trata de como evadir el control de cuenta de usuario (UAC) en hosts Windows. Vamos a ver formas comunes de eludir la función de seguridad dispoibile que tiene implementado Windows conocido como user Account Control.
Esta característica permite que cualquier proceso se ejecute con privilegios bajos (independientemente de quien lo este ejecutando). 

Desde la perpesctiva de un atacante, eludir UAC es esencial para poder salir de entornos restrictivos y elevar los privilegios en el host objetivo. También mientras aprendemos técnicas de bypass, veremos cualquier alerta que se pueda activar y artefactos que podrian crearse el sistema y que Blue Team pueda detectar.
Para poder entender esta sala, recomiendo que se tenga conocimientos básicos de como funciona Windows en su núcleo.


# [](#header-1) ¿Qué es UAC?
User Account Control es una funcion de seguridad en Windows, que en su medida, obliga a cualquier proceso nuevo se ejecute en el contexto de una cuenta sin privilegios. Esta política es implementada a los procesos iniciados por cualquier usuario, incluido los administradores del sistema. La idea es que no podemos basarnos solamente en la identidad del usuariuo para determinar si se debenm autorizar ciertas acciones.

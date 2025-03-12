---
title: Vulnerabilitat XSS a fundacion.fcbarcelona.es
published: true
---

S'ha identificat una vulnerabilitat de Reflected Cross-Site Scripting (XSS) en el subdomini de la Fundació del FC Barcelona. Aquest informe detalla els aspectes tècnics, el payload utilitzat i les recomanacions per a la mitigació.

## Subdomini afectat
https://fundacion.fcbarcelona.es

## Descripció de la vulnerabilitat
La vulnerabilitat permet a un atacant injectar codi JavaScript maliciós en una pàgina web, que s'executa al navegador de l'usuari que visita un enllaç manipulat. Això pot resultar en el robatori de dades sensibles (com cookies de sessió), modificació de continguts o redirecció a llocs maliciosos.

## Funcionament de la vulnerabilitat
El XSS es produeix quan l'aplicació web no neteja ni valida les dades d'entrada dels usuaris abans d'incloure-les a la pàgina. En aquest cas, un paràmetre d'entrada reflecteix les dades sense cap tipus de sanejament, permetent l'execució de codi arbitrari.

### Payload utilitzat
```html
<script src=data:text/j\0061v\0061&#115&#99&#114&#105&#112&#116, \u0061%6C%65 %72%A4/XSS/)></script>
## Cadena desofuscada
<script src="data:text/javascript,alert(/XSS/)"></script>
Aquest payload evita filtres mitjançant codificació hexadecimal i Unicode.


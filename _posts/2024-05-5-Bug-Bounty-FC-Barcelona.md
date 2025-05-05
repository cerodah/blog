---
title: Bug Bounty FC Barcelona.
published: true
---
# Informe de Vulnerabilitat: Reflected XSS a fundacion.fcbarcelona.es

## Subdomini afectat

                                              https://fundacion.fcbarcelona.es

## Descripció de la vulnerabilitat

S'ha identificat una vulnerabilitat de tipus **Reflected Cross-Site Scripting (XSS)** en el subdomini indicat. 

Aquesta vulnerabilitat permet a un atacant injectar codi JavaScript maliciós dins d'una pàgina web, que es reflecteix i s'executa al navegador de l'usuari quan visita l'enllaç preparat.

## Funcionament de la vulnerabilitat

El XSS es produeix quan una aplicació web rep dades d’un usuari i les mostra posteriorment a una pàgina sense validar o netejar correctament aquest contingut. En aquest cas concret, el paràmetre vulnerable reflecteix el contingut sense cap sanejament, permetent així l’execució del codi injectat.

Quan un usuari accedeix a un enllaç que conté el payload maliciós, el JavaScript inclòs s’executa en el context de la sessió d’aquell usuari. Això pot permetre accions com:

- Robar cookies de sessió
- Modificar el contingut de la pàgina
- Redirigir l’usuari a un lloc extern controlat per l’atacant

## Payload utilitzat

A continuació, es mostra el payload utilitzat per demostrar l’explotació de la vulnerabilitat:

`‹script src=data:text/j\0061v\0061&#115&#99&#114&#105&#112&#116,\u0061%6C%65%72%74(/XSS/)›‹/script›`

### Cadena desofuscada

`<script src="data:text/javascript,alert(/XSS/)"></script>`

Aquest payload fa ús de codificacions com hexadecimal i Unicode per evadir filtres de seguretat habituals.

## Prova de concepte (PoC)

S’ha creat un vídeo de prova de concepte on es demostra l’execució del XSS a través de l’enllaç maliciós. Aquest vídeo mostra el comportament de la pàgina quan es carrega el codi injectat i l’execució del `alert()` com a prova.

**[PoC: XSS_PoC_FCBarcelona.mp4]**
<video controls width="600" autoplay="">
  <source src="https://github.com/cerodah/blog/raw/refs/heads/master/assets/XSS_PoC_FCBarcelona(1).mp4">
  Tu navegador no soporddta videos HTML5.
</video>


## Recomanacions

Per mitigar aquesta vulnerabilitat, es recomana al departament de desenvolupament aplicar les següents mesures:

- **Validació i neteja d’entrada:** Totes les dades rebudes dels usuaris han de ser validades i netejades abans de ser mostrades.
- **Encapçalaments de seguretat HTTP:** Implementar polítiques CSP (Content Security Policy) per limitar les fonts de scripts i prevenir l’execució de codi no autoritzat.
- **Revisió del codi:** Realitzar auditories regulars del codi per detectar i corregir vectors d’injecció de codi.

---

*Aquest informe ha estat generat amb finalitats educatives i d'investigació en ciberseguretat.*

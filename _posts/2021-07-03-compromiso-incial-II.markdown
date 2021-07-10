---
title: Compromiso Incial II - Working ..
date: 2021-07-03 19:33:00 +02:00
tags:
- Cobalt Strike
license: false
show_edit_on_github: false
show_subscribe: false
---

El objetivo de está serie de artículos es aprender técnicas tantas ofensivas como defensivas. Por ello, habrá dos apartados, uno principal demostrando técnicas para recabar, en este caso información, y como determinar su defensa.


![image025.png](/uploads/InitialAccess2/image025.png)

![image026.png](/uploads/InitialAccess2/image026.png)


Si deseas proponer alguna mejora, o algún comentario, ya sea duda o simplemente conocer más sobre esto, por favor, sin miedo escribe por telegram o twitter :)


### Infraestructura:

- Debian 10 , corriendo ELK
- Windows 10, ordenador del usuario, corriendo sysmon.
- Kali Linux, máquina atacante

A nivel de herramientas, utilizaré CobaltStrike, como command and control.

# Acceso Inicial

El objetivo de esta lab será mostrar el funcionamiento de cobalt strike , y su utilización, utillizando técnicas como ofuscación y spear phising, a através de macros VBA y excel 4.0


### Cobalt Strike

Lo interesante de cobalt strike es la posibilidad de modificar los comportamientos, que processos spameamos , como las llamadas a las api hacen las injeciones, parámetros de red. 



![image001.png](/uploads/InitialAccess2/image001.png)


Como decía anteriormente, la capacidad de modificar los comportamientos es una de las técnicas interesantes de cobalt strike.

Existen repositorios, como el siguiente, https://github.com/xx0hcd/Malleable-C2-Profiles

Estos perfiles proporcionan la capacidad de modificar los indicadores.

Dentro de cobalt, tenemos la herramienta c2lint, que nos permite comprobar si el profile es correcto, así como ver la información que genera.

A modo de ejemplo, utilizando el template de [youtube](https://raw.githubusercontent.com/xx0hcd/Malleable-C2-Profiles/master/normal/youtube_video.profile)

![image003.png](/uploads/InitialAccess2/image003.png)


Y observamos el tráfico generado vía wireshark

![image004.png](/uploads/InitialAccess2/image004.png)



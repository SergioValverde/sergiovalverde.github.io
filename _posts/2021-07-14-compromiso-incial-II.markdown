---
title: Compromiso Incial II
date: 2021-07-14 19:33:00 +02:00
tags:
- Cobalt Strike
- SharpShooter
- Excel 4.0
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

Como vemos es bastante interesante, así como totalmente modificable y destacable, el crear nuestros propios templates, e imitar comportamientos que nos encontremos dentro de una red.



El primer paso, será crear nuestro listener



![image005.png](/uploads/InitialAccess2/image005.png)


Una vez creado el listener, tenemos que crear el payload que descargará nuestro beacon.


![image006.png](/uploads/InitialAccess2/image006.png)

Nos generará una instrucción para realizar la descarga:

![image007.png](/uploads/InitialAccess2/image007.png)



### Ofuscación

Al ser herramientas muy utilizadas, son cazadas por los antivirus. Con este fin, la recomendación es ofuscar nuestros payloads.


Utilizamos la herramienta **Invoke-Stealth**



![image008.png](/uploads/InitialAccess2/image008.png)

Los AV´s detectan strings o cadenas de texto para identicar patrones, asique utilizamos la opción Chimera.

La instrucción la copiamos en un script, y le pasamos invoke-stealth:

![image009.png](/uploads/InitialAccess2/image009.png)



Esta ejecución nos provoca múltiples procesos, aunque por un lado puede parecer que nos protege contra los AV´s, esto es incorrento, AV salta.

![image010.png](/uploads/InitialAccess2/image010.png)

EL primer proceso creado, 

![image011.png](/uploads/InitialAccess2/image011.png)

![image012.png](/uploads/InitialAccess2/image012.png)

Los eventos siguientes, con el ID 4104, conocido como proceso de ejecución de comandos, siguen este mismo patrón:

![image013.png](/uploads/InitialAccess2/image013.png)

![image014.png](/uploads/InitialAccess2/image014.png)

Como vemos está opción es totalmente inviable para ser ejecutada en un entorno real.





### Creación de macros


En esta parte del ejercicio, mi intentión era ver las macros XML 4.0, ya que están teniendo gran tirón por parte de los ciberdelicuentes.

Y la verdad es que me ha aparecido bastante interesante como funcionan. Incluso, como veremos más adelante, las dificultades para analizar un fichero de este tipo.


Primer paso, debemos generar un payload con cobalt strike


![image015.png](/uploads/InitialAccess2/image015.png)


![image027.png](/uploads/InitialAccess2/image027.png)

Una vez generado nuestro payload, debemos eliminar los null bytes con msfvenom:

![image028.png](/uploads/InitialAccess2/image028.png)

Y utilizaremos la herramienta SharpShooter de parte del equipo de MDSec.

![image029.png](/uploads/InitialAccess2/image029.png)

Una vez generado el fichero, debemos exportarnos a nuestro equipo. Como datos vemos que el fichero es identificado como malicioso. 

![image030.png](/uploads/InitialAccess2/image030.png)

Por lo tanto, nos genererá alertas de seguridad 

![image031.png](/uploads/InitialAccess2/image031.png)

![image032.png](/uploads/InitialAccess2/image032.png)

## Ejecución

En este paso, ejecutaremos el binario con el objetivo de obtener una conexión entre ambos equipo y también, para ver que información estamos generando.

El fichero creado tiene la siguiente imágen:


![image019.png](/uploads/InitialAccess2/image019.png)

Una vez, se ejecuten las macros, se producirá la conexión a nuestro command and control.

![image034.png](/uploads/InitialAccess2/image034.png)

Se nos genera la siguiente información, información relacionada con la descarga del fichero:

![image035.png](/uploads/InitialAccess2/image035.png)

La primera alerta, la creación de un proceso

![image036.png](/uploads/InitialAccess2/image036.png)

![image037.png](/uploads/InitialAccess2/image037.png)

La segunda alerta, creación de un fichero

![image038.png](/uploads/InitialAccess2/image038.png)

![image039.png](/uploads/InitialAccess2/image039.png)

Una tercera alerta, un nuevo proceso creado

![image040.png](/uploads/InitialAccess2/image040.png)

![image041.png](/uploads/InitialAccess2/image041.png)

Reference a la conexión con el C2

![image042.png](/uploads/InitialAccess2/image042.png)

![image043.png](/uploads/InitialAccess2/image043.png)



## Analizando Macros XML 4.0



A la hora de analizar fichero offices tenemos diferentes formas ade llevarlo acabo, existen herramientas automatizadas como anyrun o VT. U otras herramientas como oledump, olevba.

**A la hora de analizar un incidente de seguridad, realizar un análisis automatizado puede que sea la opción más correcta. Una rápida respuesta a esta situción es crítico.**


La muestra que analizaremos se trata de **TrickBot**.

https://bazaar.abuse.ch/sample/888614bdf13110e3934ff3a852074f6cde2998db6631d443bdee73dec5ecfe30/

Curiosidad: En las bases de datos como [bazar-abuse](https://bazaar.abuse.ch/browse/), los usuarios se integran en botnets, con el fin de obtener toda posible muestra de malware y así estudiar los comportamientos y nuevas features.



Una vez lo descomprimimos, como se nos indica, se trata de un fichero xlsx


![image044.png](/uploads/InitialAccess2/image044.png)

Podemos utilizar el hash del fichero para obtener más información.

![image045.png](/uploads/InitialAccess2/image045.png)

Otbetemos información interesante, como dominios o ip´s

Tenemos la opción de AnyRun, para ver la muestra en movimiento.

![image046.png](/uploads/InitialAccess2/image046.png)

Como vemos en los procesos, podemos entender que una vez ejecuta las macros lo que hace es dropear un fichero .dll y este es cargado con rundll32.exe

Utilizar oledump, pero en este caso es interesante, ya que no nos muestra información sobre macros VBA.

![image047.png](/uploads/InitialAccess2/image047.png)

Oledump, tiene plugins que nos da un aporte extra de conocimiento, en este caso, utilizaremos el plugin_biff.py:

`oledump.py trickbot.xlsx -p plugin_biff.py`

![image048.png](/uploads/InitialAccess2/image048.png)



Como vemos, la información es mucho más extensa. Pero sin embargo, al estar el fichero protegido por password no funciona del todo bien…


Si queremos obtener mayor información sobre el fichero, podemos desencriptarlo con **msoffcrypto-crack.py**

![image049.png](/uploads/InitialAccess2/image049.png)

Una vez obtenido la password, necesitamos desencriptar este fichero, para ello utilizamos **msoffcrypto-tool**

![image050.png](/uploads/InitialAccess2/image050.png)


Una vez obtenido el fichero desencriptado, podemos analizar con **xmldeobfuscator**

![image051.png](/uploads/InitialAccess2/image051.png)


OS suenan las formulas del Excel? Ejem ejem…

![image052.png](/uploads/InitialAccess2/image052.png)




Esto es una pequeña introducción al mundo de las macros, que sin duda, es un tema interesante para investigar para tanto red como blue team.

* * *

## Artículos Relacionados

**Thanks To:**

### Cobalt

* Riccardo Ancarani for initial access workshop´s, [3](https://labs.f-secure.com/blog/attack-detection-fundamentals-initial-access-lab-3/) and [4](https://labs.f-secure.com/blog/attack-detection-fundamentals-initial-access-lab-4/)

* Raphael Mudge for Malleable C2, [read ](https://cobaltstrike.com/help-malleable-c2) and [video](https://www.youtube.com/watch?v=0AOQqd6OQzk)


* Pablo Ambite for [Aprovechando de Microsoft Team para persistir y ocultar ](https://www.blackarrow.net/es/aprovechando-microsoft-teams-para-persistir-y-ocultar-el-trafico-de-cobalt-strike/ )


* Assaf Dahan for [Operation Cobalt Kitty ](https://attack.mitre.org/docs/training-cti/Cybereason%20Cobalt%20Kitty%20-%20original%20report.pdf)


* Joel Gámez for [Invoke-Stealth ](https://darkbyte.net/ofuscando-scripts-de-powershell-con-invoke-stealth/)

### Macros

* Marcin Ogorzelski for [The first step in Excel 4.0 for Red Team](https://blog.stmcyber.com/excel-4-0-for-red-team/)

* Joe Leon & Matt Grandy for [Offensive Maldocs in 2020](https://www.youtube.com/watch?v=RW5U9yxilf4) and [pdf](https://github.com/FortyNorthSecurity/Presentations/blob/master/Offensive%20Maldocs%20in%202020.pdf)

* Stan Hegt for [Old school: evil Excel 4.0 macros (XLM)](https://outflank.nl/blog/2018/10/06/old-school-evil-excel-4-0-macros-xlm/)


* Pieter Ceelen and Stan Hegt [Office in Wonderland](https://www.youtube.com/watch?v=9ULzZA70Dzg) and [pdf](https://i.blackhat.com/asia-19/Thu-March-28/bh-asia-Hegt-MS-Office-in-Wonderland.pdf)


* Daniel Heinsen for [Phishing in a Macro-less World](https://www.youtube.com/watch?v=WlR01tEgi_8)

### SharpShooter

* D. Chell for [Cracking the Perimeter with SharpShooter](https://www.youtube.com/watch?v=z89xNXLsXLU) and [pdf](https://hackinparis.com/data/slides/2019/talks/HIP2019-Dominic_Chell-Cracking_The_Perimeter_With_Sharpshooter.pdf)
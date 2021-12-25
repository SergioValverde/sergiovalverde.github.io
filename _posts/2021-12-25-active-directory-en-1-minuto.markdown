---
title: Active Directory en 1 minuto
date: 2021-12-25 13:30:00 +01:00
tags:
- AD
sharing: false
license: false
show_edit_on_github: false
show_subscribe: false
---

**Uno de los problemas de prácticar con AD puede ser los recursos necesarios, tiempo dedicado en montar un entorno de pruebas.**

Aún teniendo un equipo potente, puede ser tedioso trabajar con máquinas virtuales. 

En este blog os resumo la idea de **snaplabs**.

A través de AWS, nos permite crear laboratios en apenas segundos, son templates, totalmente modificables.

![snap2.png](/uploads/AD/snap2.png)

Tenemos diferentes opciones, el límite son nuestra imáginación, por un coste rídiculo.

En mi caso, he generado un laboratorio de AD - 2016, donde tenemos un Windows Server 2016, y otro WS 2016 promocionado a Domain Control.

![snap1.png](/uploads/AD/snap1.png)

----------------------------------------

Creando el laboratorio, utilizamos el siguiente script.
https://github.com/WazeHell/vulnerable-AD/blob/master/vulnad.ps1

Antes de ello, debemos de realizar unas configuraciones.

**Primero**, realizar la instalación del modulo de AD para PowerShell

`Install-WindowsFeature -Name "RSAT-AD-PowerShell" –IncludeAllSubFeature`

**Segundo**, desactivar la característica de active monitoring, del windows defender. Necesario para cuando importemos a nuestro equipo el script de WazeHell, no nos genere problemas.

`Set-MpPreference -DisableRealtimeMonitoring $TRUE`

**Tercero**, importarno el script, en mi caso, realizo la descarga desde mi kali linux

`certutil.exe -urlcache -split -f "http://IP/vulnad.ps1`

**Cuarto**, importamos el script
`. .\vulnad.ps1`

**Quinto y último paso**, correr el script

`Invoke-VulnAD -UsersLimit 100 -DomainName snaplabs.local`



Y ya tendríamos instalado las siguientes características:

•	Abusing ACLs/ACEs
•	Kerberoasting
•	AS-REP Roasting
•	Abuse DnsAdmins
•	Password in Object Description
•	User Objects With Default password (Changeme123!)
•	Password Spraying
•	DCSync
•	Silver Ticket
•	Golden Ticket
•	Pass-the-Hash
•	Pass-the-Ticket
•	SMB Signing Disabled



### Consideraciones

**Eliminar la información Banner Grabbing del script, ya que nos genera errores **

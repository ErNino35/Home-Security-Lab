# Fase 1 — Reconocimiento

## Introducción
El reconocimiento es la primera fase de cualquier auditoría de seguridad.
El objetivo es obtener información sobre la red y las máquinas objetivo
sin realizar ningún ataque. Cuanta más información recopilemos, mejor
podremos planificar las siguientes fases.

## Objetivos de esta fase
- Descubrir qué máquinas están activas en la red
- Identificar los puertos abiertos de la máquina víctima
- Conocer los servicios y versiones que corren en cada puerto
- Analizar el tráfico de red con Wireshark

## Entorno
- **Atacante:** Kali Linux — 192.168.85.128
- **Víctima:** Metasploitable 2 — 192.168.85.129

---

## 1.1 · Escaneo de red (Ping Scan)

**Comando:**
```bash
nmap -sn 192.168.85.0/24
```

**¿Qué hace?**
Detecta qué máquinas están activas en la red sin escanear puertos.

**Resultado:**

<img width="587" height="309" alt="escaneo_red" src="https://github.com/user-attachments/assets/b6ff1756-cc50-443e-be28-2b3c1e713792" />

**Conclusión:**
Se han detectado 4 hosts activos: el gateway (192.168.85.1),
el DHCP de VMware (192.168.85.254), Kali (192.168.85.128)
y Metasploitable (192.168.85.129).



---

## 1.2 · Escaneo de puertos y servicios (Version Scan)

**Comando:**
```bash
nmap -sV 192.168.85.129
```

**¿Qué hace?**
Escanea todos los puertos de la máquina víctima e identifica qué servicio
y versión exacta corre en cada puerto abierto.

**Resultado:**

<img width="995" height="647" alt="image" src="https://github.com/user-attachments/assets/dea72c5f-4dd3-41ba-a1d8-b0fd88415305" />


**Servicios destacados encontrados:**

| Puerto | Servicio | Versión | Riesgo |
|---|---|---|---|
| 21 | FTP | vsftpd 2.3.4 | 🔴 Crítico — backdoor conocida |
| 22 | SSH | OpenSSH 4.7p1 | 🟡 Medio |
| 23 | Telnet | Linux telnetd | 🔴 Crítico — sin cifrado |
| 25 | SMTP | Postfix smtpd | 🟡 Medio |
| 53 | DNS | ISC BIND 9.4.2 | 🟡 Medio |
| 80 | HTTP | Apache 2.2.8 | 🟡 Medio |
| 111 | RPC | rpcbind 2 | 🟡 Medio |
| 139 | NetBIOS | Samba 3.X-4.X | 🔴 Crítico |
| 445 | SMB | Samba 3.X-4.X | 🔴 Crítico — sin dominio |
| 512 | exec | netkit-rsh rexecd | 🔴 Crítico — sin autenticación |
| 513 | login | rlogin | 🔴 Crítico — sin autenticación |
| 514 | shell | Netkit rshd | 🔴 Crítico — sin autenticación |
| 1099 | Java-RMI | GNU Classpath | 🟡 Medio |
| 1524 | Bindshell | Root shell | 🔴 Crítico — acceso root directo |
| 2049 | NFS | 2-4 | 🟡 Medio |
| 2121 | FTP | ProFTPD 1.3.1 | 🔴 Crítico |
| 3306 | MySQL | 5.0.51a | 🟡 Medio |
| 5432 | PostgreSQL | 8.3.0-8.3.7 | 🟡 Medio |
| 5900 | VNC | protocol 3.3 | 🔴 Crítico |
| 6000 | X11 | access denied | 🟡 Medio |
| 6667 | IRC | UnrealIRCd | 🔴 Crítico |
| 8009 | AJP | Apache Jserv | 🟡 Medio |
| 8180 | HTTP | Apache Tomcat | 🟡 Medio |

**Conclusión:**
Metasploitable tiene más de 20 puertos abiertos con servicios vulnerables.
Los más críticos son vsftpd 2.3.4 (backdoor), el puerto 1524 (shell de root
directamente accesible) y los servicios rsh antiguos sin autenticación.
Estos serán los objetivos principales en la Fase 2.


---

## 1.3 · Escaneo avanzado (Aggressive Scan)

**Comando:**
```bash
nmap -A -T4 192.168.85.129
```

**¿Qué hace?**
Combina detección de SO, versiones de servicios y scripts automáticos
de reconocimiento contra cada puerto. Es el escaneo más completo de Nmap.

**Resultado:**

<img width="1056" height="836" alt="image" src="https://github.com/user-attachments/assets/6ba6f52a-d742-4031-a3b3-5bc85b414f2c" />


<img width="1339" height="832" alt="image" src="https://github.com/user-attachments/assets/3a13ce56-745e-4a5a-b415-776be3c02a53" />


<img width="983" height="452" alt="image" src="https://github.com/user-attachments/assets/c5ba8237-daf0-47b2-8bfd-8707a20e2bff" />


**Hallazgos más importantes:**

| Hallazgo | Detalle | Riesgo |
|---|---|---|
| FTP anónimo | Acceso sin usuario ni contraseña al puerto 21 | 🔴 Crítico |
| vsftpd 2.3.4 | Versión con backdoor conocida | 🔴 Crítico |
| UnrealIRCd 3.2.8.1 | Versión con backdoor conocida (puerto 6667) | 🔴 Crítico |
| MySQL expuesto | Revela versión, protocolo y salt interno | 🔴 Crítico |
| SMB sin firma | Permite ataques de tipo relay | 🔴 Crítico |
| PostgreSQL | Certificado SSL caducado desde 2010 | 🟡 Medio |
| VNC puerto 5900 | Versión antigua con autenticación débil | 🔴 Crítico |
| RPC lista servicios | Expone servicios internos públicamente | 🟡 Medio |
| Kernel Linux | Versión 2.6.9-2.6.33 (2004-2009) | 🔴 Crítico |
| SMB1 únicamente | No soporta SMB2, protocolo obsoleto | 🔴 Crítico |

**Conclusión:**
El escaneo avanzado confirma múltiples backdoors conocidas (vsftpd, UnrealIRCd),
servicios que exponen información sensible (MySQL, RPC) y configuraciones
peligrosas (SMB sin firma, FTP anónimo, certificados caducados).
El sistema operativo es un kernel de Linux extremadamente desactualizado.
La máquina está lista para ser explotada en la Fase 2.

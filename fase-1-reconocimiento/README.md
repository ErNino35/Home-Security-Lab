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
| 80 | HTTP | Apache 2.2.8 | 🟡 Medio |
| 139/445 | SMB | Samba 3.X-4.X | 🔴 Crítico — sin dominio |
| 1524 | Bindshell | Root shell | 🔴 Crítico — acceso root directo |
| 3306 | MySQL | 5.0.51a | 🟡 Medio |
| 5432 | PostgreSQL | 8.3.0-8.3.7 | 🟡 Medio |
| 512/513/514 | rsh/rexec | Netkit | 🔴 Crítico — sin autenticación |

**Conclusión:**
Metasploitable tiene más de 20 puertos abiertos con servicios vulnerables.
Los más críticos son vsftpd 2.3.4 (backdoor), el puerto 1524 (shell de root
directamente accesible) y los servicios rsh antiguos sin autenticación.
Estos serán los objetivos principales en la Fase 2.

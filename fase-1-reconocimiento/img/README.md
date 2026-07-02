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

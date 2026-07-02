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

**Comando:**
```bash
nc 192.168.85.129 1524
```


<img width="688" height="313" alt="image" src="https://github.com/user-attachments/assets/698538a7-8c2d-4f0f-9b4b-ae01e8720acb" />



**¿Qué es una bindshell?**
Una bindshell es una terminal que está escuchando en un puerto esperando
que alguien se conecte. En este caso está en el puerto 1524 con permisos
de root — cualquiera que se conecte tiene control total de la máquina
sin necesitar usuario ni contraseña.

**Resultado:**

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

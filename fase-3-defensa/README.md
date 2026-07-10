# Fase 3 — Defensa y Detección (Blue Team)

## Introducción
En esta fase cambiamos de rol — de atacante a defensor. Instalaremos
Wazuh, un SIEM (Security Information and Event Management) open source
y gratuito, para detectar los mismos ataques que hemos realizado en
la Fase 2. El objetivo es entender cómo se ven los ataques desde el
lado de la defensa y qué alertas generan.

## ¿Qué es un SIEM?
Un SIEM es una plataforma que centraliza y analiza logs de seguridad
de distintos sistemas en tiempo real. Detecta comportamientos sospechosos,
genera alertas y permite investigar incidentes. Es una herramienta
fundamental en cualquier equipo de ciberseguridad defensiva (Blue Team).

## Objetivos de esta fase
- Instalar Wazuh SIEM en Kali Linux
- Conectar Metasploitable como agente monitoriazdo
- Repetir los ataques de la Fase 2 y observar las alertas generadas
- Documentar qué detecta Wazuh y qué no

## Entorno
- **Atacante / SIEM:** Kali Linux — 192.168.85.128
- **Víctima / Agente:** Metasploitable 2 — 192.168.85.129

---

# VPN (Site-to-Site & Client VPN)

## Qué es

Una **VPN en AWS** permite conectar redes externas (on‑premises o usuarios remotos) con una **VPC** usando **internet** y **IPsec**.

No es una solución de alto rendimiento, sino una **opción rápida y flexible** para conectividad híbrida.

---

## Tipos de VPN en AWS

### 1️⃣ Site-to-Site VPN

Conecta un **datacenter on‑premises** con una **VPC**.

**Componentes:**

* Virtual Private Gateway (VGW) **o** Transit Gateway (TGW)
* Customer Gateway (CGW)
* Túneles IPsec (normalmente 2 para HA)

**Características clave:**

* Usa **internet público**
* Cifrado IPsec
* Alta disponibilidad lógica (2 túneles)
* Latencia **variable**
* Throughput limitado

📌 **Examen:** no se espera configuración detallada.

---

### 2️⃣ Client VPN

Permite que **usuarios remotos** (portátiles) accedan a recursos dentro de la VPC.

**Casos típicos:**

* Teletrabajo
* Acceso administrativo

📌 Muy poco preguntada en SAA.

---

## Cuándo usar VPN (examen)

* Necesidad de conexión **rápida**
* **Bajo coste**
* Tráfico moderado
* No se requiere latencia estable

📌 Frase típica:

> "Necesitamos conectar nuestro datacenter a AWS rápidamente y con bajo coste"

➡️ **VPN**

---

## Cuándo NO usar VPN

* Aplicaciones sensibles a latencia
* Tráfico alto y constante
* Requisitos de rendimiento predecible

---

## Ventajas

* Barata
* Rápida de desplegar
* Cifrada

## Desventajas

* Depende de internet
* Latencia impredecible
* Menor throughput

---

## Trampas de examen

* VPN **no** es dedicada
* VPN **sí** usa internet
* VPN puede coexistir con Direct Connect como backup

---

## Resumen rápido

| Aspecto           | VPN      |
| ----------------- | -------- |
| Medio             | Internet |
| Latencia          | Variable |
| Coste             | Bajo     |
| Tiempo despliegue | Rápido   |
| Seguridad         | IPsec    |
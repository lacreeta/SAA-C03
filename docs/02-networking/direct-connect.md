
# Direct Connect

## Qué es

**AWS Direct Connect (DX)** es una conexión **física dedicada** entre tu red on‑premises y AWS.

No pasa por internet.

---

## Características clave

* Conexión privada
* Latencia **estable y predecible**
* Alto throughput
* Coste elevado
* Tiempo de provisión alto (semanas)

---

## Casos de uso

* Workloads críticos
* Aplicaciones sensibles a latencia
* Grandes volúmenes de datos
* Requisitos regulatorios

📌 Frase típica de examen:

> "Necesitamos una conexión privada con latencia consistente"

➡️ **Direct Connect**

---

## Arquitectura habitual

* Enlace DX principal
* VPN como **backup**

📌 Esta combinación aparece mucho en preguntas.

---

## Ventajas

* Estabilidad
* Rendimiento alto
* No depende de internet

## Desventajas

* Coste alto
* No es inmediato
* Complejidad operativa

---

## Trampas de examen

* Direct Connect **no cifra por defecto**
* Puede combinarse con VPN para cifrado
* No sustituye a Security Groups ni NACLs

---

## Comparativa VPN vs Direct Connect

| Característica | VPN      | Direct Connect  |
| -------------- | -------- | --------------- |
| Medio          | Internet | Enlace dedicado |
| Latencia       | Variable | Estable         |
| Coste          | Bajo     | Alto            |
| Tiempo         | Rápido   | Lento           |
| Throughput     | Limitado | Alto            |

---

## Regla de oro para SAA

* **Rápido y barato** → VPN
* **Estable y crítico** → Direct Connect
* **Producción seria** → DX + VPN backup

---

## Nivel esperado en el examen

* Saber **cuándo elegir cada uno**
* Entender trade-offs
* NO configuración técnica

---

> Estos apuntes están pensados específicamente para **AWS SAA-C03**.

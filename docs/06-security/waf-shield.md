# AWS WAF & AWS Shield — Protección de aplicaciones y DDoS (SAA-C03)

> Este documento cubre **WAF y Shield**, dos servicios que en el examen aparecen
> casi siempre **juntos** y con un patrón muy claro:
>
> - **WAF** → ataques a nivel aplicación (L7)
> - **Shield** → ataques de denegación de servicio (DDoS)
>
> El objetivo aquí no es memorizar features, sino **saber cuándo elegir cada uno,
> por qué y qué NO hacen**, que es donde el examen suele meter trampas.

---

## Mapa mental rápido

| Necesidad del enunciado | Servicio |
|------------------------|----------|
| Proteger app web de SQLi / XSS / bots | **WAF** |
| Limitar requests, reglas personalizadas | **WAF** |
| Proteger contra DDoS | **Shield** |
| DDoS básico, automático | **Shield Standard** |
| DDoS crítico, SLA y soporte | **Shield Advanced** |

---

# 1) AWS WAF — Web Application Firewall

## 1.1 Qué es AWS WAF
AWS WAF es un **firewall de capa 7 (aplicación)** que:
- Inspecciona **requests HTTP/HTTPS**
- Aplica **reglas** antes de que lleguen a tu aplicación
- Bloquea o permite tráfico según patrones

👉 Piensa en WAF como:
> *“un filtro inteligente delante de tu aplicación web”*

---

## 1.2 Dónde se puede asociar WAF (MUY de examen)

AWS WAF se asocia a:
- **Application Load Balancer (ALB)**
- **API Gateway**
- **CloudFront**

📌 Trampa clásica:
- ❌ WAF **NO** se asocia a EC2 directamente
- ❌ WAF **NO** se asocia a NLB

---

## 1.3 Qué tipo de ataques bloquea WAF

### Ataques comunes
- SQL Injection (SQLi)
- Cross-Site Scripting (XSS)
- Path traversal
- Bots conocidos
- Requests malformadas

📌 En examen:
> *“Protect web app from common attacks”* → **WAF**

---

## 1.4 Tipos de reglas en WAF

### Managed Rules (las más usadas)
- Reglas mantenidas por AWS
- Cubren ataques comunes
- Fáciles de activar

👉 **Respuesta típica de examen**:
> *“Use AWS Managed Rules”*

---

### Custom Rules
- Basadas en:
  - IP
  - headers
  - query strings
  - body
  - geolocation
- Útiles para lógica específica

---

### Rate-based Rules (MUY importante)
- Limitan requests por IP
- Protegen contra:
  - brute force
  - bots
  - picos maliciosos

📌 En examen:
> *“Limit requests per IP”* → **WAF rate-based rule**

---

## 1.5 Qué WAF NO hace (para descartar opciones)

- ❌ No protege contra DDoS volumétrico masivo
- ❌ No cifra datos
- ❌ No gestiona identidad
- ❌ No protege tráfico no HTTP (TCP/UDP)

---

# 2) AWS Shield — Protección DDoS

## 2.1 Qué es AWS Shield
AWS Shield es un servicio de **protección contra DDoS**.

👉 Piensa en Shield como:
> *“el airbag automático de AWS contra DDoS”*

---

## 2.2 Shield Standard
- **Activo por defecto**
- Sin coste adicional
- Protege contra:
  - ataques volumétricos
  - SYN floods
  - UDP floods

📌 En examen:
> *“Basic DDoS protection”* → **Shield Standard**

---

## 2.3 Shield Advanced

### Qué añade
- Protección avanzada
- Soporte del **DDoS Response Team (DRT)**
- SLA de disponibilidad
- Métricas y alertas avanzadas
- Protección de costes (DDoS cost protection)

### Cuándo elegirlo (señales claras)
- Aplicaciones **críticas**
- Requisitos de SLA
- Ataques DDoS frecuentes o sofisticados
- Presupuesto alto

📌 En examen:
> *“Mission-critical application + DDoS”* → **Shield Advanced**

---

## 2.4 Qué Shield NO hace

- ❌ No filtra SQLi/XSS (eso es WAF)
- ❌ No aplica reglas de aplicación
- ❌ No inspecciona payloads HTTP

---

# 3) WAF vs Shield (comparativa clave)

| Característica | WAF | Shield |
|--------------|-----|--------|
| Tipo de ataque | App (L7) | Red/volumen |
| SQLi / XSS | ✅ | ❌ |
| Rate limiting | ✅ | ❌ |
| DDoS | ❌ | ✅ |
| Asociación | ALB, API GW, CF | Automático (infra) |
| Coste | Bajo/medio | Std: gratis / Adv: caro |

📌 Regla mental:
- Ataque **a la app** → WAF
- Ataque **a la red/volumen** → Shield

---

# 4) WAF + Shield juntos (patrón típico)

Arquitectura clásica:
```
Internet
   ↓
CloudFront / ALB
   ↓
WAF (L7)
   ↓
Aplicación
```

Shield protege:
- la infraestructura (L3/L4)

WAF protege:
- la aplicación (L7)

📌 En examen, muchas veces la respuesta correcta es:
> **“Use AWS WAF with AWS Shield”**

---

# 5) Casos típicos de examen

### Caso 1
“Proteger una web contra SQL injection y XSS”

👉 **AWS WAF (managed rules)**

---

### Caso 2
“Limitar el número de requests por IP”

👉 **WAF rate-based rule**

---

### Caso 3
“Aplicación crítica sufre ataques DDoS frecuentes”

👉 **Shield Advanced**

---

### Caso 4
“Protección básica contra DDoS sin coste adicional”

👉 **Shield Standard**

---

### Caso 5
“Proteger API Gateway de tráfico malicioso”

👉 **AWS WAF asociado a API Gateway**

---

# 6) Trampas típicas SAA-C03

- ❌ Elegir Shield para SQLi/XSS
- ❌ Elegir WAF para DDoS volumétrico
- ❌ Asociar WAF a EC2 directamente
- ❌ Pensar que Shield Standard hay que activarlo

---

# 7) Labs recomendados (solo los que valen la pena)

## 🧪 Lab 1 — WAF con ALB (RECOMENDADO)
**Objetivo:** ver reglas en acción

Pasos:
1. ALB con una app simple
2. Asociar AWS WAF
3. Activar managed rules
4. Probar request maliciosa (ej. query sospechosa)

👉 Muy didáctico y rápido.

---

## 🧪 Lab 2 — Rate-based rule
**Objetivo:** entender limitación por IP

Pasos:
1. Crear rate-based rule
2. Generar múltiples requests
3. Ver bloqueo automático

---

## 🧠 Labs NO necesarios
- Shield Advanced (caro)
- Integraciones complejas

Para SAA-C03 basta con entender el concepto.

---

# 8) Mini-resumen final

- **WAF** → protege aplicaciones web (L7)
- **Shield** → protege contra DDoS
- Shield Standard → siempre activo
- Shield Advanced → caro, crítico
- WAF se asocia a ALB / API GW / CloudFront
- SQLi/XSS → WAF
- DDoS → Shield

---

## Cierre
WAF y Shield son **servicios de protección complementarios**.
Si entiendes bien la diferencia entre **ataque de aplicación** y **ataque de red**,  
la respuesta correcta en el examen suele salir sola.

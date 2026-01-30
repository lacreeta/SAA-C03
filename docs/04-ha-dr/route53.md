# Route 53 — SAA-C03

> Enfoque: **DNS + decisiones de arquitectura**, políticas de routing y alta disponibilidad.

---

## 1) Qué es Route 53
**Amazon Route 53** es el servicio de **DNS gestionado** de AWS.
- DNS autoritativo
- Altamente disponible
- Integrado con servicios AWS (ELB, CloudFront, S3, etc.)

📌 Examen: Route 53 aparece casi siempre ligado a **alta disponibilidad**, **failover** o **routing inteligente**.

---

## 2) Conceptos básicos de DNS (examen)
- **Domain**: ejemplo.com
- **Record**: A, AAAA, CNAME, ALIAS, etc.
- **TTL**: tiempo que un resolver cachea la respuesta

📌 Examen:
- TTL bajo → cambios/failover más rápidos
- TTL alto → menos queries DNS (más cache)

---

## 3) Tipos de records importantes

### 3.1 A / AAAA
- Apuntan a IPv4 / IPv6
- Uso clásico

### 3.2 CNAME
- Apunta a otro nombre DNS
- No se puede usar en el root domain

### 3.3 ALIAS (clave AWS)
- Similar a CNAME pero:
  - Funciona en el root domain
  - No tiene coste por query
  - Apunta a recursos AWS (ELB, CloudFront, S3 website, etc.)

📌 Examen:
- Root domain → **ALIAS**, no CNAME

---

## 4) Políticas de routing (MUY IMPORTANTE)

### 4.1 Simple routing
- Un único destino
- Sin health checks

### 4.2 Weighted routing
- Reparte tráfico por porcentaje
- Casos:
  - blue/green
  - canary releases

📌 “Enviar 90% a A y 10% a B” → **Weighted**

---

### 4.3 Latency-based routing
- Envía al endpoint con **menor latencia** desde el usuario
- Ideal para usuarios globales

📌 “Usuarios globales, menor latencia” → **Latency routing**

---

### 4.4 Failover routing
- Primary / Secondary
- Basado en **health checks**

📌 “Si falla A, ir a B” → **Failover routing**

---

### 4.5 Geolocation routing
- Basado en **ubicación del usuario**
- Ej: Europa → EU endpoint, US → US endpoint

📌 Diferente de latency routing.

---

### 4.6 Geoproximity routing
- Basado en distancia geográfica + bias
- Menos común en SAA (saber que existe)

---

### 4.7 Multi-value routing
- Devuelve múltiples IPs
- Health checks básicos
- No es un load balancer real

📌 Trampa:
- Multi-value ≠ ELB

---

## 5) Health checks
Route 53 puede:
- Comprobar endpoints HTTP/HTTPS/TCP
- Integrarse con failover routing
- Comprobar endpoints fuera de AWS

📌 Examen:
- “Detectar endpoint caído y redirigir tráfico” → **Health check + Failover routing**

---

## 6) Route 53 + otros servicios AWS

### ELB
- ALIAS → ALB / NLB
- Patrón clásico de HA

### CloudFront
- Route 53 → CloudFront → orígenes

### S3 website
- ALIAS al bucket configurado como website
- Muy típico en preguntas

---

## 7) Trampas típicas de examen
- Route 53 **no es** un load balancer
- ALIAS solo existe en Route 53
- Latency ≠ Geolocation
- TTL alto puede retrasar un failover

---

## 8) Mini-resumen
- Route 53 = DNS
- ALIAS para recursos AWS
- Weighted → porcentajes
- Latency → usuarios globales
- Failover → HA/DR
- Health checks integrados

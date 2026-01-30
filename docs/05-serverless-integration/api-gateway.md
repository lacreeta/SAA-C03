# API Gateway — SAA-C03 (DETALLADO)

## 1. Qué es API Gateway
Amazon API Gateway es un servicio totalmente gestionado que permite **crear, publicar, mantener, monitorizar y proteger APIs** a cualquier escala. Es una pieza central en arquitecturas **serverless** y **microservicios**.

En el examen SAA-C03, API Gateway aparece casi siempre junto a:
- Lambda
- Autenticación
- Serverless
- Desacoplamiento

👉 Mentalidad de examen: *“necesito exponer lógica sin servidores”*.

---

## 2. Tipos de APIs

### 2.1 REST API
- Más completa
- Soporta:
  - Validación de requests
  - Transformaciones de payload
  - Autorizadores avanzados
- Más cara y con más latencia

📌 Úsala cuando el enunciado mencione:
- Autorización compleja
- Transformaciones
- Features avanzadas

---

### 2.2 HTTP API
- Más simple
- Menor coste
- Menor latencia
- Integración directa con Lambda

📌 Regla de examen:
> Si no piden nada especial → **HTTP API**

---

### 2.3 WebSocket API
- Comunicación bidireccional
- Tiempo real (chat, juegos, dashboards)

📌 Poco frecuente en SAA.

---

## 3. Integraciones

### 3.1 Lambda (la más importante)
Patrón clásico:
```
Cliente → API Gateway → Lambda → Respuesta
```

Ventajas:
- Totalmente serverless
- Escala automática
- Pago por uso

---

### 3.2 Integración con ALB
- API Gateway puede enrutar a un ALB
- Útil en migraciones híbridas

---

### 3.3 Mock integrations
- Devuelve respuestas sin backend
- Útil para tests

---

## 4. Seguridad

### 4.1 Autenticación
- IAM
- Cognito (JWT)
- Lambda Authorizers (custom logic)

📌 Examen:
- “Usuarios externos autenticados” → Cognito
- “Acceso interno AWS” → IAM

---

### 4.2 Throttling y quotas
- Límite de requests
- Protección contra abuso
- Mitiga ataques básicos de DoS

---

## 5. Escalabilidad
- Totalmente gestionada
- Escala automáticamente
- No necesitas ASG ni ELB

📌 Trampa:
- API Gateway **no es un load balancer**

---

## 6. Observabilidad
- Logs en CloudWatch
- Métricas: latencia, errores 4xx/5xx
- Integración con X-Ray

---

## 7. Casos de uso típicos (examen)
- Backend serverless
- Frontend → API → Lambda
- Microservicios desacoplados

---

## 8. Trampas típicas
- API Gateway no ejecuta lógica
- No sustituye a Lambda
- No es ELB

---

## 9. Mini-resumen
- Entrada HTTP a arquitecturas serverless
- Seguridad integrada
- Escala automática

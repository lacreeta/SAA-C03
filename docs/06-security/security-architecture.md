# Security Architecture en AWS — Diseño seguro (SAA-C03)

> Este documento no es sobre **un servicio concreto**, sino sobre **cómo se diseña seguridad en AWS**.
> En el examen SAA-C03, muchas preguntas **no dicen “usa WAF” o “usa IAM” directamente**,
> sino que describen un escenario y esperan que reconozcas **el patrón de seguridad correcto**.
>
> El objetivo aquí es:
> - entender **cómo AWS piensa la seguridad**
> - saber **qué capa resuelve qué problema**
> - razonar arquitecturas seguras, no memorizar listas

---

## 0) Idea clave (la que lo une todo)

> **La seguridad en AWS se diseña por capas (defense in depth)**  
> Ningún servicio por sí solo es suficiente.

Cada capa cubre un tipo de riesgo distinto:
- identidad
- red
- aplicación
- datos
- detección
- respuesta

---

# 1) Shared Responsibility aplicado a la arquitectura

Antes de diseñar seguridad, hay que recordar:

| AWS se encarga de | Tú te encargas de |
|------------------|------------------|
| Seguridad del cloud | Seguridad en el cloud |
| Infra física | Configuración de servicios |
| Hardware | IAM, cifrado, red, datos |

📌 En el examen:
> *“Who is responsible for X?”*  
→ piensa primero en **Shared Responsibility**.

---

# 2) Defense in Depth (pilar central)

## 2.1 Qué significa realmente
No es “poner muchos servicios”, sino:
- **múltiples controles independientes**
- si uno falla, otro sigue protegiendo

Ejemplo:
- IAM evita acceso no autorizado
- WAF bloquea ataques web
- GuardDuty detecta anomalías
- CloudTrail audita acciones

---

## 2.2 Capas típicas en AWS

```
Usuario
  ↓
Identidad (IAM, MFA)
  ↓
Red (SG, NACL, WAF, Shield)
  ↓
Aplicación (Auth, validaciones)
  ↓
Datos (KMS, Secrets)
  ↓
Detección (CloudTrail, GuardDuty)
```

📌 Este diagrama mental **resuelve muchas preguntas**.

---

# 3) Capa de Identidad (IAM)

### Objetivo
- Controlar **quién puede hacer qué**

### Servicios clave
- IAM Users / Roles
- Policies
- MFA
- STS

### Patrones de arquitectura
- Roles > Users
- Acceso temporal
- Least privilege

📌 Trampa:
- Identidad ≠ red
- Si el problema es “quién”, **IAM**.

---

# 4) Capa de Red

### Objetivo
- Limitar **desde dónde** se puede acceder

### Servicios clave
- Security Groups (stateful)
- NACLs (stateless)
- VPC design
- WAF / Shield

### Patrones
- Subnets privadas para backends
- ALB en subnets públicas
- No exponer instancias directamente

📌 Trampa:
- SG ≠ IAM
- SG controla tráfico, no permisos de API.

---

# 5) Capa de Aplicación

### Objetivo
- Proteger la lógica de negocio

### Servicios / mecanismos
- Autenticación (Cognito, JWT, etc.)
- Validaciones
- Rate limiting
- AWS WAF

📌 En examen:
- Ataques SQLi/XSS → **WAF**
- Bots / brute force → **rate-based rules**

---

# 6) Capa de Datos

### Objetivo
- Proteger información sensible

### Servicios clave
- KMS
- Secrets Manager
- SSM Parameter Store
- Cifrado en reposo y en tránsito

### Patrones
- Encryption at rest + in transit
- No hardcodear secretos
- Acceso controlado por IAM

📌 Trampa:
- KMS no gestiona permisos, cifra.
- IAM decide quién puede usar la clave.

---

# 7) Capa de Detección y Auditoría

### Objetivo
- Saber **qué ha pasado** y **qué está pasando**

### Servicios clave
- CloudTrail
- CloudWatch
- GuardDuty
- AWS Config

### Patrones
- Logs centralizados
- Alertas
- Compliance continuo

📌 En examen:
- “Detect suspicious activity” → GuardDuty
- “Audit who changed what” → CloudTrail
- “Evaluate compliance over time” → Config

---

# 8) Capa de Respuesta (automática)

Aunque SAA-C03 no entra profundo, conceptualmente:

### Ejemplo
- GuardDuty detecta amenaza
- EventBridge lanza Lambda
- Lambda bloquea acceso o notifica

📌 Esto muestra **madurez de arquitectura**, aunque no pidan implementarlo.

---

# 9) Patrones de arquitectura segura (muy de examen)

## 9.1 Web application segura
```
Internet
  ↓
CloudFront
  ↓
WAF + Shield
  ↓
ALB
  ↓
EC2 / ECS (privadas)
```

Protecciones:
- DDoS (Shield)
- App attacks (WAF)
- Red (SG)
- Identidad (IAM Roles)

---

## 9.2 Arquitectura serverless segura
```
Cliente
  ↓
API Gateway
  ↓
WAF
  ↓
Lambda (Role IAM)
  ↓
DynamoDB (KMS)
```

Claves:
- Sin servidores expuestos
- IAM roles
- KMS para datos

---

## 9.3 Acceso a datos cross-account
```
Cuenta A
  ↓ AssumeRole
Cuenta B
  ↓
Recurso (policy)
```

Claves:
- Trust policy
- Resource-based policy
- Least privilege

---

# 10) Trampas típicas del examen

- ❌ Resolver todo con IAM
- ❌ Pensar que un solo servicio basta
- ❌ Exponer backends directamente a internet
- ❌ Confundir detección con prevención
- ❌ Olvidar cifrado

---

# 11) Cómo razonar una pregunta de security (pasos)

1. ¿El problema es **identidad**? → IAM
2. ¿Es **red/tráfico**? → SG / NACL / WAF / Shield
3. ¿Es **aplicación**? → WAF / Auth
4. ¿Son **datos**? → KMS / Secrets
5. ¿Es **detección/auditoría**? → CloudTrail / GuardDuty / Config

👉 Elige la capa correcta, luego el servicio.

---

# 12) Mini-resumen para memorizar

- Seguridad = capas
- Defense in depth
- Identidad ≠ red ≠ datos
- Cifrar + controlar acceso
- Detectar + auditar

---

## Cierre
Si entiendes **arquitectura de seguridad**, los servicios individuales encajan solos.
Este fichero es el “pegamento” que hace coherente todo el bloque de Security.

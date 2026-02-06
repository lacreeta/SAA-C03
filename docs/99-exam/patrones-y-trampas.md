# Patrones y Trampas — AWS SAA-C03

> Este documento recoge **los errores más comunes** y **los patrones que el examen repite**.
> Léelo el día antes o la mañana del examen.

---

## 1) Patrones que se repiten

### “Minimize operational overhead”
👉 Serverless / Managed services

### “Highly available within a region”
👉 Multi-AZ (ALB, RDS Multi-AZ)

### “Disaster recovery across regions”
👉 Route 53 Failover + DR strategy

### “Secure access without credentials”
👉 IAM Roles + STS

### “Do not hardcode credentials”
👉 Secrets Manager / SSM

### “Track who did what”
👉 CloudTrail

### “Detect suspicious activity”
👉 GuardDuty

### “Evaluate compliance over time”
👉 AWS Config

---

## 2) Trampas clásicas (MUY frecuentes)

### ❌ Confundir HA con DR
- Multi-AZ ≠ multi-region
- Route 53 ≠ HA instantáneo

### ❌ Pensar que IAM cifra
- IAM controla acceso
- KMS cifra

### ❌ Usar SNS como cola
- SNS = push
- SQS = buffer

### ❌ Pensar que Lambda es para todo
- 15 min máx
- stateless

### ❌ Pensar que CloudWatch audita
- Auditoría = CloudTrail

---

## 3) Comparaciones rápidas que caen

| Si dudas entre | Pregúntate |
|---------------|-----------|
| SNS vs SQS | ¿push o buffer? |
| EventBridge vs SNS | ¿evento AWS o notificación simple? |
| WAF vs Shield | ¿app o DDoS? |
| HA vs DR | ¿seguir vivo o recuperar? |
| Config vs CloudTrail | ¿estado o acción? |

---

## 4) Trampas de red

- ❌ Abrir EC2 a internet cuando hay ALB
- ❌ Usar NACL para lógica compleja
- ❌ Pensar que SG es stateless

---

## 5) Trampas de CloudFormation

- ❌ Pensar que CFN es CI/CD
- ❌ No prever rollback
- ❌ Mezclar datos críticos con compute
- ❌ Cambiar recursos a mano (drift)

---

## 6) Trampas de seguridad

- ❌ Pensar que Shield bloquea SQLi
- ❌ Pensar que WAF bloquea DDoS volumétrico
- ❌ Pensar que GuardDuty bloquea ataques
- ❌ Pensar que Macie cifra datos

---

## 7) Cómo ganar puntos en preguntas largas

1. Lee el **objetivo principal**
2. Identifica la **restricción clave** (coste, tiempo, gestión)
3. Elimina opciones que:
   - requieren más gestión
   - no atacan el problema real
4. Elige la opción **más simple que cumpla todo**

---

## 8) Frases mágicas del examen

- “without managing servers”
- “with minimal operational effort”
- “highly available and fault tolerant”
- “secure and compliant”
- “automatically”

👉 Suelen apuntar a la respuesta correcta.

---

## Cierre
El examen **no busca que lo sepas todo**, sino que **elijas bien**.
Estos patrones y trampas son los que separan el aprobado del suspenso.

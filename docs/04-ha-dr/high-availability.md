# High Availability en AWS — SAA-C03

> Enfoque: **diseñar sistemas que sigan funcionando cuando algo falla**.

---

## 1) Qué es Alta Disponibilidad (HA)
Alta disponibilidad significa:
- El sistema **sigue funcionando**
- A pesar de fallos de:
  - instancias
  - hardware
  - una AZ completa

📌 Examen:
- HA ≠ Disaster Recovery
- HA = fallos **esperados** y frecuentes

---

## 2) Unidad básica de HA en AWS
👉 **Availability Zone (AZ)**

- Una AZ es independiente de otra
- Compartir región ≠ compartir fallo

📌 Regla de oro:
> Alta disponibilidad = **múltiples AZ**

---

## 3) Servicios clave para HA

### 3.1 Load Balancers (ELB)
- ALB / NLB distribuyen tráfico
- Health checks automáticos
- En múltiples AZ

📌 Examen:
- “Distribuir tráfico y detectar instancias caídas” → **ELB**

---

### 3.2 Auto Scaling Groups (ASG)
- Mantienen número deseado de instancias
- Reemplazan instancias fallidas
- Escalan automáticamente

📌 Examen:
- “Reemplazar instancias automáticamente” → **ASG**

---

### 3.3 Bases de datos

#### RDS Multi-AZ
- Standby en otra AZ
- Failover automático

#### Aurora
- Storage multi-AZ
- Failover rápido

📌 Examen:
- Relacional + HA → **Multi-AZ / Aurora**

---

## 4) HA por capas (patrón clásico)

### Capa web
- ALB + ASG en 2+ AZ

### Capa aplicación
- Stateless (ideal)
- Escalable horizontalmente

### Capa datos
- RDS Multi-AZ / Aurora
- DynamoDB (HA por diseño)

---

## 5) Servicios HA por diseño
- S3
- DynamoDB
- SQS
- SNS
- Lambda

📌 Examen:
- No requieren Multi-AZ manual

---

## 6) Errores comunes
- Una sola AZ → **NO HA**
- EC2 con backup ≠ HA
- HA no implica backup histórico

---

## 7) Frases típicas de examen
- “Alta disponibilidad dentro de una región” → Multi-AZ
- “Minimizar downtime” → ELB + ASG
- “Servicio gestionado y altamente disponible” → DynamoDB / S3 / Aurora

---

## 8) Mini-resumen
- HA = múltiples AZ
- ELB + ASG = base
- Bases de datos con replicación
- Stateless > stateful

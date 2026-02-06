# Amazon CloudWatch — Observabilidad, Monitorización y Alarmas (SAA-C03)

> CloudWatch es el **servicio central de observabilidad en AWS**.
> En el examen SAA-C03 aparece constantemente, muchas veces **de forma indirecta**,
> cuando el enunciado habla de:
>
> - “monitor resources”
> - “trigger alarms”
> - “react automatically to metrics”
> - “logs and troubleshooting”
>
> El objetivo de este documento es que **entiendas CloudWatch como sistema**,  
> sepas **qué problema resuelve cada componente**,  
> y **no lo confundas** con CloudTrail, Config u otros servicios.

---

## Mapa mental rápido (imprescindible)

| Necesidad del enunciado | CloudWatch |
|------------------------|-----------|
| Monitorizar métricas | ✅ |
| Crear alarmas automáticas | ✅ |
| Ver logs de aplicaciones | ✅ |
| Auditar quién hizo qué | ❌ (CloudTrail) |
| Ver compliance en el tiempo | ❌ (Config) |

📌 Regla de oro:
> **CloudWatch = lo que está pasando AHORA (o casi)**

---

# 1) Qué es CloudWatch (y qué NO es)

## 1.1 Qué es CloudWatch
Amazon CloudWatch es un servicio gestionado que permite:
- Recoger **métricas**
- Almacenar **logs**
- Crear **alarmas**
- Reaccionar automáticamente a eventos

👉 Piensa en CloudWatch como:
> *“el panel de control y alertas de AWS”*

---

## 1.2 Qué NO es CloudWatch
- ❌ No audita acciones API (eso es CloudTrail)
- ❌ No gestiona configuración/compliance (eso es Config)
- ❌ No es un sistema de logging externo generalista
- ❌ No sustituye sistemas APM avanzados (aunque cubre mucho)

---

# 2) CloudWatch Metrics

## 2.1 Qué son las métricas
Datos numéricos en el tiempo, por ejemplo:
- CPUUtilization
- NetworkIn / NetworkOut
- Latency
- RequestCount

Las métricas:
- se almacenan por namespace
- tienen dimensiones (InstanceId, LoadBalancer, etc.)

---

## 2.2 Métricas por defecto (muy de examen)

### EC2
- CPUUtilization
- NetworkIn / NetworkOut
- DiskReadOps / DiskWriteOps

📌 Trampa:
- **Memory y disk usage NO vienen por defecto**
- requieren CloudWatch Agent

---

### ALB / ELB
- RequestCount
- TargetResponseTime
- HTTPCode_ELB_5XX
- HealthyHostCount / UnHealthyHostCount

📌 En examen:
> “Detect unhealthy instances behind ALB” → CloudWatch metrics + alarms

---

### RDS
- CPUUtilization
- FreeStorageSpace
- DatabaseConnections

---

### Lambda
- Invocations
- Duration
- Errors
- Throttles

📌 En examen:
> “Detect Lambda throttling” → CloudWatch metrics

---

## 2.3 Custom Metrics
Puedes publicar métricas propias:
- desde aplicaciones
- desde scripts

📌 SAA-C03: basta con saber que existen y para qué.

---

# 3) CloudWatch Alarms (MUY IMPORTANTE)

## 3.1 Qué es una alarma
Una alarma evalúa una métrica y:
- cambia de estado (OK / ALARM / INSUFFICIENT_DATA)
- dispara una acción

---

## 3.2 Acciones típicas de una alarma

- Enviar notificación (SNS)
- Ejecutar acción automática:
  - scale ASG
  - stop/terminate EC2
  - invocar Lambda (vía EventBridge)

📌 En examen:
> “Automatically react when threshold is crossed” → **CloudWatch Alarm**

---

## 3.3 Ejemplos clásicos de examen

### Caso A
“Scale out EC2 instances when CPU exceeds 70%”

👉 **CloudWatch Alarm + Auto Scaling**

---

### Caso B
“Send notification when disk space is low”

👉 **CloudWatch Alarm + SNS**

---

### Caso C
“Trigger remediation when metric exceeds threshold”

👉 **CloudWatch Alarm (+ Lambda/EventBridge)**

---

# 4) CloudWatch Logs

## 4.1 Qué son los logs
CloudWatch Logs almacena:
- logs de aplicaciones
- logs de sistemas
- logs de servicios AWS

Ejemplos:
- logs de Lambda
- logs de EC2 (si los envías)
- logs de API Gateway

---

## 4.2 Log Groups y Log Streams
- **Log Group**: contenedor lógico (ej. `/aws/lambda/my-func`)
- **Log Stream**: secuencia concreta (instancia, ejecución, etc.)

---

## 4.3 Logs por servicio (muy de examen)

### Lambda
- Logs automáticos
- Cada ejecución escribe logs

📌 En examen:
> “Troubleshoot Lambda execution” → **CloudWatch Logs**

---

### API Gateway
- Access logs
- Execution logs

---

### EC2
- Requiere **CloudWatch Agent**
- Logs del SO o apps

📌 Trampa:
- logs de EC2 NO aparecen solos

---

## 4.4 Metric Filters
Puedes crear métricas a partir de logs:
- buscar patrón
- contar ocurrencias
- crear alarma

📌 En examen:
> “Trigger alarm based on log pattern” → **Metric Filter + Alarm**

---

# 5) CloudWatch Dashboards

## Qué son
- Visualización personalizada
- Métricas y alarmas juntas

📌 SAA-C03:
- Saber que existen
- No profundizan mucho

---

# 6) CloudWatch vs CloudTrail vs Config (comparativa CLAVE)

| Servicio | Qué responde |
|--------|-------------|
| CloudWatch | ¿Qué está pasando ahora? |
| CloudTrail | ¿Quién hizo qué acción API? |
| Config | ¿Es el recurso compliant en el tiempo? |

📌 Trampa clásica:
- No usar CloudWatch para auditoría

---

# 7) Casos de uso complejos (razonados)

## Caso 1 — Auto-remediación
**Problema**
- Detectar comportamiento anómalo y reaccionar

**Solución**
- CloudWatch Metric / Alarm
- EventBridge rule
- Lambda de remediación

---

## Caso 2 — Observabilidad completa de app web
```
ALB → EC2 / ECS / Lambda
```

- Métricas de ALB (latencia, 5XX)
- Logs de aplicación
- Alarmas de error rate

👉 CloudWatch como eje central.

---

## Caso 3 — Serverless monitoring
- Métricas Lambda (Errors, Duration)
- Logs automáticos
- Alarmas de throttling

---

# 8) Limitaciones y trade-offs

- Retención de logs cuesta dinero
- Muchas métricas personalizadas → coste
- No es un APM completo (aunque cubre mucho)

📌 En examen:
- casi nunca preguntan costes finos
- solo entender que “más datos = más coste”

---

# 9) Trampas típicas SAA-C03

- ❌ Usar CloudWatch para saber quién cambió un SG
- ❌ Asumir que EC2 logs aparecen automáticamente
- ❌ Confundir alarmas con métricas
- ❌ Pensar que CloudWatch bloquea tráfico

---

# 10) Tabla de decisiones rápidas

| Escenario | Solución |
|---------|---------|
| Monitorizar CPU | CloudWatch Metrics |
| Reaccionar a umbral | CloudWatch Alarm |
| Logs de Lambda | CloudWatch Logs |
| Logs de EC2 | CloudWatch Agent + Logs |
| Alarma por error en logs | Metric Filter |
| Escalar automáticamente | Alarm + ASG |

---

# 11) Labs recomendados (los que sí valen la pena)

## 🧪 Lab 1 — Alarm + Auto Scaling (IMPRESCINDIBLE)
1. ASG con EC2
2. Alarm por CPU
3. Ver scaling automático

👉 Te graba CloudWatch para siempre.

---

## 🧪 Lab 2 — Logs + Metric Filter
1. Lambda que escribe logs
2. Crear metric filter
3. Crear alarma

👉 Muy bueno para entender logs → métricas.

---

## 🧪 Lab 3 — EC2 logs (opcional)
1. Instalar CloudWatch Agent
2. Enviar logs de sistema
3. Verlos en CloudWatch

---

## Limpieza
- Borrar alarmas
- Borrar dashboards
- Revisar logs con retención corta
- Terminar recursos de labs

---

## Mini-resumen final

- CloudWatch = métricas + logs + alarmas
- Reacción automática a eventos
- No es auditoría (CloudTrail)
- No es compliance (Config)
- Aparece en MUCHAS preguntas indirectas

---

## Cierre
Si entiendes CloudWatch como **el sistema nervioso de AWS**,  
muchas preguntas de operación, escalado y seguridad se vuelven triviales.

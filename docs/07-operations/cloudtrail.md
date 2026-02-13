# AWS CloudTrail — Auditoría y Trazabilidad (SAA-C03)

> CloudTrail es el **servicio de auditoría de AWS**.
> En el examen SAA‑C03 aparece cuando el enunciado habla de:
>
> - “audit activity”
> - “track API calls”
> - “who did what”
> - “compliance and governance”
>
> CloudTrail **no sirve para monitorizar rendimiento**, sino para **saber quién hizo qué acción, cuándo y desde dónde**.

---

## 0) Mapa mental rápido

| Pregunta del enunciado | Servicio |
|------------------------|----------|
| ¿Quién hizo esta acción? | **CloudTrail** |
| ¿Cuándo se modificó un recurso? | **CloudTrail** |
| ¿Qué API se llamó? | **CloudTrail** |
| ¿Está el recurso sano ahora? | ❌ CloudWatch |
| ¿Cumple reglas en el tiempo? | ❌ Config |

📌 Regla de oro:
> **CloudTrail = auditoría (pasado)**  
> **CloudWatch = métricas/logs (presente)**

---

# 1) Qué es CloudTrail (y qué NO es)

## 1.1 Qué es CloudTrail
CloudTrail registra:
- **API calls** de AWS
- Acciones hechas desde:
  - consola
  - CLI
  - SDK
  - servicios AWS

Cada registro indica:
- quién hizo la acción
- desde qué IP/servicio
- cuándo
- con qué parámetros

👉 Piensa en CloudTrail como:
> *“el registro de actividad de la cuenta AWS”*

---

## 1.2 Qué NO es CloudTrail
- ❌ No es un sistema de métricas
- ❌ No es un sistema de alertas por sí solo
- ❌ No detecta amenazas (eso es GuardDuty)
- ❌ No gestiona configuración (eso es Config)

---

# 2) Qué registra CloudTrail

## 2.1 Management Events (CLAVE)
Acciones de control:
- CreateInstance
- DeleteBucket
- ModifySecurityGroup
- AttachRole
- etc.

📌 **Siempre son los más importantes en el examen**.

---

## 2.2 Data Events
Acciones sobre datos:
- GetObject / PutObject (S3)
- InvokeFunction (Lambda)

📌 Trampa:
- Data events **no siempre están activados por defecto**
- Cuestan más

En SAA-C03:
- basta con saber que existen y cuándo se usarían.

---

## 2.3 Insight Events (conceptual)
Detectan:
- comportamientos anómalos
- picos inusuales de actividad

📌 Sale poco, solo reconocer el nombre.

---

# 3) Trails y alcance

## 3.1 Trail
Un **trail** define:
- qué eventos se registran
- dónde se guardan (S3)
- si son multi-region

---

## 3.2 Multi-Region Trail (MUY de examen)
Un trail puede ser:
- regional
- **multi-region**

📌 En examen:
> *“Ensure API activity is logged across all regions”*  
→ **Multi-region trail**

---

## 3.3 Almacenamiento
- Los logs se guardan en **S3**
- Se pueden cifrar con **KMS**
- Se pueden retener largo tiempo

📌 CloudTrail **no borra logs automáticamente** (según retención de S3).

---

# 4) CloudTrail + otros servicios (arquitectura real)

## 4.1 CloudTrail + CloudWatch Logs
- Enviar eventos a CloudWatch Logs
- Crear métricas/alertas

📌 En examen:
> *“Trigger alert on specific API call”*  
→ **CloudTrail → CloudWatch Logs → Metric Filter → Alarm**

---

## 4.2 CloudTrail + GuardDuty
- GuardDuty analiza logs de CloudTrail
- Detecta actividad sospechosa

👉 CloudTrail **provee datos**, GuardDuty **detecta amenazas**.

---

## 4.3 CloudTrail + Config
- CloudTrail dice *quién cambió*
- Config dice *si el estado cumple reglas*

---

# 5) Casos típicos de examen

### Caso 1
“Find out who deleted an S3 bucket”

👉 **CloudTrail**

---

### Caso 2
“Track all API activity across the account and regions”

👉 **CloudTrail with multi-region trail**

---

### Caso 3
“Trigger an alert when someone changes a security group”

👉 **CloudTrail + CloudWatch Logs + Alarm**

---

### Caso 4
“Meet audit and compliance requirements”

👉 **CloudTrail (logs en S3, cifrados, retenidos)**

---

# 6) Trampas típicas SAA-C03

- ❌ Usar CloudWatch para saber quién hizo algo
- ❌ Pensar que CloudTrail bloquea acciones
- ❌ Olvidar activar multi-region trail
- ❌ Confundir CloudTrail con Config

---

# 7) Tabla comparativa clave

| Servicio | Rol principal |
|--------|---------------|
| CloudTrail | Auditoría de acciones |
| CloudWatch | Métricas y logs |
| Config | Compliance y estado |
| GuardDuty | Detección de amenazas |

---

# 8) Buenas prácticas (en clave de examen)

- Activar **CloudTrail en todas las cuentas**
- Usar **multi-region trail**
- Guardar logs en S3 centralizado
- Cifrar logs con KMS
- Integrar con CloudWatch para alertas

---

# 9) Labs recomendados (los que sí valen la pena)

## 🧪 Lab 1 — CloudTrail básico (IMPRESCINDIBLE)
1. Activar trail multi-region
2. Ejecutar acciones (crear SG, lanzar EC2)
3. Ver eventos en CloudTrail

👉 Te graba el concepto para siempre.

---

## 🧪 Lab 2 — CloudTrail + CloudWatch Alarm
1. Enviar CloudTrail a CloudWatch Logs
2. Crear metric filter para acción sensible
3. Crear alarma

👉 Muy bueno para entender auditoría + reacción.

---

## Limpieza
- Borrar trails de prueba (si no los quieres)
- Revisar buckets S3 creados
- Borrar métricas/alarms de labs

---

## 10) Mini-resumen final

- CloudTrail = quién hizo qué
- Registra API calls
- Multi-region trail = cobertura total
- Logs en S3
- Base para auditoría y compliance

---

## Cierre
Si entiendes CloudTrail como **la caja negra de AWS**,  
las preguntas de auditoría y seguridad se vuelven obvias.

## Laboratorio
👉 [Operations — Laboratorio](operations-lab.md)
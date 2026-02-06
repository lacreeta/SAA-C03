# AWS Config, GuardDuty y Macie — Seguridad y Compliance (SAA-C03)

> Este documento cubre **tres servicios clave de seguridad “pasiva” en AWS**:
> no bloquean tráfico ni ejecutan lógica, sino que **observan, detectan y evalúan**.
>
> En el examen SAA-C03 aparecen mucho en preguntas del tipo:
> - *“detect suspicious activity”*
> - *“evaluate compliance over time”*
> - *“identify sensitive data”*
>
> El objetivo aquí es que entiendas **qué hace cada uno**, **cuándo elegirlo**,  
> y **por qué NO sirven para otras cosas**.

---

## Mapa mental rápido (antes de entrar al detalle)

| Necesidad del enunciado | Servicio |
|------------------------|----------|
| Ver si los recursos cumplen reglas de seguridad en el tiempo | **AWS Config** |
| Detectar actividad sospechosa / amenazas | **GuardDuty** |
| Descubrir datos sensibles (PII) en S3 | **Macie** |

---

# 1) AWS Config — Compliance y estado en el tiempo

## 1.1 Qué es AWS Config
AWS Config es un servicio que:
- **Registra el estado de los recursos AWS**
- Mantiene un **histórico de cambios**
- Evalúa si los recursos **cumplen reglas de compliance**

👉 Piensa en Config como:
> *“una cámara de vigilancia del estado de mi infraestructura”*

---

## 1.2 Qué problemas resuelve

### Problemas típicos
- ¿Quién cambió esta security group?
- ¿Este bucket S3 ha sido público alguna vez?
- ¿Mis recursos cumplen las políticas corporativas?

### Lo que Config **SÍ hace**
- Detecta **drift**
- Evalúa **compliance a lo largo del tiempo**
- Permite auditorías

### Lo que Config **NO hace**
- ❌ No bloquea cambios
- ❌ No detecta ataques
- ❌ No analiza tráfico

---

## 1.3 Componentes principales

### Configuration Items
- “Snapshot” del estado de un recurso
- Incluye:
  - propiedades
  - relaciones
  - cambios en el tiempo

---

### Rules
Reglas que definen qué es “correcto”.

Ejemplos:
- “S3 buckets no deben ser públicos”
- “EBS volumes deben estar cifrados”
- “Security Groups no deben permitir 0.0.0.0/0 en SSH”

Las reglas pueden ser:
- **Managed** (AWS las proporciona)
- **Custom** (Lambda, no entra en SAA-C03)

---

### Compliance Status
Cada recurso queda marcado como:
- **COMPLIANT**
- **NON-COMPLIANT**

Esto es lo que usan los auditores.

---

## 1.4 Casos de uso típicos (examen)

📌 Enunciados tipo:
- “Evaluate compliance over time”
- “Track configuration changes”
- “Audit security posture”

👉 **Respuesta**: AWS Config

---

## 1.5 Config vs CloudTrail (trampa clásica)

| Config | CloudTrail |
|------|------------|
| Estado del recurso | Acciones/API calls |
| Histórico de configuración | Histórico de acciones |
| Compliance | Auditoría de quién hizo qué |

📌 En examen:
- *“Who changed it?”* → **CloudTrail**
- *“Is it compliant?”* → **Config**

---

# 2) Amazon GuardDuty — Detección de amenazas

## 2.1 Qué es GuardDuty
Amazon GuardDuty es un servicio de **detección de amenazas gestionado** que:
- Analiza logs
- Usa ML + threat intelligence
- Genera **findings** de seguridad

👉 Piensa en GuardDuty como:
> *“el sistema de alarmas de AWS”*

---

## 2.2 Qué datos analiza (muy importante)

GuardDuty analiza:
- **CloudTrail logs** (management events)
- **VPC Flow Logs**
- **DNS logs**

⚠️ No necesitas habilitar manualmente esos logs para GuardDuty (detalle fino de examen).

---

## 2.3 Qué detecta GuardDuty

Ejemplos:
- Uso de credenciales comprometidas
- Acceso desde ubicaciones sospechosas
- Escaneo de puertos
- Comunicación con IPs maliciosas

📌 Importante:
- GuardDuty **detecta**, no bloquea.

---

## 2.4 Qué GuardDuty NO es

- ❌ No es un firewall
- ❌ No bloquea tráfico
- ❌ No corrige automáticamente

(La respuesta suele ser “detectar”, no “prevenir”).

---

## 2.5 Casos de uso típicos (examen)

📌 Enunciados tipo:
- “Detect suspicious activity”
- “Threat detection”
- “Identify compromised credentials”

👉 **Respuesta**: GuardDuty

---

## 2.6 GuardDuty + otros servicios
GuardDuty suele combinarse con:
- CloudWatch (alarmas)
- EventBridge (automatizar respuesta)
- Lambda (remediación automática)

Pero en SAA-C03 basta con **reconocer GuardDuty**.

---

# 3) Amazon Macie — Datos sensibles en S3

## 3.1 Qué es Macie
Amazon Macie es un servicio que:
- **Analiza buckets S3**
- Usa ML para detectar:
  - PII
  - datos sensibles
  - datos regulados

👉 Piensa en Macie como:
> *“el detector de secretos olvidados en S3”*

---

## 3.2 Qué tipo de datos detecta

Ejemplos:
- Números de tarjetas de crédito
- DNI / SSN
- Emails
- Datos personales

📌 Muy orientado a **compliance y privacidad**.

---

## 3.3 Qué Macie NO hace

- ❌ No cifra datos
- ❌ No bloquea accesos
- ❌ No analiza tráfico
- ❌ No aplica políticas

Macie **solo identifica y clasifica**.

---

## 3.4 Casos de uso típicos (examen)

📌 Enunciados tipo:
- “Identify sensitive data in S3”
- “Discover PII”
- “Data privacy compliance”

👉 **Respuesta**: Macie

---

## 3.5 Macie vs KMS (trampa)

| KMS | Macie |
|----|------|
| Cifra datos | Detecta datos sensibles |
| Protección | Descubrimiento |
| En reposo | Análisis de contenido |

📌 Si el enunciado dice:
- *“encrypt data”* → **KMS**
- *“identify sensitive data”* → **Macie**

---

# 4) Comparativa final (muy de examen)

| Servicio | ¿Qué hace? | ¿Qué NO hace? | Palabras clave |
|-------|------------|---------------|----------------|
| Config | Compliance, estado, drift | No detecta ataques | compliance, audit, rules |
| GuardDuty | Detecta amenazas | No bloquea | suspicious activity |
| Macie | Detecta PII en S3 | No cifra ni bloquea | sensitive data |

---

# 5) Trampas típicas del examen

- ❌ Pensar que GuardDuty bloquea ataques → NO
- ❌ Pensar que Config detecta amenazas → NO
- ❌ Pensar que Macie cifra datos → NO
- ❌ Confundir Config con CloudTrail

---

# 6) Mini-resumen para memorizar

- **AWS Config** → compliance y cambios en el tiempo  
- **GuardDuty** → detección de amenazas  
- **Macie** → datos sensibles en S3  

Si el enunciado habla de:
- *“compliance”* → Config
- *“suspicious activity”* → GuardDuty
- *“PII in S3”* → Macie

---

## Cierre
Estos tres servicios **no sustituyen firewalls, IAM ni cifrado**.  
Son la capa de **visibilidad y detección**, fundamental en arquitecturas seguras.

Con este fichero ya cubres **100% de lo que SAA-C03 espera** sobre Config, GuardDuty y Macie.

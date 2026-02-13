# Serverless (SAA-C03) — Guía “de examen” ultra práctica (decisiones, porqués y trampas)

> Este documento NO es “qué es cada servicio” (eso ya lo tienes en tus apuntes).
> Esto es lo que te hace **marcar la respuesta correcta** en SAA‑C03:  
> **cuándo elegir qué**, **por qué**, y **qué trampas evitar**.
>
> Servicios cubiertos:
> - Lambda
> - API Gateway (HTTP API / REST API / WebSocket)
> - SQS (Standard / FIFO, DLQ, redrive)
> - SNS (fan-out)
> - EventBridge (event bus)
> - Step Functions (orquestación)

---

## 0) Mapa mental general (la foto que resuelve muchas preguntas)

### Serverless “clásico” (requests HTTP)
```
Cliente → API Gateway → Lambda → (DynamoDB/S3/etc.)
```

### Asíncrono / desacoplado (buffer)
```
Productor → SQS → Consumidores (Lambda/ECS/EC2)
```

### Fan-out (un mensaje a muchos)
```
Productor → SNS Topic → (SQS) → múltiples consumidores
```

### Event-driven (eventos AWS / eventos de negocio)
```
Evento → EventBridge Rule → Target (Lambda/SQS/SNS/Step Functions)
```

### Workflows (pasos, decisiones, retries “bonitos”)
```
Step Functions → (Lambda / AWS services) → estados + control
```

---

## 1) “Si el enunciado dice X → elige Y” (reglas rápidas)

### Exponer una API HTTP “sin servidores”
- **API Gateway + Lambda**
  - Si no piden features raras → **HTTP API**
  - Si piden features avanzadas → **REST API**
- Por qué:
  - mínimo overhead, escala automático, pago por uso

### Procesar tareas en background / desacoplar y absorber picos
- **SQS + consumidor**
  - consumidor típico: **Lambda** (si encaja en 15 min) o ECS/EC2 si es pesado
- Por qué:
  - SQS almacena, reintenta, desacopla, protege downstream

### Notificar a múltiples sistemas (pub/sub)
- **SNS**
  - y si quieres “cada consumidor a su ritmo” → **SNS → SQS (fan-out con colas)**
- Por qué:
  - SNS “empuja” a muchos; SQS garantiza buffering por consumidor

### Reaccionar a “eventos” del ecosistema AWS (S3, EC2, CloudTrail…) o eventos de negocio con filtros
- **EventBridge**
- Por qué:
  - routing por patrón JSON, integración nativa, desacoplamiento por eventos

### Orquestar múltiples pasos con control de errores, waits, paralelismo, decisiones
- **Step Functions**
- Por qué:
  - visibilidad y control del flujo sin meterlo todo en una Lambda gigante

### Requisitos de orden estricto o “exactly once”
- **SQS FIFO** (cuando el enunciado explícitamente pide orden/duplicados controlados)
- Por qué:
  - Standard = at-least-once + posible desorden; FIFO = orden + dedup

---

## 2) Lambda: cómo pensarla en el examen

### Modelo mental
Lambda = **función stateless** que se ejecuta:
- por eventos (S3, EventBridge, SQS, DynamoDB Streams…)
- o por requests (API Gateway)

### Límites/ideas que te ayudan a descartar
- **Máx 15 minutos** → si el enunciado habla de procesos largos (ETL pesado, horas), NO Lambda
- Stateless → si necesita estado persistente, guarda en DynamoDB/S3/RDS/etc.
- Concurrencia y throttling existen (concepto). Si piden absorber picos, combina con SQS.

### Patrones típicos que caen
- **API Gateway → Lambda** (backend serverless)
- **S3 event → Lambda** (procesar objetos)
- **SQS → Lambda** (workers serverless)
- **EventBridge → Lambda** (reaccionar a eventos)

### Trampas comunes
- “Lambda para proceso de 2 horas” → NO
- “Lambda como cola” → NO (eso es SQS)
- “Lambda stateful” → NO (usa DB/caché externa)

### Por qué Lambda suele ser la respuesta
- mínimo ops (no ASG, no patching)
- escalado automático
- pago por uso

---

## 3) API Gateway (HTTP API vs REST API vs WebSocket)

### 3.1 HTTP API (la opción “por defecto” en el examen)
**Cuándo elegirla**
- API simple
- menor coste/latencia
- integración directa con Lambda

**Frase de examen**
> Si no piden nada especial → **HTTP API**.

### 3.2 REST API (cuando el enunciado pide “features avanzadas”)
**Señales típicas en el enunciado**
- autorización/validación avanzada
- transformaciones de request/response
- features “enterprise” (enunciado muy cargado)

**Por qué**
- REST API tiene más capacidades, a costa de más complejidad/coste

### 3.3 WebSocket API (tiempo real)
**Señales típicas**
- chat
- dashboards en tiempo real
- bidireccional

### Trampas típicas
- “API Gateway es un Load Balancer” → NO (es gateway de APIs)
- Para “balancear” compute tradicional: ALB/NLB; para “exponer endpoints serverless”: API Gateway.

---

## 4) SQS: cola, buffering, reintentos y DLQ (esto cae muchísimo)

### 4.1 Standard vs FIFO (decisión clásica)
#### Standard
- Muy alta escala
- **At-least-once** (puede duplicar)
- **No garantiza orden**

**El examen la usa cuando**
- quieres throughput
- puedes tolerar reorden/duplicados (o haces idempotencia)

#### FIFO
- **Orden garantizado**
- Deduplicación (exactly-once processing “práctico” bajo ciertas condiciones)
- menor throughput

**El examen la usa cuando**
- “orden estricto”
- “no puedo procesar duplicados” (y/o deduplication)

> Regla: si NO mencionan orden/duplicados estrictos, casi siempre Standard.

### 4.2 Visibility Timeout (trampa típica)
Si un consumidor lee un mensaje, SQS lo “esconde” un tiempo.
- Si el consumidor no confirma (delete) antes de acabar, el mensaje reaparece.
**Señal de examen**
- “se procesa dos veces” → ajustar visibility timeout o hacer idempotencia

### 4.3 DLQ (Dead-Letter Queue)
**Cuándo**
- quieres “sacar” mensajes que fallan repetidamente
- evitar bucles infinitos de retries

**En examen**
> “Mensajes fallidos deben ser analizados sin bloquear el sistema” → **DLQ**

### 4.4 Fan-out robusto (patrón de oro)
SNS por sí solo empuja, pero no “bufferiza” por consumidor.
Para fan-out **fiable**:
```
SNS Topic → múltiples SQS → cada consumer procesa a su ritmo
```

---

## 5) SNS: pub/sub “push” y fan-out

### Cuándo SNS es la respuesta
- “notificar” a múltiples subscriptores
- “un evento debe activar varios sistemas”
- “fan-out” explícito

### Por qué SNS + SQS es tan común
- SNS distribuye
- SQS “aguanta” y desacopla cada consumer
- si un consumer cae, su cola retiene mensajes y no afecta al resto

### Trampas típicas
- “SNS retiene mensajes como cola” → NO (para retención/buffering usa SQS)
- “SNS garantiza orden” → NO (si necesitas orden, piensa en FIFO / otras estrategias)

---

## 6) EventBridge: el “router de eventos” (cuando el enunciado dice “event-driven”)

### Cuándo EventBridge es la respuesta
- “cuando ocurra X en AWS, ejecuta Y”
- “evento de negocio” publicado en un bus
- “múltiples reglas” con filtros por contenido del evento

### Por qué EventBridge es diferente a SNS
- SNS es “pub/sub simple”
- EventBridge es “event bus” con **routing por patrón** (JSON)
- integración natural con muchos servicios AWS

### Trampas típicas
- “EventBridge es una cola” → NO
- “EventBridge garantiza orden” → NO (en general no es su promesa principal)

### Señales típicas del enunciado
- “microservices loosely coupled”
- “react to events”
- “decouple producers and consumers”
- “audit / automation based on AWS service events”

---

## 7) Step Functions: orquestación (no ejecutes todo en una Lambda)

### Cuándo Step Functions es la respuesta
- workflow con pasos
- decisiones (if/else)
- reintentos y backoff
- paralelismo
- esperas (wait)
- visibilidad del estado de cada paso

### Por qué es mejor que “una Lambda enorme”
- Observabilidad del flujo
- Retries controlados por estado
- Menos lógica “pegada” en código
- Reduce complejidad operativa y debugging

### Step Functions vs SQS (diferencia que cae)
- **SQS**: buffer de mensajes para consumidores (desacoplar y absorber picos)
- **Step Functions**: coordina pasos y dependencias (control de proceso)

---

## 8) Tabla de decisión (lo que te salva en preguntas mixtas)

| Necesidad del enunciado | Servicio(s) | Por qué |
|---|---|---|
| Endpoint HTTP serverless | API Gateway + Lambda | Expones lógica sin servidores |
| API simple y barata | HTTP API | Menor coste/latencia, suficiente |
| API con features avanzadas | REST API | Más capacidades (validación/transformaciones/autorización avanzada) |
| Procesar asíncrono y desacoplar | SQS + Consumer | Buffer, reintentos, protege downstream |
| Orden estricto / dedup | SQS FIFO | Garantiza orden + deduplicación |
| Fan-out a muchos consumidores | SNS (+ SQS por consumer) | Pub/Sub + buffering por consumidor |
| Eventos AWS (S3/EC2/etc.) → acción | EventBridge | Event bus + reglas por patrón |
| Workflow con pasos, retries, decisiones | Step Functions | Orquesta, controla errores, visibilidad |
| Minimizar ops | “Serverless stack” | Menos servidores y gestión |

---

## 9) Preguntas tipo examen (y cómo razonar)

### Caso A
“Necesitan exponer un endpoint HTTP para ejecutar lógica sin administrar servidores. Quieren bajo coste y baja latencia.”
→ **API Gateway HTTP API + Lambda**

**Por qué:** HTTP API es el “default” si no piden features de REST API.

### Caso B
“Procesamiento de pedidos asíncrono para no sobrecargar el backend. Debe absorber picos.”
→ **SQS + consumidores**

**Por qué:** buffering y desacoplamiento.

### Caso C
“Un evento debe activar 3 sistemas distintos, y si uno está caído no debe perder mensajes.”
→ **SNS → 3 colas SQS → consumidores**

**Por qué:** fan-out + retención por consumer.

### Caso D
“Cuando ocurra X en AWS, lanzar un flujo que ejecute varios pasos con retries y decisiones.”
→ **EventBridge Rule → Step Functions**

**Por qué:** EventBridge dispara, Step Functions orquesta.

### Caso E
“Necesitan garantizar orden estricto de procesamiento.”
→ **SQS FIFO**

**Por qué:** Standard no garantiza orden.

---

## 10) Trampas rápidas (lista de verificación mental)

- API Gateway **no** es un Load Balancer.
- Route 53 failover = DNS (DR), **no** HA instantánea.
- SNS **no** es una cola (no es para retención/buffering).
- EventBridge **no** es una cola (es routing de eventos).
- Lambda **no** es para procesos largos/stateful.
- SQS Standard puede duplicar y desordenar (idempotencia).
- Step Functions orquesta; Lambda ejecuta.

---

## 11) “Lo mínimo que debes memorizar” (si tuvieras 10 minutos)

1. HTTP endpoint serverless → **API Gateway + Lambda**  
2. Asíncrono/buffer → **SQS**  
3. Fan-out → **SNS** (y para robustez: SNS → SQS)  
4. Eventos AWS → **EventBridge**  
5. Workflows → **Step Functions**  
6. Orden estricto → **SQS FIFO**  
7. Lambda máx 15 min, stateless


## Laboratorio
👉 [Serverless — Laboratorio](serverless-lab.md)
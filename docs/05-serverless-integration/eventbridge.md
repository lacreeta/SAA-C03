# EventBridge — SAA-C03 (DETALLADO)

## 1. Qué es EventBridge
Amazon EventBridge es un **event bus** totalmente gestionado que permite **conectar servicios mediante eventos** sin acoplamiento directo.

👉 Arquitectura **event-driven**.

---

## 2. Concepto clave
Un evento es algo que **ya ocurrió**.

Ejemplos:
- Se creó una instancia
- Se subió un objeto a S3
- Se completó una orden

EventBridge:
- Escucha eventos
- Aplica reglas
- Lanza acciones

---

## 3. Componentes

### 3.1 Event Bus
- Default bus (servicios AWS)
- Custom bus (eventos propios)

---

### 3.2 Rules
- Filtran eventos por patrón (JSON)
- Deciden qué evento interesa

---

### 3.3 Targets
- Lambda
- SQS
- SNS
- Step Functions

---

## 4. Cuándo usar EventBridge
- Integración entre servicios
- Arquitecturas desacopladas
- Reaccionar a eventos

📌 Examen:
> “Cuando ocurra X, ejecuta Y” → **EventBridge**

---

## 5. EventBridge vs SNS vs SQS
- EventBridge: routing inteligente por eventos
- SNS: notificaciones push
- SQS: cola y buffer

---

## 6. Casos típicos
- Auditoría
- Automatización
- Microservicios

---

## 7. Trampas
- No es una cola
- No garantiza orden

---

## 8. Mini-resumen
- Event bus central
- Event-driven architecture

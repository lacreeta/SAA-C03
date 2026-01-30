# SNS — SAA-C03 (DETALLADO)

## 1. Qué es SNS
Amazon SNS es un servicio de **publicación/suscripción (pub/sub)**.

Un productor envía un mensaje a un **topic**, y SNS lo distribuye a todos los suscriptores.

---

## 2. Modelo
- Push-based
- Fan-out

---

## 3. Suscriptores
- Lambda
- SQS
- Email / SMS
- HTTP endpoints

---

## 4. Casos de uso
- Notificaciones
- Fan-out a múltiples sistemas
- Eventos simples

---

## 5. SNS vs SQS
- SNS empuja mensajes
- SQS almacena mensajes

📌 Examen:
- “Notificar a muchos” → SNS
- “Procesar asíncrono” → SQS

---

## 6. Trampas
- SNS no retiene mensajes
- No es una cola

---

## 7. Mini-resumen
- Pub/Sub
- Push

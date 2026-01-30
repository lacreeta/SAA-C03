# Lambda — SAA-C03 (DETALLADO)

## 1. Qué es Lambda
AWS Lambda es un servicio de **compute serverless** que ejecuta código **en respuesta a eventos**.

No gestionas:
- Servidores
- SO
- Scaling

Pagas solo:
- Por ejecución
- Por tiempo

---

## 2. Modelo mental
Lambda = función:
- Stateless
- De corta duración
- Escala horizontalmente

---

## 3. Triggers comunes
- API Gateway
- S3
- DynamoDB Streams
- EventBridge
- SQS

---

## 4. Límites importantes (examen)
- Tiempo máximo de ejecución
- Memoria configurable
- /tmp limitado
- Cold starts (concepto)

📌 Examen:
- “Proceso largo” → NO Lambda

---

## 5. Casos de uso
- APIs serverless
- Procesamiento de eventos
- Automatización

---

## 6. Ventajas
- Sin servidores
- Escala automática
- Alta disponibilidad

---

## 7. Trampas
- No es para workloads largos
- No es stateful

---

## 8. Mini-resumen
- Compute bajo demanda
- Event-driven

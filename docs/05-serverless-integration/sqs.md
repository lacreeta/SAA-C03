# SQS — SAA-C03 (DETALLADO)

## 1. Qué es SQS
Amazon SQS es un servicio de **colas gestionadas** para desacoplar sistemas.

---

## 2. Modelo
- Pull-based
- Los consumidores leen mensajes

---

## 3. Tipos de cola

### Standard
- Alta escalabilidad
- Al menos una vez
- No garantiza orden

### FIFO
- Orden garantizado
- Exactly-once processing
- Menor throughput

📌 Examen:
- “Orden estricto” → FIFO

---

## 4. Casos de uso
- Buffering
- Procesamiento asíncrono
- Protección de sistemas downstream

---

## 5. SQS vs SNS
- SQS = cola
- SNS = notificación

---

## 6. Trampas
- SQS no ejecuta lógica
- Necesita consumidores

---

## 7. Mini-resumen
- Desacoplamiento
- Buffer

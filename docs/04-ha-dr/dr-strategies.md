# Disaster Recovery (DR) Strategies — SAA-C03

> Enfoque: **qué hacer cuando ocurre una catástrofe** (región caída, datos corruptos, etc.).

---

## 1) HA vs DR (clave del examen)
- **HA**: fallos normales (instancia/AZ)
- **DR**: fallos graves (región, data loss)

📌 Examen:
- Muchas preguntas distinguen claramente esto.

---

## 2) Métricas clave de DR

### RPO (Recovery Point Objective)
- Cuánta pérdida de datos es aceptable

### RTO (Recovery Time Objective)
- Cuánto tiempo puedo estar caído

📌 Menor RPO/RTO = mayor coste

---

## 3) Estrategias de DR (ordenadas por coste)

### 3.1 Backup & Restore
- Backups en S3
- Infra se recrea desde cero

✅ Barato  
❌ RTO/RPO altos

📌 Examen:
- “Coste mínimo, recuperación lenta” → **Backup & Restore**

---

### 3.2 Pilot Light
- Infra mínima activa
- Datos replicados
- Escala bajo demanda

📌 Equilibrio coste/tiempo

---

### 3.3 Warm Standby
- Sistema reducido siempre activo
- Escalado rápido

📌 Más caro, menor RTO

---

### 3.4 Multi-site / Active-Active
- Dos regiones activas
- Tráfico distribuido

📌 RTO/RPO mínimos  
❌ Muy caro

---

## 4) Servicios usados en DR
- Route 53 (failover routing)
- S3 cross-region replication
- DynamoDB Global Tables
- Aurora Global Database
- Snapshots cross-region

---

## 5) Route 53 en DR
- Failover routing
- Health checks
- Cambio de tráfico entre regiones

📌 Examen:
- “Redirigir tráfico a otra región automáticamente” → Route 53 Failover

---

## 6) Trampas típicas
- Multi-AZ ≠ DR multi-región
- Backup ≠ HA
- DR siempre implica planificación previa

---

## 7) Mini-resumen
- DR = región caída
- RPO/RTO definen la estrategia
- Backup & Restore (barato)
- Active-Active (caro, rápido)

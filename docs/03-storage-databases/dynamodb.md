# DynamoDB — SAA-C03

> DynamoDB es la opción **NoSQL gestionada** de AWS, optimizada para **latencia baja**, **escala masiva** y **operación mínima**.

---

## 1) Qué es DynamoDB
**Amazon DynamoDB** es una base de datos **NoSQL key-value / document**:
- Totalmente gestionada
- Escala automática (según modo)
- Latencia de milisegundos
- Alta disponibilidad por diseño

📌 Examen: cuando el enunciado pide “serverless”, “baja latencia”, “alto tráfico” o “escala extrema”, suelen querer DynamoDB.

---

## 2) Modelo mental (clave de examen)

### 2.1 Tabla, items y atributos
- **Tabla**: colección
- **Item**: “fila” (documento)
- **Atributos**: campos

### 2.2 Claves (lo más preguntado)
#### Partition key (PK)
- Determina **distribución** y escalado
- Debe tener buena cardinalidad (evitar “hot partitions”)

#### Sort key (SK) (opcional)
- Permite ordenar/consultar rangos dentro de una misma PK

📌 Patrón típico:
- PK = `USER#123`
- SK = `ORDER#2026-01-30T12:00:00Z`

---

## 3) Consistencia: eventual vs strong
- **Eventually consistent reads** (por defecto): más eficientes
- **Strongly consistent reads**: consistencia fuerte (cuando aplica)

📌 Examen:
- “Necesito lecturas inmediatamente consistentes” → **strongly consistent** (si la pregunta lo permite)

---

## 4) Capacity modes: On-Demand vs Provisioned

### 4.1 On-Demand
- Pagas por request
- Ideal para:
  - tráfico impredecible
  - picos
  - sistemas nuevos

### 4.2 Provisioned
- Defines RCU/WCU
- Ideal para:
  - tráfico estable
  - coste optimizado

📌 Examen:
- “Tráfico impredecible” → **On-Demand**
- “Tráfico estable, quiero ahorrar” → **Provisioned**

---

## 5) Acelerar lecturas: DAX
**DynamoDB Accelerator (DAX)** es un **cache en memoria** para DynamoDB:
- Reduce latencia de lecturas (muy rápido)
- Útil cuando:
  - muchas lecturas repetidas
  - quieres menos carga en DynamoDB

📌 Examen:
- “Necesito lecturas ultra rápidas sin cambiar demasiado la app” → **DAX**

⚠️ Trampa:
- DAX es para **lecturas**; no garantiza consistencia fuerte.

---

## 6) Streams y eventos
**DynamoDB Streams** captura cambios (insert/update/delete) y permite:
- disparar Lambda
- integraciones event-driven
- auditoría

📌 Examen:
- “Cada vez que cambie un item, dispara un proceso serverless” → **Streams + Lambda**

---

## 7) Global Tables (multi-región)
Para apps globales con baja latencia:
- Replicación multi-región
- Acceso local por región

📌 Examen:
- “Usuarios globales, baja latencia, NoSQL” → **Global Tables**

---

## 8) Backups y recuperación
- Backups on-demand (snapshots)
- **PITR** si está habilitado

---

## 9) Seguridad
- IAM para control de acceso
- KMS para cifrado en reposo
- VPC endpoints para acceso privado (sin internet)

📌 Examen:
- “Acceso privado a DynamoDB desde VPC sin internet” → **Gateway VPC Endpoint** (concepto)

---

## 10) DynamoDB vs RDS/Aurora (decisión)
### DynamoDB
- Key-value/document
- escala enorme
- latencia baja
- serverless

### RDS/Aurora
- SQL
- joins
- constraints e integridad referencial
- transacciones complejas

📌 Frase típica:
- “Necesito joins y SQL complejas” → **RDS/Aurora**
- “Necesito escala masiva y latencia baja con clave-valor” → **DynamoDB**

---

## 11) Trampas típicas
- PK mal elegida → hot partition / throttling
- “SQL” → no es DynamoDB
- DAX ≠ ElastiCache

---

## 12) Mini-resumen
- DynamoDB = NoSQL gestionada
- PK/SK son el núcleo
- On-demand para picos
- Provisioned para estable
- DAX acelera lecturas
- Streams para eventos
- Global Tables para multi-región

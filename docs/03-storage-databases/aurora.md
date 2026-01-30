# Aurora — SAA-C03

> Aurora es el motor relacional cloud-native de AWS (MySQL/PostgreSQL compatible) diseñado para **alta disponibilidad** y **escala** con menos trabajo operativo.

---

## 1) Qué es Aurora
**Amazon Aurora** es parte de RDS, pero con arquitectura propia:
- Compatible con **MySQL** o **PostgreSQL** (a nivel de API/driver)
- Pensada para **alto rendimiento** + **HA**
- Se organiza en **clusters** (writer + readers)

📌 Examen: Aurora suele aparecer cuando el enunciado dice “necesito relacional + HA + escalabilidad + rendimiento”, y quieren que elijas Aurora en lugar de “RDS estándar”.

---

## 2) Arquitectura mental (examen)

### 2.1 Cluster de Aurora
Un **Aurora cluster** tiene:
- 1 **Writer (primary)**
- 0..N **Readers (Aurora Replicas)**
- **Cluster endpoint** (para la app)
- **Writer endpoint** y **Reader endpoint** (para separar escrituras/lecturas)

### 2.2 Compute vs storage desacoplados
Aurora desacopla **compute** y **storage**:
- El storage es **compartido** y replicado automáticamente en múltiples AZ.
- Los nodos de compute (writer/readers) se apoyan en ese storage distribuido.

✅ Implicación: failover y HA suelen ser mejores/rápidos que RDS tradicional.

---

## 3) Replicas: Aurora Replicas vs RDS Read Replicas
En Aurora:
- Los **readers** forman parte del cluster.
- Puedes tener múltiples readers y **escalar lecturas**.
- El failover suele ser más directo (promoción de reader a writer).

⚠️ Trampa de examen:
- En RDS Multi-AZ, el standby no se usa para lecturas.
- En Aurora, los readers sí se usan para lecturas.

---

## 4) Alta disponibilidad y failover
Aurora está pensada para:
- Alta durabilidad (storage multi-AZ)
- Failover automático: promoción de un reader a writer

📌 Examen:
- “Relacional con HA muy fuerte” → **Aurora**
- “Necesito escalar lecturas con múltiples réplicas en un cluster” → **Aurora**

---

## 5) Aurora Serverless (cuándo encaja)
Aurora Serverless (especialmente v2 en diseños modernos) encaja cuando:
- Carga **intermitente** o con picos impredecibles
- Quieres capacidad que sube/baja automáticamente
- No quieres gestionar dimensionamiento

✅ Caso típico:
- dev/test
- apps con picos

⚠️ No ideal:
- cargas constantes y previsibles (normalmente sale más eficiente provisionado)

---

## 6) Multi-región (concepto)
Aurora soporta opciones para arquitecturas globales (según modo y necesidad):
- Replicación a regiones
- Lecturas más cercanas al usuario

📌 Enunciado típico:
> “Usuarios globales, lecturas rápidas en varias regiones, relacional”
➡️ **Aurora** (si además piden DR global/lecturas globales, Aurora suele ser candidata)

---

## 7) Seguridad
Igual que RDS:
- Dentro de VPC
- SG controla acceso
- KMS para cifrado en reposo
- TLS en tránsito

---

## 8) Cuándo elegir Aurora (regla práctica)
Elige **Aurora** cuando necesites:
- SQL relacional
- Alto rendimiento
- HA fuerte
- Escala de lecturas con múltiples réplicas
- Opciones más cloud-native (cluster, endpoints)

Elige **RDS estándar** cuando:
- Requisitos “normales”
- Coste menor
- Motor específico (Oracle/SQL Server) que no está en Aurora

---

## 9) Trampas típicas
- Aurora no es NoSQL (para eso DynamoDB).
- Aurora no es Redshift (analítica).
- Si el enunciado pide “escala masiva sin gestionar” y no necesita SQL → probablemente DynamoDB.

---

## 10) Mini-resumen
- Aurora = cluster (writer + readers)
- Storage distribuido multi-AZ
- Reader endpoint para lecturas
- Failover promoviendo reader
- Serverless si carga intermitente

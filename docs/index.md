# AWS SAA-C03 — Apuntes

Apuntes orientados a **decisiones de arquitectura**, no a definiciones.

## Cómo estudiar
- Problema → Servicio → Por qué → Trade-offs
- Enfocado al examen SAA-C03
- Ejemplos y trampas frecuentes

## Tablas clave de decisión (examen)

### Storage & Databases
| Servicio | Tipo | ¿Compartido? | Alcance | Uso típico (examen) | Trampa común |
|--------|------|--------------|--------|--------------------|--------------|
| **EBS** | Block storage | ❌ No | 1 AZ | Disco para UNA EC2 | Pensar que se comparte |
| **EFS** | File system (NFS) | ✅ Sí | Multi-AZ (regional) | Filesystem Linux compartido | Confundirlo con S3 |
| **FSx Windows** | File system (SMB) | ✅ Sí | Multi-AZ | File share Windows / AD | Usar EFS para Windows |
| **FSx Lustre** | File system (HPC) | ✅ Sí | Multi-AZ | HPC, ML, alto throughput | Usarlo como EFS |
| **S3** | Object storage | ✅ Sí | Regional | Objetos, backups, static web | Pensar que es filesystem |
| **DynamoDB** | NoSQL (Key-Value) | N/A | Regional | Serverless, escala masiva | Buscar puertos/IP |
| **RDS** | Relacional | ❌ No | 1 AZ / Multi-AZ | SQL gestionado estándar | Multi-AZ = lecturas ❌ |
| **Aurora** | Relacional (cluster) | ❌ No | Multi-AZ | SQL + HA + escala lecturas | Confundir con DynamoDB |

---

### Networking

### Serverless

## Enlaces a apuntes detallados

- [RDS](rds.md)
- [Aurora](aurora.md)
- [DynamoDB](dynamodb.md)
- [EFS & FSx](efs-fsx.md)

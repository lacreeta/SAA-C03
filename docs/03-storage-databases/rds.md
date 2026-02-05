# RDS (Relational Database Service) — SAA-C03

> Enfoque: **decisiones de arquitectura** (cuándo elegir qué), trade-offs y trampas típicas de examen.

---

## 1) Qué es RDS
**Amazon RDS** es un servicio gestionado para bases de datos relacionales. AWS se encarga de gran parte de la operación: provisioning, backups, parches del motor (según configuración), monitoreo y opciones de alta disponibilidad.

**Motores comunes en RDS:**
- MySQL / MariaDB
- PostgreSQL
- Oracle
- SQL Server

### Qué te quita RDS de encima
- Instalación y mantenimiento del motor
- Backups automáticos y snapshots
- Parches (gestionados / controlados según ventana)
- Failover con Multi-AZ
- Métricas/monitorización (CloudWatch) e integraciones

### Qué NO te quita
- Modelado (schema, índices)
- Tuning de queries
- Decidir tamaño/IOPS/almacenamiento
- Seguridad (IAM, SG, KMS, etc.)

---

## 2) Conceptos clave (examen)

### 2.1 Multi-AZ (Alta disponibilidad)
**Multi-AZ** en RDS significa:
- Se crea un **standby** (réplica sincronizada) en **otra AZ**
- RDS hace **replicación síncrona**
- Si hay fallo (AZ, hardware, etc.) se hace **failover automático**

✅ **Cuándo elegir Multi-AZ:**
- Producción
- Requisitos de alta disponibilidad (HA)
- Queremos minimizar downtime ante fallos

⚠️ **Trampa típica:**
- Multi-AZ **no es para escalar lecturas** (para eso: **Read Replicas**)

### 2.2 Read Replicas (Escalado de lecturas)
**Read Replicas**:
- Replicación **asíncrona** (normalmente)
- Se usan para **descargar lecturas** del primario
- Pueden existir múltiples réplicas

✅ **Cuándo elegir Read Replicas:**
- Mucho tráfico de lectura
- Reportes/BI, dashboards
- Separar cargas (lectura vs escritura)

⚠️ **Trampas típicas:**
- Read Replica puede tener **lag** (no es “read-your-writes” garantizado)
- Para DR multi-región: Read Replica cross-region (según motor)

### 2.3 Backups automáticos vs Snapshots
- **Backups automáticos**: continuos + retención; permiten **point-in-time restore (PITR)**.
- **Snapshots**: manuales o programadas; se usan para **copias puntuales** o antes de cambios.

📌 Examen:
- “Restaurar a un punto exacto en el tiempo” → **PITR / backups automáticos**

---

## 3) Almacenamiento: gp3 vs Provisioned IOPS
En RDS eliges el tipo de almacenamiento (depende del motor y opciones):
- **General Purpose (gp3/gp2)**: balance coste/rendimiento.
- **Provisioned IOPS (io1/io2)**: para cargas intensivas con latencia/IOPS predecibles.

📌 Examen:
- “Necesito IOPS consistentes y alta performance” → **Provisioned IOPS**

---

## 4) Seguridad en RDS

### 4.1 Seguridad de red
- RDS vive en tu **VPC**
- Controlas acceso con **Security Groups**
- Buen patrón: RDS en **subnets privadas**, sin IP pública

📌 Examen:
- “BD no debe ser accesible desde internet” → **subnet privada + SG restrictivo**

### 4.2 Cifrado
- **En reposo**: con **KMS** (ideal habilitar desde el principio; si no, suele implicar snapshot/restore)
- **En tránsito**: TLS/SSL entre aplicación y BD

### 4.3 Auth / credenciales
- Usuarios/password del motor
- En algunos motores: integración IAM para autenticación (a alto nivel)

---

## 5) Mantenimiento, parches y ventanas
RDS permite configurar:
- **Maintenance window** (parches del motor/OS administrado)
- **Backup window**

📌 Examen:
- “Minimizar impacto de mantenimiento” → configurar ventanas

---

## 6) Observabilidad
- **CloudWatch metrics**: CPU, conexiones, free storage, read/write IOPS, etc.
- **Enhanced Monitoring** (más granular a nivel SO)
- **Performance Insights**: análisis de carga/queries (útil para diagnóstico)

📌 Examen:
- “Analizar queries lentas / cuello de botella” → **Performance Insights**

---

## 7) Patrones típicos de arquitectura

### Patrón A: Web app en 2 AZ + RDS Multi-AZ
- EC2/ECS en 2 AZ
- RDS Multi-AZ
- SG: app → RDS en puerto del motor

✅ HA sin complicación.

### Patrón B: Muchas lecturas
- RDS primario + 1..N Read Replicas
- App de lectura/reporting apuntando a réplicas

✅ Escala lecturas sin tocar writes.

### Patrón C: DR / región secundaria
- Snapshots/copias o replicación (según motor)
- A menudo: Read Replica cross-region

✅ Mejor RTO/RPO.

---

## 8) Preguntas tipo examen (cómo decidir)
1) **Quiero alta disponibilidad** → **Multi-AZ**
2) **Quiero escalar lecturas** → **Read Replicas**
3) **Quiero restaurar a un punto en el tiempo** → **Backups automáticos (PITR)**
4) **Quiero rendimiento de disco predecible** → **Provisioned IOPS**
5) **La BD no debe tener exposición pública** → **subnets privadas + SG + KMS**

---

## 9) Trampas frecuentes
- Multi-AZ ≠ escala lecturas.
- Read Replicas pueden tener lag.
- “Pública” o “privada” no es por el checkbox de IP pública, sino por routing/subnets/SG.
- KMS es una decisión de diseño: actívalo desde el principio si el enunciado habla de cifrado.

---

## 10) Mini-resumen
- **Multi-AZ**: HA y failover automático.
- **Read Replicas**: escalar lecturas.
- **Backups automáticos**: PITR.
- **Snapshots**: copia puntual.
- **Seguridad**: subnet privada + SG + KMS.

## Laboratorio
👉 [RDS — Laboratorio](rds-lab.md)
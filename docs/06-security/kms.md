# AWS KMS — Key Management Service (SAA-C03)

> KMS es el **servicio central de cifrado en AWS**.
> En el examen SAA‑C03 aparece constantemente, muchas veces **de forma indirecta**,
> cuando el enunciado habla de:
>
> - “encrypt data at rest”
> - “customer-managed keys”
> - “control access to encrypted data”
> - “compliance requirements”
>
> El objetivo de este documento es que **entiendas KMS como sistema**,  
> no como una lista de opciones del menú.

---

## Mapa mental rápido (imprescindible)

| Necesidad del enunciado | Respuesta |
|------------------------|-----------|
| Cifrar datos en reposo en AWS | **KMS** |
| Controlar quién puede usar una clave | **IAM + Key Policy** |
| Claves gestionadas por AWS | **AWS-managed CMK** |
| Control total y auditoría | **Customer-managed CMK** |
| Cumplimiento/compliance | **KMS** |

📌 Regla de oro:
> **KMS no guarda datos, guarda CLAVES**.

---

# 1) Qué es AWS KMS (y qué NO es)

## 1.1 Qué es KMS
AWS KMS es un servicio gestionado que:
- Crea y gestiona **claves criptográficas**
- Cifra y descifra datos **a petición de otros servicios**
- Integra control de acceso con **IAM**

👉 Piensa en KMS como:
> *“la autoridad central de cifrado de AWS”*

---

## 1.2 Qué NO es KMS
- ❌ No almacena datos
- ❌ No almacena secretos (eso es Secrets Manager / SSM)
- ❌ No cifra tráfico en tránsito (eso es TLS)
- ❌ No reemplaza IAM

KMS **solo gestiona claves y operaciones criptográficas**.

---

# 2) Tipos de claves en KMS (MUY de examen)

## 2.1 AWS-managed keys
- Creadas y gestionadas por AWS
- Asociadas automáticamente a servicios (ej. `aws/s3`, `aws/ebs`)
- Sin coste directo por clave

### Cuándo usarlas
- No necesitas control fino
- Quieres simplicidad
- No hay requisitos estrictos de compliance

📌 En examen:
> *“Encrypt data at rest with minimal effort”* → **AWS-managed key**

---

## 2.2 Customer-managed keys (CMK)
- Creadas por ti
- Control total:
  - rotación
  - políticas
  - auditoría
- Coste mensual por clave

### Cuándo usarlas
- Requisitos de compliance
- Control de acceso fino
- Auditoría estricta
- Cross-account access

📌 En examen:
> *“Need full control over encryption keys”* → **Customer-managed CMK**

---

## 2.3 AWS-owned keys
- Totalmente gestionadas por AWS
- No visibles en tu cuenta
- Poco relevantes para el examen

---

# 3) Envelope Encryption (concepto CLAVE)

## 3.1 Qué es
KMS **no cifra grandes volúmenes directamente**.

Proceso:
1. KMS cifra una **Data Key**
2. El servicio cifra los datos con esa Data Key
3. KMS protege la Data Key

👉 Esto se llama **envelope encryption**.

📌 En examen:
- No te piden implementarlo
- Pero explica por qué KMS escala bien

---

# 4) Key Policies vs IAM Policies (TRAMPA CLÁSICA)

## 4.1 Key Policy
- Política **propia de la clave**
- Define:
  - quién puede usar la clave
  - quién puede administrarla

📌 **SIN key policy correcta, nadie puede usar la clave**, aunque IAM lo permita.

---

## 4.2 IAM Policy
- Adjunta a users/roles
- Permite usar KMS APIs (`Encrypt`, `Decrypt`, etc.)

📌 Regla mental:
> **Para usar una clave necesitas:**
> - permiso en IAM
> - permiso en la Key Policy

---

## 5) KMS + servicios AWS (casos reales)

## 5.1 S3
- Server-Side Encryption:
  - SSE-S3 (NO KMS)
  - SSE-KMS (usa KMS)

📌 En examen:
> *“Encrypt S3 with KMS”* → **SSE-KMS**

---

## 5.2 EBS
- Volúmenes cifrados
- Snapshots cifrados
- Copia cross-region requiere acceso a la clave

📌 Trampa:
- Snapshot cifrado → volumen cifrado siempre

---

## 5.3 RDS / Aurora
- Cifrado a nivel storage
- No se puede “activar después”
- Réplicas heredan cifrado

---

## 5.4 Lambda / Secrets Manager / SSM
- Usan KMS para cifrar secretos
- Control de acceso vía IAM + Key Policy

---

# 6) KMS y control de acceso (muy preguntado)

### Escenario típico
“Solo ciertas aplicaciones pueden descifrar datos”

👉 Solución:
- Customer-managed CMK
- Key policy restrictiva
- IAM roles específicos

---

### Cross-account access con KMS
- Key policy permite otra cuenta
- IAM role en la otra cuenta

📌 Muy de examen.

---

# 7) Rotación de claves

## 7.1 Automatic rotation
- Solo para **customer-managed CMK**
- Rotación anual automática

📌 En examen:
> *“Automatic key rotation”* → **Customer-managed CMK**

---

## 7.2 Manual rotation
- Crear nueva clave
- Re-encriptar datos

No suele entrar en detalle en SAA.

---

# 8) Auditoría y logging

- Todas las llamadas a KMS se registran en **CloudTrail**
- Permite:
  - saber quién usó una clave
  - cuándo
  - desde dónde

📌 En examen:
> *“Audit key usage”* → **CloudTrail + KMS**

---

# 9) Trampas típicas SAA-C03

- ❌ Pensar que KMS cifra datos directamente
- ❌ Pensar que IAM basta sin Key Policy
- ❌ Confundir KMS con Secrets Manager
- ❌ Pensar que SSE-S3 usa KMS (NO)
- ❌ Pensar que puedes descifrar sin permisos explícitos

---

# 10) Tabla de decisiones (oro puro)

| Requisito | Elección |
|----------|----------|
| Cifrado simple y automático | AWS-managed key |
| Control total de claves | Customer-managed CMK |
| Cumplimiento / auditoría | Customer-managed CMK |
| S3 encryption con KMS | SSE-KMS |
| Secrets cifrados | Secrets Manager + KMS |
| Control de acceso fino | Key Policy + IAM |

---

# 11) Labs recomendados (los que sí valen la pena)

## 🧪 Lab 1 — S3 + SSE-KMS (IMPRESCINDIBLE)
1. Crear CMK
2. Crear bucket S3
3. Activar SSE-KMS
4. Probar acceso con/ sin permisos

👉 Muy didáctico y rápido.

---

## 🧪 Lab 2 — EBS cifrado + snapshot
1. Crear volumen cifrado
2. Crear snapshot
3. Copiar snapshot a otra región

👉 Ayuda a entender DR + KMS.

---

## 🧠 Labs NO necesarios
- Implementar envelope encryption a mano
- KMS APIs avanzadas

---

# 12) Mini-resumen final

- KMS gestiona **claves**, no datos
- AWS-managed = simple
- Customer-managed = control
- Key Policy + IAM = acceso
- Envelope encryption explica escalabilidad
- KMS aparece en MUCHAS preguntas indirectas

---

## Cierre
Si entiendes bien KMS, **muchas preguntas de cifrado se responden solas**.
No memorices opciones: **razona control, acceso y compliance**.

## Laboratorio
👉 [Security — Laboratorio](security-lab.md)
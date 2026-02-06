# AWS Shared Responsibility Model — Responsabilidad Compartida (SAA-C03)

> El **Shared Responsibility Model** es de esas cosas que parecen “teoría básica”,
> pero en SAA-C03 se usa muchísimo para **preguntas trampa**.
>
> La idea no es memorizar una tabla, sino entender el principio:
>
> ✅ AWS es responsable de la **seguridad DEL cloud**  
> ✅ Tú eres responsable de la **seguridad EN el cloud**
>
> Este documento te enseña a razonar escenarios de examen y a no caer en ambigüedades.

---

## 0) La frase clave (para repetir mentalmente)

> **AWS secures the infrastructure. You secure what you build on top.**

Cuando el enunciado pregunta “¿quién es responsable de…?”:
1) piensa si es **infra física / capa base**
2) o es **configuración / datos / accesos**

---

# 1) Qué significa realmente “del cloud” vs “en el cloud”

## 1.1 Seguridad **DEL** cloud (AWS)
AWS se encarga de:
- Centros de datos (seguridad física)
- Hardware (servidores, discos)
- Redes backbone
- Hypervisor y capa de virtualización
- Disponibilidad física de la infraestructura

👉 Esto es invisible para ti, pero es la base.

---

## 1.2 Seguridad **EN** el cloud (cliente)
Tú te encargas de:
- Identidad y accesos (IAM)
- Configuración de red (SG/NACL/VPC)
- Cifrado y claves (KMS, cifrado en servicios)
- Configuración de servicios (S3, RDS, etc.)
- Datos y su clasificación (PII, compliance)
- Parches dentro de tus sistemas (según servicio)

👉 Esto es exactamente lo que define la seguridad real de tu arquitectura.

---

# 2) El modelo depende del tipo de servicio (IaaS / PaaS / SaaS)

Esta es la parte que más se pregunta en examen:  
**cuanto más “managed” es el servicio, menos responsabilidad tienes tú en operación, pero nunca en datos y accesos.**

---

## 2.1 IaaS (ej: EC2)
AWS:
- Infra física
- Hypervisor

Tú:
- Sistema operativo (parches, hardening)
- Software instalado
- Security Groups
- Configuración de apps
- Datos y cifrado

📌 En examen:
> “Apply OS patches” en EC2 → **tú**

---

## 2.2 PaaS / Managed (ej: RDS, Lambda)
AWS:
- OS del servicio
- Parches del motor (según servicio)
- Alta disponibilidad del servicio (capas internas)

Tú:
- Configuración (public/private, encryption)
- Permisos (IAM, security groups de RDS)
- Datos, backups y retención
- Usuarios/roles dentro del DB (en RDS)

📌 En examen:
> “Patch database engine for RDS” → **AWS (en general)**  
pero
> “Configure security groups and encryption for RDS” → **tú**

---

## 2.3 SaaS (ej: algunos servicios totalmente gestionados)
AWS:
- casi todo el stack

Tú:
- control de acceso
- configuración de seguridad disponible
- datos y cumplimiento

📌 Regla:
> Aunque AWS gestione el servicio, **tú sigues siendo responsable de tus datos**.

---

# 3) Tabla mental (rápida y útil)

| Área | Responsable |
|------|-------------|
| Seguridad física del datacenter | AWS |
| Hardware y red backbone | AWS |
| Hypervisor | AWS |
| Configuración de IAM | Cliente |
| Configuración de VPC/SG/NACL | Cliente |
| Datos (clasificación, acceso, retención) | Cliente |
| Cifrado (decidir y activar) | Cliente |
| Parches en EC2 | Cliente |
| Parches del servicio en RDS/Lambda | AWS (en gran parte) |

---

# 4) Ejemplos de examen (razonados)

## Caso A — “Who is responsible for patching EC2 operating system?”
👉 **Cliente**

Por qué:
- EC2 = IaaS
- AWS no entra en tu SO

---

## Caso B — “Who is responsible for the physical security of the servers running EC2?”
👉 **AWS**

Por qué:
- infra física

---

## Caso C — “Who is responsible for enabling encryption for an S3 bucket?”
👉 **Cliente**

Por qué:
- AWS ofrece la opción
- tú decides activarla y qué política usar

---

## Caso D — “Who is responsible for DDoS protection?”
👉 Depende:
- Shield Standard básico → AWS lo provee automáticamente (infra)
- Shield Advanced y diseño → cliente decide si lo compra/configura

📌 En examen:
- “Basic DDoS protection” → AWS ya lo da
- “Need enhanced protection” → cliente elige Shield Advanced

---

## Caso E — “Who is responsible for configuring security groups for RDS?”
👉 **Cliente**

Porque:
- aunque RDS sea managed, tú controlas red y acceso

---

# 5) Trampas típicas SAA-C03

## 5.1 “AWS handles security” (falso)
AWS gestiona mucha parte de la infraestructura, pero:
- si tú haces un bucket público, el fallo es tuyo
- si abres SSH al mundo, el fallo es tuyo
- si metes claves en Git, el fallo es tuyo

---

## 5.2 Confundir “managed” con “sin responsabilidad”
Un servicio managed:
- reduce operación
- pero no elimina:
  - IAM
  - permisos
  - datos
  - configuración

---

## 5.3 “Encryption at rest” (quién lo habilita)
AWS provee mecanismos, pero:
- tú eliges activarlo
- tú eliges KMS/keys/policies
- tú gestionas acceso

---

# 6) Cómo responder preguntas de Shared Responsibility (método rápido)

1. ¿Es físico/hardware/backbone/hypervisor? → **AWS**
2. ¿Es configuración, permisos, datos, red, cifrado? → **Cliente**
3. ¿Es servicio managed (RDS/Lambda)?  
   - AWS gestiona SO/motor (en general)
   - tú configuras y proteges datos

---

# 7) Conexión con “Security Architecture”
Shared Responsibility es el fundamento de:
- Defense in Depth
- Least privilege
- Compliance

Porque:
- AWS te da herramientas
- tú diseñas la arquitectura segura

---

# 8) Mini-resumen final (para memorizar)

- AWS: seguridad **DEL** cloud (infra)
- Tú: seguridad **EN** el cloud (config/datos/acceso)
- Más managed = menos operación, pero **datos y accesos siguen siendo tuyos**
- En examen, mira si el problema es “infra” o “configuración”

---

## Cierre
Si dominas este modelo, evitas errores de examen del tipo:
- “AWS es responsable de mis permisos” (NO)
- “RDS significa que no configuro seguridad” (NO)

Este fichero es una base que debes tener clara antes de profundizar en servicios.

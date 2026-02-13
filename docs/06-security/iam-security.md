# IAM (Identity and Access Management) — Security profundo (SAA-C03)

> IAM es **EL servicio más importante de seguridad en AWS**.  
> No es solo “usuarios y permisos”: es **cómo se controla quién puede hacer qué, cuándo y desde dónde**.
>
> En SAA‑C03, IAM:
> - aparece en **muchísimas preguntas**
> - suele ser la **pieza clave correcta**
> - se mezcla con casi todos los servicios (EC2, S3, Lambda, RDS, KMS, etc.)
>
> Este documento está enfocado a:
> - **comprender IAM**
> - **razonar decisiones**
> - **evitar trampas de examen**
>
> No es para memorizar políticas JSON, sino para **pensar como arquitecto**.

---

## 0) Mapa mental antes de empezar

### La pregunta que SIEMPRE responde IAM
> **¿Quién puede hacer QUÉ sobre QUÉ recurso y en QUÉ condiciones?**

IAM responde eso con:
- identidades
- políticas
- relaciones de confianza
- credenciales temporales

---

## 1) Qué es IAM (y qué NO es)

### Qué es IAM
- Servicio **global** (no regional)
- Controla:
  - autenticación (quién eres)
  - autorización (qué puedes hacer)
- Aplica a **todos los servicios AWS**

### Qué NO es IAM
- ❌ No es un firewall
- ❌ No protege tráfico de red
- ❌ No cifra datos
- ❌ No detecta ataques

IAM es **control de acceso**, no protección perimetral.

---

## 2) Identidades IAM (los “quién”)

### 2.1 Root User (solo para crear la cuenta)
- Control total
- Credenciales de altísimo riesgo

📌 Buenas prácticas (y examen):
- Root solo para:
  - crear la cuenta
  - configurar MFA
- ❌ Nunca usar root para trabajo diario

---

### 2.2 IAM Users
- Identidad **permanente**
- Tienen:
  - username
  - credenciales (password / access keys)

#### Cuándo usar usuarios
- Personas reales
- Casos simples
- Pequeños equipos

#### Trampas
- ❌ Usar access keys en aplicaciones → mala práctica
- ❌ Usuarios para servicios → mal diseño

📌 En examen:
> *“Avoid long-term credentials”* → **NO users, usar roles**

---

### 2.3 IAM Groups
- Agrupan usuarios
- **NO tienen credenciales**
- Sirven para **gestionar permisos en bloque**

Ejemplo:
- Group: `Developers`
- Policy: acceso a EC2/S3
- Usuarios heredan permisos

📌 Conceptual, poco preguntado directamente.

---

### 2.4 IAM Roles (los más importantes)

#### Qué es un Role
- Identidad **sin credenciales propias**
- Se **asume** temporalmente
- Usa **credenciales temporales**

👉 Piensa en un role como:
> *“un sombrero que te pones durante un rato”*

---

#### Casos típicos (MUY de examen)

##### a) Servicios AWS
- EC2 necesita acceder a S3
- Lambda necesita escribir en DynamoDB

👉 **IAM Role para el servicio**, no keys hardcoded.

##### b) Cross-account access
- Cuenta A quiere acceder a recursos en cuenta B

👉 **AssumeRole + Trust Policy**

##### c) Acceso temporal
- Usuarios externos
- Acceso limitado en el tiempo

👉 **STS + Role**

---

#### Por qué Roles > Users
- No hay secrets largos
- Credenciales rotan automáticamente
- Mucho más seguro

📌 En examen:
> *“Temporary credentials”* → **IAM Role / STS**

---

## 3) Políticas IAM (los “qué”)

### 3.1 Qué es una Policy
Documento JSON que define:
- **Effect**: Allow / Deny
- **Action**: qué acción
- **Resource**: sobre qué recurso
- **Condition**: bajo qué condiciones

👉 Mentalidad:
> *Allow lo mínimo necesario*

---

### 3.2 Identity-based vs Resource-based (trampa clave)

#### Identity-based policy
- Se adjunta a:
  - User
  - Group
  - Role
- Dice:
  > “Esta identidad puede hacer X”

Ejemplo:
- Role EC2 puede leer S3

---

#### Resource-based policy
- Se adjunta al recurso
- Dice:
  > “Este recurso permite acceso a esta identidad”

Ejemplos:
- S3 Bucket Policy
- SNS Topic Policy
- SQS Queue Policy
- KMS Key Policy

📌 Trampa de examen:
- Cross-account access **requiere** resource-based policy + role

---

### 3.3 Explicit DENY (muy importante)
- **DENY siempre gana**
- Incluso si hay Allow en otro sitio

📌 En examen:
> *“Why is access denied even though there is an allow?”*  
→ **Explicit deny**

---

## 4) STS (Security Token Service)

### Qué es STS
Servicio que:
- emite **credenciales temporales**
- para roles

Usado por:
- EC2
- Lambda
- Cross-account
- Identity Federation

📌 No suele aparecer solo, sino implícito.

---

## 5) Trust Policies (el “quién puede asumir el role”)

### Trust Policy
Define:
- **quién puede asumir un role**

Ejemplo:
- EC2 puede asumir este role
- Otra cuenta puede asumir este role

📌 Trampa clásica:
- Permissions policy ≠ Trust policy
- Una dice *qué puede hacer*
- La otra *quién puede asumir*

---

## 6) IAM y servicios (casos reales)

### 6.1 EC2 accediendo a S3
❌ Access keys en la instancia  
✅ **IAM Role asociado a la EC2**

---

### 6.2 Lambda accediendo a DynamoDB
❌ Credenciales hardcoded  
✅ **Execution Role de Lambda**

---

### 6.3 Cross-account S3 access
- Role en cuenta B
- Trust policy permite a cuenta A
- Bucket policy permite acceso

📌 Aparece mucho en examen.

---

## 7) Buenas prácticas (mucho de examen)

- Least privilege
- Roles > Users
- MFA para usuarios humanos
- No hardcodear secrets
- Policies pequeñas y claras
- Usar condiciones (IP, MFA, time)

---

## 8) Errores y trampas típicas SAA-C03

- Pensar que IAM es regional → ❌
- Usar users para servicios → ❌
- Confundir role con policy → ❌
- Ignorar trust policy → ❌
- Olvidar explicit deny → ❌

---

## 9) Tabla mental de decisiones (oro puro)

| Escenario | Respuesta |
|---------|-----------|
| EC2 accede a AWS | IAM Role |
| Lambda accede a AWS | IAM Role |
| Cross-account | AssumeRole |
| Evitar credenciales largas | Roles + STS |
| Acceso humano | IAM User + MFA |
| Control fino | Policies + Conditions |

---

## 10) IAM NO hace esto (para descartar opciones)
- No cifra datos → KMS
- No protege web apps → WAF
- No detecta amenazas → GuardDuty
- No audita → CloudTrail

---

# 11) LABS recomendados (solo los que valen la pena)

## 🧪 Lab 1 — IAM Role para EC2 (IMPRESCINDIBLE)
**Objetivo:** entender roles vs access keys

Pasos:
1. Crear role EC2 con acceso S3 read-only
2. Asociarlo a una EC2
3. Acceder a S3 desde la instancia sin credenciales

👉 Este lab **vale oro**.

---

## 🧪 Lab 2 — Lambda Execution Role
**Objetivo:** entender permisos mínimos

Pasos:
1. Lambda básica
2. Role con permiso a CloudWatch Logs
3. Ver qué pasa si quitas permisos

---

## 🧪 Lab 3 — Cross-account AssumeRole (opcional)
**Objetivo:** entender trust + permissions

Si tienes 2 cuentas:
- role en cuenta B
- acceso desde cuenta A

Si no, basta con entenderlo conceptualmente.

---

## 🧠 Labs que NO recomiendo
- Policies complejas a mano
- IAM Access Analyzer profundo
- Identity Federation avanzada

Mucho ruido para poco SAA.

---

## 12) Mini-resumen final (para repasar)

- IAM = control de acceso
- Roles > Users
- Temporary credentials > long-term
- Identity vs Resource policies
- Explicit deny wins
- Trust policy controla quién asume

---

## Cierre
Si entiendes IAM, **la mitad de las preguntas de seguridad se vuelven obvias**.  
No memorices JSON: **razona escenarios**.

Este fichero, bien entendido, te da una **ventaja enorme** en SAA‑C03.

## Laboratorio
👉 [Security — Laboratorio](security-lab.md)
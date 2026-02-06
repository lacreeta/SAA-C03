# AWS Secrets Manager & SSM Parameter Store — Gestión segura de secretos (SAA-C03)

> Este documento cubre **cómo AWS gestiona secretos y configuración sensible**.
> En el examen SAA-C03 aparece constantemente de forma implícita cuando el enunciado dice:
>
> - “Do not hardcode credentials”
> - “Store database passwords securely”
> - “Rotate secrets automatically”
>
> El objetivo aquí es **entender por qué existen estos servicios**,  
> **cuándo usar cada uno**, y **evitar confundirlos con IAM o KMS**.

---

## Mapa mental rápido

| Necesidad del enunciado | Servicio |
|------------------------|----------|
| Guardar passwords / API keys de forma segura | Secrets Manager |
| Rotación automática de credenciales | Secrets Manager |
| Configuración simple (no crítica) | SSM Parameter Store |
| Integración con KMS | Ambos |
| No hardcodear secretos | Ambos |

📌 Regla de oro:
> **IAM controla quién accede**  
> **Secrets Manager / SSM controlan dónde se guarda el secreto**

---

# 1) El problema que resuelven

## 1.1 El anti‑patrón clásico
❌ Credenciales en:
- Código fuente
- Variables de entorno hardcodeadas
- Scripts
- Repositorios Git

Consecuencias:
- Riesgo de fuga
- Difícil rotación
- Incidentes de seguridad

📌 En examen:
> *“Avoid hardcoding credentials”* → **Secrets Manager / SSM**

---

## 1.2 Qué hacen estos servicios
- Almacenan secretos **cifrados**
- Controlan acceso vía **IAM**
- Permiten rotación (según servicio)
- Centralizan gestión

---

# 2) AWS Secrets Manager

## 2.1 Qué es Secrets Manager
Servicio gestionado para:
- Guardar secretos sensibles
- **Rotarlos automáticamente**
- Integrarse con servicios AWS

👉 Piensa en Secrets Manager como:
> *“la caja fuerte viva de AWS”*

---

## 2.2 Qué tipo de secretos guarda
- Passwords de bases de datos
- API keys
- Tokens
- Credenciales de terceros

---

## 2.3 Cifrado (relación con KMS)
- Todos los secretos están cifrados **at rest**
- Usa **KMS** (customer‑managed o AWS‑managed keys)

📌 Trampa:
- KMS **no almacena secretos**
- Secrets Manager **sí**, usando KMS para cifrar

---

## 2.4 Rotación automática (CLAVE de examen)

### Cómo funciona conceptualmente
- Secrets Manager invoca una **Lambda**
- La Lambda:
  - cambia el secreto en el backend (RDS, etc.)
  - actualiza el valor almacenado
- Todo sin intervención humana

📌 En examen:
> *“Automatic rotation of database credentials”* → **Secrets Manager**

---

## 2.5 Integración típica
- RDS (MySQL, PostgreSQL, etc.)
- Lambda
- ECS / EC2
- Aplicaciones serverless

---

## 2.6 Coste
- **Tiene coste mensual**
- Justificado cuando:
  - el secreto es crítico
  - necesitas rotación automática

📌 En examen:
- Si piden **low cost** y no mencionan rotación → quizá **SSM**

---

## 2.7 Qué Secrets Manager NO hace
- ❌ No gestiona permisos (eso es IAM)
- ❌ No sustituye KMS
- ❌ No detecta secretos filtrados

---

# 3) SSM Parameter Store

## 3.1 Qué es Parameter Store
Servicio de **Systems Manager** para:
- Almacenar parámetros de configuración
- Guardar secretos simples
- Centralizar configuración

👉 Piensa en Parameter Store como:
> *“un almacén de configuración seguro”*

---

## 3.2 Tipos de parámetros
- **String**
- **StringList**
- **SecureString** (cifrado con KMS)

📌 Para secretos:
> **SecureString**

---

## 3.3 Cifrado
- SecureString se cifra con **KMS**
- Puede usar customer‑managed keys

---

## 3.4 Ventajas de Parameter Store
- Más **barato**
- Simple
- Integrado con SSM
- Ideal para:
  - variables de entorno
  - flags
  - configuración sensible pero estable

---

## 3.5 Limitaciones (por qué NO siempre vale)
- ❌ No rotación automática integrada
- ❌ Menos features avanzadas
- ❌ Menos enfoque “secreto crítico”

📌 En examen:
- Si piden **rotación automática** → **NO Parameter Store**

---

# 4) Secrets Manager vs Parameter Store (comparativa CLAVE)

| Característica | Secrets Manager | Parameter Store |
|--------------|----------------|----------------|
| Guarda secretos | ✅ | ✅ |
| Cifrado con KMS | ✅ | ✅ |
| Rotación automática | ✅ | ❌ |
| Coste | 💰 | 💲 |
| Configuración simple | ❌ | ✅ |
| Secrets críticos | ✅ | ⚠️ |

📌 Regla mental:
- **Crítico + rotación** → Secrets Manager
- **Simple + barato** → Parameter Store

---

# 5) Integración con IAM (muy importante)

## Control de acceso
El acceso a secretos se controla con:
- IAM policies
- Principals (roles, users)

Ejemplo conceptual:
- Lambda Role puede leer un secreto
- Otra Lambda no

📌 Trampa:
- Guardar secreto ≠ dar acceso

---

# 6) Casos típicos de examen

### Caso 1
“Store database credentials securely and rotate them automatically”

👉 **AWS Secrets Manager**

---

### Caso 2
“Application configuration values stored securely with minimal cost”

👉 **SSM Parameter Store (SecureString)**

---

### Caso 3
“Avoid hardcoding credentials in EC2 instances”

👉 **Secrets Manager o SSM + IAM Role**

---

### Caso 4
“Encrypt secrets at rest”

👉 **Secrets Manager / SSM + KMS**

---

# 7) Trampas típicas SAA‑C03

- ❌ Usar IAM para guardar secretos
- ❌ Pensar que KMS guarda passwords
- ❌ Pensar que Parameter Store rota secretos
- ❌ Hardcodear secretos en variables de entorno

---

# 8) Labs recomendados (los que sí valen la pena)

## 🧪 Lab 1 — Secrets Manager + Lambda (RECOMENDADO)
**Objetivo:** ver acceso seguro a secretos

Pasos:
1. Crear secreto en Secrets Manager
2. Lambda con execution role
3. Leer secreto desde Lambda

👉 Muy didáctico y rápido.

---

## 🧪 Lab 2 — Parameter Store SecureString
**Objetivo:** entender SecureString + KMS

Pasos:
1. Crear SecureString
2. Acceder desde EC2 o Lambda
3. Probar permisos IAM

---

## 🧠 Labs NO necesarios
- Rotación real de RDS (caro)
- Integraciones complejas

Para SAA‑C03 basta con entender el concepto.

---

# 9) Mini‑resumen para memorizar

- No hardcodear secretos → Secrets Manager / SSM
- Rotación automática → Secrets Manager
- Configuración barata → Parameter Store
- Ambos cifran con KMS
- IAM controla acceso

---

## Cierre
Secrets Manager y Parameter Store **no compiten con IAM ni KMS**:  
son el **lugar correcto para guardar secretos**.

Si sabes diferenciar **secreto crítico** vs **configuración sensible**,  
la respuesta correcta en el examen suele ser inmediata.

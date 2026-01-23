## Qué es IAM

Servicio global de AWS que controla **quién puede hacer qué sobre qué recursos y en qué condiciones**.

IAM **no proporciona conectividad de red**, solo **autorización**.

---

## Objetivo en el examen SAA-C03

IAM aparece como **componente transversal**:

* Seguridad
* Acceso entre servicios
* Principio de mínimo privilegio
* Buenas prácticas frente a soluciones inseguras

El examen evalúa **decisiones**, no sintaxis JSON ni comandos CLI.

---

## Conceptos fundamentales

### Autenticación vs Autorización

* **Autenticación**: quién eres (login)
* **Autorización**: qué puedes hacer (policies)

Son procesos **separados**.

---

## Identidades IAM (Who)

### Root user

* Acceso total a la cuenta
* Usa email + password
* No usa account alias

**Buenas prácticas**:

* No usar para tareas diarias
* Activar MFA
* Usar solo para acciones de nivel cuenta

**Trampa de examen**:
Nunca usar root para acceder a servicios o aplicaciones.

---

### IAM User

Representa una **persona o aplicación legacy**.

Características:

* No tiene permisos por defecto
* Puede tener:

  * Password (console)
  * Access keys (CLI / SDK)

**Buenas prácticas**:

* Permisos vía groups
* Evitar access keys si es posible

**Trampa de examen**:
Crear usuarios para servicios AWS es mala práctica.

---

### IAM Group

* Contenedor de permisos
* Solo contiene **usuarios**
* No puede usarse por servicios

**Uso recomendado**:

* Gestionar permisos de usuarios
* Facilitar administración

---

### IAM Role ⭐

Identidad **asumible**, sin credenciales permanentes.

Usado por:

* EC2
* Lambda
* ECS / EKS
* Cross-account access

Características:

* Credenciales temporales
* Basado en **trust policy**

**Regla clave de examen**:
Si un servicio AWS necesita acceder a otro → **IAM Role**.

---

## Policies (What)

### Identity-based policies

* Se adjuntan a users, groups o roles
* Definen qué acciones están permitidas o denegadas

---

### Resource-based policies

* Se adjuntan al recurso (ej. S3 bucket)
* Definen quién puede acceder al recurso

Ejemplos:

* S3 Bucket Policy
* SQS Queue Policy

---

### Estructura básica de una policy

* **Effect**: Allow | Deny
* **Action**: qué acción
* **Resource**: sobre qué recurso
* **Condition** (opcional)

---

## Evaluación de policies (muy importante)

Orden lógico:

1. Deny explícito
2. Allow
3. Default Deny

**Regla de oro**:
Un **Deny explícito siempre gana**.

---

## Principio de mínimo privilegio

Consiste en otorgar:

* Solo las acciones necesarias
* Solo sobre los recursos necesarios

**El examen siempre favorece**:

* Policies específicas
* Frente a policies amplias (`*`)

---

## Account Alias

* Nombre amigable para el Account ID
* Solo para login de **IAM users**
* No aplica al root

---

## MFA (Multi-Factor Authentication)

* Capa extra de seguridad
* Recomendado para:

  * Root user
  * Usuarios con permisos elevados

**Trampa de examen**:
MFA aparece en escenarios de compliance y seguridad.

---

## IAM y servicios AWS

### EC2

* Acceso a otros servicios → IAM Role
* Nunca usar access keys en la instancia

### Lambda

* Siempre usa IAM Role

### S3

* Identity-based + Resource-based policies
* Separación bucket-level vs object-level

---

## Qué NO hace IAM

* No controla tráfico de red
* No sustituye a Security Groups o NACLs
* No da acceso a internet

---

## Trampas típicas de examen

* Confundir IAM con Security Groups
* Usar usuarios en lugar de roles
* Dar permisos demasiado amplios
* Olvidar el Deny explícito
* Pensar que login implica permisos

---

## Resumen rápido SAA

* Root solo para acciones de cuenta
* Usuarios para personas
* Grupos para gestionar permisos
* Roles para servicios AWS
* Deny explícito siempre gana
* Least privilege siempre es la mejor respuesta

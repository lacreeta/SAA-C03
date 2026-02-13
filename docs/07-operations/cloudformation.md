# AWS CloudFormation — Infrastructure as Code (IaC) (SAA-C03)

> CloudFormation (CFN) es el servicio nativo de AWS para **Infrastructure as Code**.
> En el SAA‑C03 no te van a pedir escribir plantillas completas a mano, pero sí:
>
> - reconocer **cuándo CloudFormation es la respuesta**
> - entender **cómo funcionan stacks, updates, rollbacks**
> - entender **cross‑stack / nested stacks**
> - saber **cómo manejar parámetros, outputs, exports**
> - identificar **patrones de despliegue** y “trampas” (drift, rollback, change sets)
>
> Este documento está enfocado a **comprender** y a tomar **decisiones de arquitectura**.

---

## 0) Mapa mental (para situarte en 10 segundos)

### CloudFormation resuelve:
- **Repetibilidad**: “mismo entorno siempre”
- **Auditabilidad**: infra versionada (plantillas)
- **Seguridad**: menos clicks manuales
- **Escalabilidad**: muchos recursos consistentes
- **Governance**: estándares y control

📌 En examen, CloudFormation aparece cuando el enunciado dice:
- “reliable / repeatable deployments”
- “provision infrastructure consistently”
- “create environments quickly”
- “avoid manual configuration errors”
- “infrastructure as code”

---

## 1) Qué es CloudFormation (y qué NO es)

### Qué es
Un servicio que crea/actualiza/elimina recursos AWS a partir de una plantilla:
- YAML o JSON
- declarativo: defines **estado deseado**, CFN ejecuta

### Qué NO es
- ❌ Un servicio de CI/CD (para eso CodePipeline/CodeBuild/GitHub Actions, etc.)
- ❌ Un “monitor” (aunque puede detectar drift)
- ❌ Un “orquestador de apps” (aunque puede desplegar componentes)

CloudFormation es el **orquestador de infraestructura**.

---

## 2) Conceptos esenciales

### 2.1 Template
Archivo que define recursos y sus relaciones.

Secciones típicas:
- `Parameters`: valores variables
- `Resources`: lo que se crea
- `Outputs`: valores exportables (IDs, ARNs, endpoints)
- `Mappings`, `Conditions`: lógica declarativa
- `Metadata`: info extra (no esencial para SAA)

📌 SAA‑C03: te interesa comprender **Parameters/Resources/Outputs/Exports**.

---

### 2.2 Stack
Una “instancia” de una plantilla.

- Template = receta
- Stack = plato ya cocinado

Un stack:
- crea recursos
- los etiqueta
- mantiene dependencias
- permite updates y rollbacks

---

### 2.3 Change Sets (muy de examen)
Antes de aplicar un update, puedes generar un **Change Set**:
- “preview” de cambios
- ver qué se crea/modifica/elimina
- reducir riesgo

📌 En examen:
> “Review changes before applying” → **Change Sets**

---

### 2.4 Stack Updates y Rollbacks
Cuando actualizas:
- CFN intenta aplicar cambios en orden
- si falla, puede hacer **rollback** a estado anterior

📌 En examen:
> “Automatically roll back on failure” → CloudFormation rollback

---

### 2.5 Deletion Policy (concepto importante)
Qué pasa al borrar un stack con recursos críticos:
- por defecto, se eliminan
- puedes marcar algunos con **Retain** (conceptual)

📌 En examen:
> “Do not delete data when stack is deleted” → Retain (p. ej. RDS, S3)

---

## 3) Cómo razonar CloudFormation en preguntas de SAA-C03

### Señales típicas de “CloudFormation”
- hay que crear entornos (dev/test/prod) iguales
- infra repetible en múltiples regiones/cuentas
- se quiere evitar error manual
- auditoría y trazabilidad de cambios

### Señales de que NO es CloudFormation como respuesta principal
- “deploy code continuously” → CI/CD
- “monitor and alert” → CloudWatch
- “govern accounts at scale” → Organizations/SCPs (aunque CFN puede complementar)

---

## 4) Parametrización y reutilización (clave para entender uso real)

### 4.1 Parameters
Sirven para que una misma plantilla funcione en muchos entornos:
- `EnvironmentName`: dev/prod
- `InstanceType`: t3.micro vs m5.large
- `VpcId`, `SubnetIds`

**Por qué importa en arquitectura:**
- reduce duplicación
- permite “infra multi-entorno” limpia

---

### 4.2 Outputs
- Exponen valores creados por el stack
- Ej: `LoadBalancerDNS`, `VpcId`, `SecurityGroupId`

Esto se usa para:
- conectar stacks
- consumir IDs en automatizaciones

---

### 4.3 Cross-Stack References (Exports/Imports)
Puedes **exportar** outputs:
- Stack A exporta `VpcId`
- Stack B importa `VpcId` y crea recursos dentro

📌 En examen:
> “Share resources between stacks” → Exports/Imports

**Trampa:**
- si Stack B depende del export de A, borrar A se complica
- hay acoplamiento (trade-off)

---

## 5) Nested Stacks (y por qué existen)

### Qué es un Nested Stack
Un stack “padre” que crea otros stacks “hijos” como recursos.

**Por qué se usa:**
- separar por módulos: red, seguridad, app, base de datos
- reutilizar componentes
- plantillas enormes se vuelven mantenibles

📌 En examen:
> “Break large templates into smaller reusable components” → **Nested stacks**

---

## 6) Drift Detection (drift) — concepto de examen
Drift = alguien cambia recursos “a mano” fuera de CloudFormation.

CloudFormation puede detectar drift:
- compara estado real vs template
- marca recursos como drifted

📌 En examen:
> “Detect configuration changes outside IaC” → **Drift detection**

**Por qué importa:**
- evita “configuración fantasma”
- ayuda a compliance y control

---

## 7) Casos de uso complejos (tipo “arquitecto”)

### Caso 1 — Multi-Environment (dev/test/prod) con parámetros + nested stacks
**Problema**
- Necesitas entornos idénticos
- Cambia tamaño/capacidad según entorno

**Diseño CloudFormation**
- Stack “Root” con parámetros (`Env`, `InstanceType`, `DesiredCapacity`)
- Nested stacks:
  - `network.yaml`: VPC/subnets/route tables
  - `security.yaml`: SGs, IAM roles, WAF (si aplica)
  - `compute.yaml`: ALB/ASG/EC2
  - `data.yaml`: RDS/DynamoDB (si aplica)
- Outputs exportados:
  - VpcId, PublicSubnetIds, AlbDns

**Por qué es bueno**
- mantenible
- escalable
- seguro (menos manual)
- reproducible

---

### Caso 2 — Landing Zone “ligera” en múltiples cuentas
**Problema**
- Varias cuentas (dev, prod, shared)
- Quieres baseline consistente (logs, roles, bucket central…)

**CloudFormation en SAA**
- CloudFormation es el motor para provisionar infra
- Organizations/SCP gobiernan “a nivel cuenta”
- (En el mundo real se usa mucho StackSets; en SAA basta con entender “propagar plantillas” a varias cuentas/regiones.)

**Señal de examen**
- “deploy same stack across accounts/regions” → CloudFormation (StackSets como idea)

---

### Caso 3 — Actualizaciones seguras (Change Sets + rollback)
**Problema**
- Actualizar una VPC o un ALB sin romper producción

**Solución**
- Create Change Set
- revisar impacto
- ejecutar
- rollback automático si falla

**Por qué**
- reduce downtime
- reduce cambios destructivos sin querer

---

### Caso 4 — Gestión de recursos “delicados” (Retain en datos)
**Problema**
- stack incluye DB/S3 con datos críticos
- quieres poder borrar/recrear compute sin perder datos

**Solución conceptual**
- separar stacks: `data` vs `compute`
- o usar DeletionPolicy Retain en recursos críticos

**Por qué**
- te permite “tocar compute” sin riesgo de datos

---

## 8) Limitaciones y trade-offs (para razonar mejor)

### 8.1 Stack como “unidad de cambio”
- Un update puede tocar muchos recursos
- Si tu template es un monolito, los cambios son más riesgosos

👉 Por eso nested stacks o separación por dominios.

### 8.2 Import/Export acopla stacks
- Conveniente
- Pero crea dependencias

### 8.3 Drift rompe la confianza
- Si cambias manualmente, tu template deja de ser “verdad”
- Drift detection ayuda a descubrirlo

### 8.4 CloudFormation vs Terraform (examen)
En SAA normalmente te quedas con:
- CloudFormation es **nativo AWS**
- Terraform es multi-cloud (mención conceptual)
Si el enunciado es “AWS‑native IaC” → CloudFormation.

---

## 9) Trampas típicas SAA-C03

- ❌ Confundir CloudFormation con CodeDeploy/CodePipeline (CI/CD)
- ❌ Suponer que todo se actualiza sin reemplazos (algunos cambios requieren “replacement”)
- ❌ Olvidar que drift ocurre por cambios manuales
- ❌ Mezclar datos críticos y compute sin separación (y borrar el stack)

---

## 10) Checklist “de examen” (decisiones rápidas)

- “Infra as code / reproducible / consistent” → **CloudFormation**
- “Split big template into modules” → **Nested stacks**
- “Preview changes” → **Change Sets**
- “Detect manual changes” → **Drift detection**
- “Don’t delete data on stack deletion” → **Retain / separar data stack**
- “Same template across accounts/regions” → **CloudFormation (StackSets idea)**

---

## 11) Labs recomendados (los que realmente valen la pena)

> No hace falta complicarse con 200 recursos. Con 2–3 labs bien elegidos entiendes CFN a nivel SAA.

### 🧪 Lab 1 — Stack básico + update + rollback (IMPRESCINDIBLE)
**Objetivo:** entender ciclo de vida.
1. Crear stack con:
   - VPC mínima o Security Group + EC2 tiny (o solo S3 bucket)
2. Hacer update (por ejemplo cambiar tag/instance type)
3. Forzar un error (parámetro inválido o recurso duplicado) y ver rollback
4. Ver eventos del stack y entender el orden de creación

**Qué aprendes**
- cómo CFN aplica cambios
- eventos / troubleshooting
- rollback real

---

### 🧪 Lab 2 — Change Set (RECOMENDADO)
**Objetivo:** “ver antes de ejecutar”.
1. Crear Change Set para un update
2. Revisar qué se reemplaza / elimina
3. Ejecutar change set

**Qué aprendes**
- evitar cambios destructivos

---

### 🧪 Lab 3 — Outputs + cross-stack import/export (MUY útil)
**Objetivo:** componer stacks.
1. Stack A crea VPC (o SG) y exporta Output
2. Stack B importa y crea recurso dentro (subnet/instance/etc.)
3. Ver dependencia real entre stacks

**Qué aprendes**
- arquitectura modular “de verdad”

---

### 🧪 Lab 4 — Drift detection (opcional pero buenísimo)
1. Crea stack que cree un SG
2. Cambia una regla del SG manualmente en consola
3. Ejecuta Drift Detection y observa resultado

**Qué aprendes**
- el peligro de cambios manuales y cómo detectarlos

---

## Limpieza (para no pagar)
- Borrar stacks (esto elimina recursos asociados salvo Retain)
- Revisar:
  - EC2 (terminate)
  - ALB/Target Groups
  - NAT Gateways (si creaste VPC compleja)
  - EIPs
  - Volúmenes “available”
  - Buckets S3 (si creaste)

---

## Cierre
CloudFormation en SAA‑C03 es menos “escribir YAML” y más:
- **control de cambios**
- **repetibilidad**
- **arquitectura modular**
- **evitar errores manuales**
- **gobernanza y compliance**

## Laboratorio
👉 [Operations — Laboratorio](operations-lab.md)
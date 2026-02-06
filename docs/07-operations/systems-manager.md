# AWS Systems Manager (SSM) — Operaciones, Gestión y Seguridad (SAA-C03)

> AWS Systems Manager es uno de los servicios **más infravalorados pero más poderosos** de AWS.
> En SAA‑C03 aparece cuando el examen habla de:
>
> - “manage EC2 at scale”
> - “reduce operational overhead”
> - “avoid SSH access”
> - “centralized configuration and patching”
>
> El objetivo de este documento es **entender Systems Manager como plataforma**,  
> no como una lista de features sueltas.

---

## 0) Mapa mental rápido

> **Systems Manager = control centralizado de tus instancias y recursos**

No es solo “para EC2”:
- opera EC2, on‑prem, híbrido
- reduce acceso directo
- mejora seguridad y compliance

---

# 1) Qué es AWS Systems Manager (y qué NO es)

## 1.1 Qué es
AWS Systems Manager es un servicio que permite:
- operar instancias **sin SSH**
- automatizar tareas operativas
- gestionar configuración, parches y comandos
- centralizar parámetros y secretos (Parameter Store)

👉 Piensa en SSM como:
> *“el panel de control de tus servidores”*

---

## 1.2 Qué NO es
- ❌ No es un sistema de monitorización (eso es CloudWatch)
- ❌ No es un sistema de CI/CD
- ❌ No sustituye IAM (lo usa)
- ❌ No es solo para Linux

---

# 2) Requisitos clave (trampa de examen)

Para que una instancia funcione con SSM necesita **SIEMPRE**:

1. **SSM Agent instalado**
2. **IAM Role con permisos SSM**
3. **Conectividad** (internet o VPC endpoints)

📌 En examen:
> “SSM not working” → falta uno de estos tres

---

# 3) Session Manager — acceso sin SSH (MUY importante)

## 3.1 Qué es Session Manager
Permite:
- acceder a instancias **sin abrir el puerto 22**
- sin claves SSH
- con logging completo

👉 Es uno de los mayores wins de seguridad.

---

## 3.2 Ventajas de Session Manager
- No necesitas bastion host
- No necesitas IP pública
- Todo queda auditado (CloudTrail + logs)

📌 En examen:
> *“Secure access to EC2 without SSH”* → **Session Manager**

---

## 3.3 Session Manager + Security
- Reduce superficie de ataque
- Elimina keys estáticas
- Integración directa con IAM

---

# 4) Run Command — ejecutar comandos a escala

## 4.1 Qué es Run Command
Permite:
- ejecutar comandos en múltiples instancias
- sin SSH
- de forma paralela

Ejemplos:
- reiniciar servicio
- actualizar configuración
- recopilar información

📌 En examen:
> *“Run commands on many EC2 instances centrally”* → **Run Command**

---

## 4.2 Casos típicos
- Operaciones de emergencia
- Cambios rápidos
- Troubleshooting

---

# 5) Patch Manager — parches y compliance

## 5.1 Qué es Patch Manager
Automatiza:
- instalación de parches del SO
- ventanas de mantenimiento
- reporting de compliance

👉 Importante:
- **para EC2 tú sigues siendo responsable de parches**
- Patch Manager te ayuda a cumplirlo

📌 En examen:
> *“Apply OS patches automatically”* → **Patch Manager**

---

## 5.2 Compliance
Patch Manager se integra con:
- SSM Compliance
- AWS Config

Para saber:
- quién está parcheado
- quién no

---

# 6) Automation — playbooks operativos

## 6.1 Qué es Automation
Permite:
- definir workflows operativos
- pasos secuenciales
- remediación automática

Ejemplos:
- snapshot + reboot
- aplicar parche + validar
- remediar incidente detectado

📌 En examen:
> *“Automate operational tasks”* → **SSM Automation**

---

# 7) Parameter Store (visión operativa)

> El detalle profundo está en `secrets-manager-ssm.md`.
> Aquí lo vemos desde **operations**.

## 7.1 Qué es Parameter Store
- Almacén centralizado de:
  - parámetros
  - configuración
  - secretos simples

Tipos:
- String
- StringList
- SecureString (KMS)

📌 En examen:
- config simple → Parameter Store
- secretos críticos/rotación → Secrets Manager

---

# 8) Inventory y Compliance

## 8.1 Inventory
Recoge metadata:
- software instalado
- versiones
- configuración

Útil para:
- auditoría
- compliance
- troubleshooting

---

## 8.2 Compliance
Muestra:
- estado de parches
- estado de configuraciones

📌 En examen:
> *“Check if instances comply with policies”* → **SSM Compliance**

---

# 9) Integración con otros servicios (visión arquitectura)

- **IAM** → controla quién puede usar SSM
- **CloudTrail** → audita acciones
- **CloudWatch** → logs y métricas
- **Config** → compliance global
- **VPC Endpoints** → SSM sin internet

📌 Trampa:
- SSM puede funcionar **sin internet** usando VPC endpoints.

---

# 10) Casos de uso típicos de examen

### Caso 1
“Access EC2 securely without opening SSH”

👉 **Session Manager**

---

### Caso 2
“Run the same command on hundreds of instances”

👉 **Run Command**

---

### Caso 3
“Automate patching and track compliance”

👉 **Patch Manager**

---

### Caso 4
“Store configuration parameters securely and centrally”

👉 **Parameter Store**

---

# 11) Trampas típicas SAA-C03

- ❌ Pensar que SSM elimina necesidad de IAM
- ❌ Olvidar el IAM role del instance
- ❌ Abrir SSH cuando SSM ya lo resuelve
- ❌ Confundir SSM con CloudWatch

---

# 12) Tabla de decisiones rápidas

| Necesidad | SSM Feature |
|---------|-------------|
| Acceso seguro a EC2 | Session Manager |
| Ejecutar comandos | Run Command |
| Parchar OS | Patch Manager |
| Automatizar tareas | Automation |
| Guardar config | Parameter Store |
| Ver compliance | SSM Compliance |

---

# 13) Labs recomendados (los que sí valen la pena)

## 🧪 Lab 1 — Session Manager (IMPRESCINDIBLE)
1. EC2 sin puerto 22 abierto
2. Role con SSM
3. Acceso vía Session Manager

👉 Te cambia la forma de pensar acceso a EC2.

---

## 🧪 Lab 2 — Run Command
1. Ejecutar comando en varias EC2
2. Ver resultados centralizados

---

## 🧪 Lab 3 — Patch Manager (opcional)
1. Definir patch baseline
2. Ejecutar parcheo
3. Ver compliance

---

## Limpieza
- Terminar EC2
- Revisar roles IAM
- Borrar parámetros de prueba

---

## Mini-resumen final

- Systems Manager = operaciones centralizadas
- Reduce SSH y claves
- Muy alineado con seguridad
- Aparece mucho en SAA-C03 indirectamente

---

## Cierre
Si entiendes Systems Manager, muchas preguntas de **operations + security**
se vuelven obvias: menos acceso directo, más automatización y más control.

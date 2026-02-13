# Operations Labs — AWS SAA-C03 (Free Tier Safe)

> Objetivo: practicar los flujos operativos que realmente importan en el examen SAA-C03.
> Cada lab está escrito paso a paso, con explicaciones, decisiones arquitectónicas, comandos útiles, trampas de examen y limpieza final.
> Región recomendada: **us-east-1** (o la que uses consistentemente).

---

# LAB A — CloudFormation (Stacks, Change Sets, Rollback, Drift)

## Por qué hacerlo
CloudFormation no es solo "infra-as-code": el examen pide entender **qué pasa al aplicar cambios** (change sets), cómo evitar interrupciones, y cómo responder a errores (rollback, drift).

## Objetivos del lab
- Crear un stack sencillo que cree un bucket S3 + IAM role.
- Actualizar el stack usando un Change Set (simular cambios).
- Forzar un error en una actualización para ver rollback.
- Detectar drift y entender su significado.

## Recursos que crearás
- Stack: `lab-cfn-stack`
- Template inicial: crea un bucket `lab-cfn-bucket-<sufijo>` y un IAM Role `lab-cfn-role` (con permisos mínimos).

> **Nota**: usa nombres únicos (añade timestamp o tu usuario) para evitar choques.

---
### Paso 0 — Template (CloudFormation YAML)
Guarda este archivo localmente como `lab-cfn-template-v1.yaml`:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Lab CFN v1 - bucket + role minimal"
Resources:
  LabBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "lab-cfn-bucket-${AWS::AccountId}-${AWS::Region}"
  LabRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "lab-cfn-role-${AWS::AccountId}"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### Paso 1 — Crear Stack (con consola o CLI)
#### Opción consola:
1. CloudFormation → Create stack → With new resources (standard)
2. Upload template file → `lab-cfn-template-v1.yaml`
3. Stack name: `lab-cfn-stack`
4. Next → Create stack

#### Opción CLI:
```
aws cloudformation create-stack --stack-name lab-cfn-stack --template-body file://lab-cfn-template-v1.yaml --capabilities CAPABILITY_NAMED_IAM
```

### Paso 2 — Verificar recursos creados
- S3 → confirma que existe el bucket.
- IAM → confirma que existe el role `lab-cfn-role`.

### Paso 3 — Preparar Change Set (simular cambio controlado)
Crea un segundo template `lab-cfn-template-v2.yaml` que cambia la propiedad del bucket (por ejemplo, añade un Tag) y añade un recurso que intentes crear con nombre fijo que cause conflicto en update si ya existe (prueba a renombrar role o cambiar RoleName a uno existente para forzar fallo más adelante). Ejemplo sencillo (añadir Tag):

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Lab CFN v2 - bucket tag + same role"
Resources:
  LabBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "lab-cfn-bucket-${AWS::AccountId}-${AWS::Region}"
      Tags:
        - Key: env
          Value: lab
  LabRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "lab-cfn-role-${AWS::AccountId}"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

#### Crear Change Set (consola)
- CloudFormation → tu stack `lab-cfn-stack` → Actions → Create change set for current stack → Upload `lab-cfn-template-v2.yaml` → Create change set
- Revisa los cambios propuestos (Change set → Execution details).

#### Crear Change Set (CLI)
```
aws cloudformation create-change-set --stack-name lab-cfn-stack --change-set-name lab-change-v2 --template-body file://lab-cfn-template-v2.yaml --capabilities CAPABILITY_NAMED_IAM
aws cloudformation describe-change-set --stack-name lab-cfn-stack --change-set-name lab-change-v2
```

### Paso 4 — Ejecutar Change Set (aplicar update)
- Si todo OK, Execute change set → revisa recursos actualizados.
- Observa que solo cambian lo propuesto (tags en el bucket).

### Paso 5 — Forzar error y ver rollback
Modifica el template para que cambie `RoleName` a uno **ya existente** en la cuenta (por ejemplo, usa un nombre que ya exista por fuera del stack). Esto provoca fallo en la creación/actualización. Crea change set y ejecútalo. Verás que CloudFormation intenta aplicar cambios y, al fallar, dependiendo del tipo de change set y configuración, puede intentar rollback (revertir a estado estable anterior).

**Qué mirar:** Events en CloudFormation → verás entradas de error y rollback steps.

### Paso 6 — Drift Detection
- CloudFormation → tu stack → Actions → Detect drift
- Cambia algo manualmente (p. ej. añade un Tag al bucket via consola) y luego re-run drift detection → verás “drifted” sobre el recurso modificado.

---

## Limpieza (CloudFormation)
```
aws cloudformation delete-stack --stack-name lab-cfn-stack
```

CloudFormation borrará recursos creados por el stack (S3 bucket debe estar vacío para borrarse).

---

## Trampas de examen / Puntos a recordar
- Change sets te permiten revisar antes de aplicar. Es la herramienta recomendada para producción.
- Stack updates pueden fallar por resource name collisions o policies que bloquean cambios (IAM role con RoleName puede bloquear).
- Drift detection NO corrige; solo detecta desviaciones.
- Rollback puede dejar recursos en estado ROLLBACK_FAILED si algo impide la reversión.

---

# LAB B — CloudWatch (Métricas, Alarms, Logs, Metric Filters)

## Por qué hacerlo
CloudWatch es el núcleo de observabilidad: el examen pide encadenar Metric → Alarm → Action (SNS/Lambda), entender Logs, Retention y Metric Filters.

## Objetivos del lab
- Crear Log Group y enviar logs manualmente (via CloudWatch console o CLI).
- Crear una Metric Filter que transforme un patrón en métrica personalizada.
- Crear un Alarm basado en la métrica (o sobre CPU de EC2 si tienes EC2).
- Enlazar la Alarm a una acción (SNS topic + email).

## Recursos que crearás
- Log Group: `/aws/lab/logs`
- Metric Filter: `ErrorCountFilter`
- Metric Namespace: `Lab/Custom`
- Alarm: `lab-error-alarm`
- SNS topic opcional: `lab-alerts-topic` (email subscription)

---
### Paso 0 — Crear Log Group
1. CloudWatch → Log groups → Create log group → `/aws/lab/logs`
2. Configure retention (por ejemplo 7 days para el lab)

### Paso 1 — Enviar logs manualmente (CLI) — método rápido
Puedes usar el comando `aws logs put-log-events` pero necesita crear un log stream y secuencia de tokens; para el lab más simple, usa la consola test:

- CloudWatch → Log groups → `/aws/lab/logs` → Create log stream → `stream-1`
- En stream → Actions → Put log events → pega un evento JSON simple:
```json
{ "timestamp": 1620000000000, "message": "ERROR: user failed to authenticate" }
```

### Paso 2 — Crear Metric Filter (extraer número de errores)
- En Log group `/aws/lab/logs` → Create Metric Filter
- Filter pattern: `ERROR`
- Metric namespace: `Lab/Custom`
- Metric name: `ErrorCount`
- Metric value: `1`

Esto convierte cada línea con `ERROR` en una métrica `Lab/Custom -> ErrorCount` con valor 1.

### Paso 3 — Ver la métrica en Metrics
- CloudWatch → Metrics → Browse namespaces → `Lab/Custom` → ver `ErrorCount`
- Si no aparece inmediatamente, espera 1–2 minutos o genera más eventos.

### Paso 4 — Alarm sobre la métrica
- CloudWatch → Alarms → Create alarm → Select metric `Lab/Custom/ErrorCount`
- Define threshold: e.g., whenever `Sum` over 1 minute >= 1
- Action: Create an SNS topic `lab-alerts-topic` and add your email subscription (confirm the email).
- Finish creating alarm.

### Paso 5 — Probar la cadena completa
- Put a new log event containing `ERROR` → la metric filter genera la métrica → CloudWatch la agrega → alarma se dispara → recibirás email.

### Extra — Alarm sobre EC2 CPU
Si tienes una EC2 t3.micro (free tier):
- Select metric: `EC2 -> Per-Instance -> CPUUtilization`
- Create alarm when > 1% for 1 datapoint (solo para demo)
- Action: SNS topic

### Limpieza CloudWatch/SNS
- Delete alarm
- Delete metric filter (desde Log group)
- Delete log group
- Delete SNS topic and subscription

---

## Trampas de examen / Puntos a recordar
- Metric Filters generan métricas personalizadas; se cobran en base a métricas (pero para lab tiny está OK).
- Alarms se basan en agregaciones (Sum/Avg) y periodo. Entiende el período y estadística.
- Logs retention default es 'Never expire' — para labs pon 7 days y luego borra.

---

# LAB C — CloudTrail (Auditoría y Trails multi-region)

## Por qué hacerlo
CloudTrail es la fuente de verdad para auditoría. En el examen piden saber diferencias entre CloudTrail y CloudWatch, y cómo centralizar logs multi-account/ multi-region.

## Objetivos del lab
- Crear un Trail multi-region que entregue eventos a un S3 bucket.
- Generar eventos (crear / delete IAM user) y verificar que aparecen en S3 (o Event history).
- Ver la integración opcional con CloudWatch Logs (no obligatorio para el lab).

## Recursos que crearás
- S3 bucket: `lab-cloudtrail-logs-<sufijo>`
- Trail: `lab-multi-trail` (multi-region)

---
### Paso 1 — Crear S3 bucket para logs
- S3 → Create bucket → `lab-cloudtrail-logs-<sufijo>` (bloquea acceso público)

### Paso 2 — Crear Trail (multi-region)
1. CloudTrail → Trails → Create trail
2. Trail name: `lab-multi-trail`
3. Apply trail to all regions: ✅
4. Store logs in S3: selecciona `lab-cloudtrail-logs-<sufijo>`
5. Create

### Paso 3 — Generar eventos y verificar
- Realiza acciones: crear un IAM user o crear/terminate una EC2 (acciones pequeñas).
- CloudTrail → Event history → busca el evento (filtra por Event name).
- En S3 → abre prefijo `AWSLogs/<account-id>/CloudTrail/...` y busca archivos .json.gz con eventos recientes.
- Abre (descarga y descomprime) y revisa el contenido (verás `eventName`, `userIdentity`, `eventTime`).

### Paso 4 — (Opcional) Enviar CloudTrail a CloudWatch Logs
- In CloudTrail → tu Trail → Edit → CloudWatch Logs → Configure
- Debes crear un IAM role `CloudTrail_CloudWatchLogs_Role` con permissions para `logs:PutLogEvents` y `logs:CreateLogStream` (consola te ofrece crear el role automáticamente).
- Esto permite búsquedas en tiempo real y crear metric filters sobre eventos CloudTrail.

### Limpieza CloudTrail
- Delete trail (esto solo detiene la captura; los objetos en S3 permanecen y debes borrarlos manualmente)
- Borrar S3 bucket (vaciar primero)

---

## Trampas de examen / Puntos a recordar
- CloudTrail = auditoría (API activity). CloudWatch = métricas/logs operacionales. No son lo mismo.
- Multi-region trail cubre todas las regiones; recomendable para cumplir requisitos de auditoría.
- Trail entrega a S3 y (opcionalmente) a CloudWatch Logs y to CloudWatch Events/EventBridge.

---

# LAB D — AWS Systems Manager (Session Manager, Run Command, Inventory)

## Por qué hacerlo
SSM reduces the need for SSH and is key in secure operations. The exam values understanding Session Manager without opening ports, and Patch Manager / Run Command as core ops tools.

## Objetivos del lab
- Probar Session Manager para conectarte a una EC2 sin ssh.
- Usar Run Command para ejecutar un comando simple en la instancia.
- Revisar Inventory básico.
- (Opcional) Mostrar Patch Manager y cómo planificar parches (sin ejecutar parches masivos en el lab).

## Recursos que crearás (o reutilizarás)
- EC2 t3.micro con Amazon Linux 2 (Free Tier OK)
- IAM role: `AmazonSSMManagedInstanceCore` asociado a la instancia

---
### Paso 0 — Lanzar EC2 (si no tienes)
- EC2 → Launch instance → Amazon Linux 2 AMI → t3.micro → Key pair (opcional)
- In Network settings: puedes dejar en default VPC and subnet
- En IAM role: crea/selecciona role con `AmazonSSMManagedInstanceCore`

**Nota:** SSM Agent viene instalado en Amazon Linux 2 y Ubuntu LTS imágenes modernas. Si usas otra AMI, instala el agent.

### Paso 1 — Ver que la instancia está “Managed Instance”
- Systems Manager → Fleet Manager / Managed Instances → deberías ver la instancia con status `Online`.
- Si no aparece: revisa IAM role y que la instancia tenga conectividad saliente (NAT/IGW) para Reach SSM endpoints.

### Paso 2 — Session Manager (consola)
- Systems Manager → Session Manager → Start session → selecciona la instancia → Start session
- Tendrás una shell en el navegador conectada sin ssh ni puertos abiertos.

### Paso 3 — Run Command (ejecutar comando)
- Systems Manager → Run Command → Run a command
- Document: `AWS-RunShellScript`
- Targets: selecciona la instancia
- Commands: `echo "hello from ssm" > /tmp/ssm-test.txt`
- Run
- Ver output (Command history) y confirma el archivo existe via Session Manager shell: `cat /tmp/ssm-test.txt`

### Paso 4 — Inventory (opcional)
- Systems Manager → Inventory → Configure inventory (si no se configura, habilitar)
- Recolecta información básica de paquetes/OS.

### Paso 5 — Patch Manager (informativo)
- Patch Manager permite definir Patch Baselines y Programar ventanas de mantenimiento.
- No ejecutes parches masivos durante el lab; configurar una baseline y política de mantenimiento es suficiente para entender su uso en SAA.

### Limpieza (Systems Manager / EC2)
- Termina la instancia EC2 (Actions → Instance state → Terminate)
- El role `lab-ec2-ssm-role` puedes borrarlo si fue creado solo para el lab

---

## Trampas de examen / Puntos a recordar
- Session Manager no necesita puerto SSH y funciona sobre HTTPS hacia los endpoints de SSM (por lo que VPC sin internet necesita endpoints VPC para SSM).
- Para que SSM funcione necesitas: SSM Agent + IAM role con `AmazonSSMManagedInstanceCore` + conectividad hacia SSM endpoints (internet o VPC endpoint).
- Patch Manager y Automation son herramientas para operaciones a escala.

---

# Limpieza final (Checklist)
Para no dejar recursos que cobren:

- CloudFormation: delete stack (`lab-cfn-stack`) → luego borrar objetos S3 si quedan.
- CloudWatch: delete alarms, log groups, metric filters, SNS topics.
- CloudTrail: delete trail, empty and delete S3 bucket used for logs.
- Systems Manager: terminate EC2 instances, delete IAM roles created para el lab.

---

# Preguntas tipo examen (práctica rápida)
1. ¿Por qué usar Change Sets antes de Update Stack? ¿Qué problema resuelven?
2. ¿Qué diferencia hay entre un CloudWatch metric filter y EventBridge?
3. ¿Por qué Session Manager es más seguro que SSH directo? ¿Qué debes configurar para que funcione en una VPC privada?
4. Si un stack falla y entra en `ROLLBACK_FAILED`, ¿qué acciones tomarías?

---

# Consejos finales para el examen
- Practica workflows: Metric → Alarm → SNS → Action.
- Entiende la distinción entre auditoría (CloudTrail) y métricas (CloudWatch).
- Automate safe updates with Change Sets and IAM least privilege.
- Consigue experiencia práctica con Session Manager: es común en soluciones seguras.

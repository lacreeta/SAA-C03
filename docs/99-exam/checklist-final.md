# Checklist Final — AWS SAA-C03

> Este documento es **tu lista de verificación final** antes del examen.
> No explica en profundidad: **comprueba que sabes decidir**.
> Si puedes responder mentalmente a todo esto, estás listo.

---

## 1) Identidad y Acceso (IAM)

- [ ] Sé explicar **Shared Responsibility** sin dudar
- [ ] Sé cuándo usar **IAM User vs Role**
- [ ] Sé que **Roles > Users** para servicios
- [ ] Sé qué es **STS / AssumeRole**
- [ ] Sé la diferencia entre:
  - identity-based policy
  - resource-based policy
- [ ] Sé que **Explicit DENY siempre gana**
- [ ] Sé que IAM es **global**

---

## 2) Core Compute & Storage (EC2 / S3 / EBS)

### EC2
- [ ] Sé cuándo usar **EC2 vs Lambda vs ECS/Fargate**
- [ ] Sé que EC2 da **control del sistema operativo**
- [ ] Sé que una EC2:
  - puede tener IP pública o privada
  - **no sale a internet** desde subnet privada sin NAT
- [ ] Sé que:
  - **Security Group = stateful**
  - **NACL = stateless**
- [ ] Sé que **EBS está ligado a una AZ**
- [ ] Sé que **Snapshot de EBS = incremental**
- [ ] Sé que:
  - cambiar tipo de instancia **no recrea** la EC2
  - cambiar AZ **sí obliga a recrear**

### S3
- [ ] Sé que S3 es **regional y altamente duradero**
- [ ] Sé que S3 **NO es un filesystem**
- [ ] Sé cuándo usar:
  - Standard
  - Infrequent Access
  - Glacier
- [ ] Sé que:
  - **Versioning** protege contra borrados
  - **Lifecycle** = optimización de costes
- [ ] Sé que el acceso se controla con:
  - IAM policy
  - Bucket policy
- [ ] Sé que **S3 puede ser privado sin IP pública**
- [ ] Sé que **S3 + CloudFront = baja latencia global**

### Trampas de examen
- [ ] S3 ≠ EBS ≠ EFS
- [ ] EC2 en subnet pública **sin IP pública** no es accesible
- [ ] Snapshot ≠ Backup multi-región (hasta que se copie)

---

## 3) Networking (VPC, Routing, Conectividad)

### VPC y Subnets
- [ ] Sé que una **VPC es regional**
- [ ] Sé que una **subnet pertenece a una AZ**
- [ ] Sé que una subnet es pública **solo si**:
  - tiene `0.0.0.0/0 → Internet Gateway`
- [ ] Sé que **auto-assign public IP ≠ subnet pública**

### Routing
- [ ] Sé que:
  - Internet Gateway permite internet **si hay ruta**
  - NAT Gateway permite **solo salida** desde subnets privadas
- [ ] Sé que:
  - NAT Gateway vive en **subnet pública**
  - las subnets privadas rutean hacia él
- [ ] Sé que **sin route table correcta no hay conectividad**

### Conectividad entre VPCs
- [ ] Sé cuándo usar:
  - **VPC Peering** (simple, no transitivo)
  - **Transit Gateway** (hub-and-spoke)
- [ ] Sé que:
  - VPC Peering **no es transitivo**
  - Transit Gateway **sí permite segmentación y control**

### Conectividad híbrida
- [ ] Sé la diferencia entre:
  - **Site-to-Site VPN** (rápido, cifrado, sobre internet)
  - **Direct Connect** (privado, estable, caro)
- [ ] Sé que:
  - VPN suele usarse como **backup de DX**
  - Direct Connect **no cifra por defecto**

### Trampas de examen
- [ ] Una VPC sin IGW **nunca** tiene acceso a internet
- [ ] Un IGW sin ruta en la route table **no se usa**
- [ ] Un NACL puede bloquear tráfico aunque el SG permita
- [ ] Las route tables deciden **el camino**, no los permisos


## 4) Seguridad de Datos (KMS / Secrets)

- [ ] Sé cuándo usar **AWS-managed key**
- [ ] Sé cuándo usar **Customer-managed CMK**
- [ ] Sé qué es **SSE-KMS**
- [ ] Sé que **KMS no guarda datos**
- [ ] Sé cuándo usar:
  - Secrets Manager
  - SSM Parameter Store
- [ ] Sé que **rotación automática = Secrets Manager**
- [ ] Sé que **IAM controla acceso al secreto**

---

## 5) Red y Protección

- [ ] Sé la diferencia entre **SG y NACL**
- [ ] Sé que **WAF = L7 (SQLi/XSS)**
- [ ] Sé que **Shield = DDoS**
- [ ] Sé dónde se asocia WAF (ALB / API GW / CloudFront)
- [ ] Sé cuándo usar **rate-based rules**

---

## 6) Observabilidad y Auditoría

- [ ] Sé que:
  - CloudWatch = métricas/logs
  - CloudTrail = auditoría
  - Config = compliance
- [ ] Sé qué es un **multi-region trail**
- [ ] Sé cuándo usar **CloudWatch Alarm**
- [ ] Sé crear el flujo mental:
  - Metric → Alarm → Action

---

## 7) Alta Disponibilidad y DR

- [ ] Sé que **HA ≠ DR**
- [ ] Sé que **HA intra-región = ALB + Multi-AZ**
- [ ] Sé que **Route 53 Failover = DR**
- [ ] Sé elegir entre:
  - Backup & Restore
  - Pilot Light
  - Warm Standby
  - Active-Active
- [ ] Sé qué son **RTO y RPO**

---

## 8) Serverless

- [ ] Sé cuándo usar **Lambda**
- [ ] Sé cuándo NO usar Lambda (15 min)
- [ ] Sé diferenciar:
  - SNS
  - SQS
  - EventBridge
  - Step Functions
- [ ] Sé que:
  - fan-out = SNS (+ SQS)
  - buffer = SQS
  - eventos AWS = EventBridge
  - workflows = Step Functions

---

## 9) Infrastructure as Code

- [ ] Sé qué es **CloudFormation**
- [ ] Sé qué es un **stack**
- [ ] Sé qué es un **change set**
- [ ] Sé qué es **drift**
- [ ] Sé cuándo usar **nested stacks**
- [ ] Sé por qué separar **data vs compute**

---

## 10) Operaciones

- [ ] Sé qué hace **Systems Manager**
- [ ] Sé qué es **Session Manager**
- [ ] Sé qué es **Patch Manager**
- [ ] Sé que SSM reduce SSH
- [ ] Sé que SSM requiere:
  - agent
  - IAM role
  - conectividad

---

## 11) Bases de Datos (recordatorio)

- [ ] Sé que **Multi-AZ = HA**
- [ ] Sé que **Read Replica = scaling**
- [ ] Sé cuándo usar:
  - RDS
  - DynamoDB
  - Aurora
- [ ] Sé que backups ≠ HA

---

## 12) Regla final de oro

> Si el enunciado habla de **menos gestión, más automático** → piensa **servicios gestionados / serverless**  
> Si habla de **control fino** → piensa **IAM, KMS, Config**

---

## Cierre
Si esta checklist te sale fluida, **estás preparado para aprobar**.

# Subnets (SAA-C03)

Apuntes orientados a **decisiones de arquitectura** y **trampas de examen**.  
(No es “qué es una subnet”, sino **cómo se usa bien**.)

---

## Qué debes saber sí o sí (examen)

### Alcance y constraints
- Una **subnet vive en 1 sola AZ** (zonal).
- Una **VPC es regional**.
- Alta disponibilidad en AWS = **duplicar subnets por AZ** (misma capa en múltiples AZs).

📌 Implicación típica:
- “Multi-AZ” en RDS/ALB/ECS/EC2 implica **subnets en al menos 2 AZs**.

---

## Pública vs privada (la regla que más puntúa)

### Regla de oro
Una subnet es **pública** solo si su **Route Table** tiene:
- `0.0.0.0/0 -> Internet Gateway (IGW)`

Si NO tiene esa ruta, es **privada**.

### Trampas frecuentes
- “Auto-assign public IPv4” ≠ subnet pública.  
  (Eso solo afecta a si las instancias lanzadas reciben IP pública, no al routing.)
- Tener acceso a **S3** desde una subnet privada puede ser por **VPC Endpoint**, no por NAT.

---

## Diseño típico por capas (lo que AWS espera en examen)

Por AZ, lo más común:

### Public Subnet (edge)
- ALB (internet-facing)
- NAT Gateway (si lo usas)
- Bastion host (cada vez menos común, mejor SSM)

### Private Subnet (app)
- EC2 / ECS / EKS workers
- Internal ALB
- Servicios que necesitan salir a internet *solo outbound* (via NAT si aplica)

### Private Subnet (data)
- RDS / ElastiCache
- Sin rutas a IGW
- Con NACL más restrictiva (si el caso lo justifica)

**Patrón clave**: misma estructura replicada en 2+ AZs.

---

## Route tables: cómo pensar como arquitecto

### 1) No “una route table para todo”
En arquitectura real y en examen:
- Public subnets → route table con `0.0.0.0/0 -> IGW`
- Private app subnets → `0.0.0.0/0 -> NAT` (si necesitan outbound a internet)
- Private data subnets → normalmente sin `0.0.0.0/0` (o muy controlado)

### 2) VPC Endpoints cambian el juego
- **S3/DynamoDB Gateway Endpoint**: se asocia a route tables privadas.
- Permite acceder a esos servicios **sin NAT**.

📌 Pregunta típica:
“Reducir coste de NAT por tráfico a S3” → **S3 Gateway Endpoint**.

---

## Seguridad: SG vs NACL (subnet en el centro)

### Security Groups (SG)
- Stateful
- A nivel de ENI/instancia
- “Permitir” es suficiente para retorno

### Network ACLs (NACL)
- Stateless
- A nivel de subnet
- Reglas inbound y outbound separadas (si permites ida, permite vuelta)

📌 Trampa:
“SG permite, pero no funciona” → revisa NACL / routing.

---

## Subnets y servicios gestionados (puntos de examen)

### ALB
- Un **internet-facing ALB** necesita estar en **subnets públicas** (2+ AZ).
- Un **internal ALB** va en **subnets privadas**.

### NAT Gateway
- NAT Gateway se crea en una **subnet pública** (porque necesita IGW).
- Las subnets privadas enrutan `0.0.0.0/0` hacia el NAT.

### RDS
- Multi-AZ requiere subnets en 2+ AZ (en un DB Subnet Group).
- Para “privado de verdad”: sin ruta a IGW, y acceso solo desde app subnets.

### VPC Endpoints (recordatorio)
- Subnet privada puede hablar con S3/SSM/etc. sin internet.
- Esto es muy “compliance-friendly”.

---

## Errores típicos del examen (y cómo detectarlos)

1) **Creen que IP pública hace pública la subnet**
- No. La subnet es pública si su route table va al IGW.

2) **Confunden “subnet privada” con “sin salida”**
- Privada solo significa “sin ruta al IGW”.
- Puede tener salida via NAT o via VPC Endpoint.

3) **Olvidan el requisito multi-AZ**
- Si dicen “highly available”, “fault tolerant”, “multi-AZ”:
  - ALB: 2+ subnets en 2+ AZ
  - App tier: 2+ subnets en 2+ AZ
  - DB: subnet group en 2+ AZ

4) **NAT mal colocado**
- NAT en subnet privada = mal (no funciona como debe).
- NAT siempre en subnet pública.

---

## Checklist mental SAA-C03

- ¿Internet inbound? → Public subnet + IGW + (ALB) + SG correcto
- ¿Outbound only desde privado? → Private subnet + NAT (si es a internet) o Endpoint (si es servicio AWS)
- ¿Multi-AZ? → Subnets duplicadas en ≥2 AZs
- ¿DB segura? → Private data subnets, sin IGW, acceso solo desde app


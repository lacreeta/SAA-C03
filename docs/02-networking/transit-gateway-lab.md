# transitgateway-lab.md — Labs de Transit Gateway (Lab 1–3)

**Fecha:** 30-01-2026  
**Contexto:** AWS SAA-C03 (networking / arquitectura)  
**Objetivo global:** pasar de conectividad simple (2 VPCs) → hub & spoke (3 VPCs) → **segmentación** (quién habla con quién) usando **AWS Transit Gateway (TGW)**.

---

## Índice
1. [Modelo mental (antes de tocar nada)](#modelo-mental-antes-de-tocar-nada)
2. [Pre-requisitos y diseño base](#pre-requisitos-y-diseño-base)
3. [UI notes 2026: dónde están los Attachments](#ui-notes-2026-dónde-están-los-attachments)
4. [Lab 1 — TGW mínimo viable (VPC-A ↔ VPC-B)](#lab-1--tgw-mínimo-viable-vpc-a--vpc-b)
5. [Lab 2 — Hub & Spoke + Bastion (Prod, Dev, Shared)](#lab-2--hub--spoke--bastion-prod-dev-shared)
6. [Lab 3 — Segmentación con TGW Route Tables (3 RT-TGW)](#lab-3--segmentación-con-tgw-route-tables-3-rt-tgw)
7. [Troubleshooting real (lo que te pasó y cómo se detecta)](#troubleshooting-real-lo-que-te-pasó-y-cómo-se-detecta)
8. [Costes y limpieza (para no pagar de más)](#costes-y-limpieza-para-no-pagar-de-más)
9. [Takeaways de examen SAA-C03](#takeaways-de-examen-saa-c03)

---

## Modelo mental (antes de tocar nada)

### Qué es TGW (sin humo)
**Transit Gateway = router regional gestionado** que conecta:
- múltiples VPCs
- on-prem (VPN / Direct Connect)
- con **routing transitivo** (transitive routing)

### Dos capas de routing (la clave)
Para que una VPC hable con otra por TGW necesitas **DOS** cosas:

1) **Route tables de las subnets (en la VPC)**  
   Le dicen a la VPC:  
   > “Si el destino es el CIDR de la otra VPC → envíalo al TGW”.

2) **Route table(s) del TGW**  
   Le dicen al TGW:  
   > “Si el destino es el CIDR X → mándalo por el attachment correcto”.

**Attachment ≠ ruta**.  
Un attachment es “el cable enchufado”. La ruta es “por dónde circula el tráfico”.

### Association vs Propagation (TGW)
- **Association** = “por qué TGW route table entra el tráfico de este attachment”.  
- **Propagation** = “qué CIDR anuncia (enseña) este attachment a una TGW route table”.

Regla importantísima:
- Un attachment puede estar **asociado a 1 sola RT-TGW**.
- Un attachment puede **propagar a varias RT-TGW**.

---

## Pre-requisitos y diseño base

### Requisitos mínimos para labs
- Todo en **una sola región** (usa la misma donde vienes trabajando).
- CIDRs **no solapados**.
- Una EC2 por VPC para probar (ping / ssh).
- Security Groups razonables (no el foco, pero necesarios para test).

### CIDRs usados (ejemplo)
**Lab 1**
- VPC-A: `10.0.0.0/16`
- VPC-B: `10.1.0.0/16`

**Lab 2 y 3 (hub & spoke)**
- VPC-Prod (bastion): `10.0.0.0/16`
- VPC-Dev: `10.1.0.0/16`
- VPC-Shared: `10.2.0.0/16`

> Si tú usaste otros, adapta los CIDRs en las rutas/SG.

---

# Lab 1 — TGW mínimo viable (VPC-A ↔ VPC-B)

## Objetivo
Conectar **dos VPCs** usando un solo Transit Gateway y validar:
- routing VPC → TGW
- routing TGW → VPC
- conectividad por IP privada (ping)

## Arquitectura
```
EC2-A (VPC-A) → Subnet RT-A → TGW → TGW RT → Attachment-B → Subnet RT-B → EC2-B
```

## Pasos (operativos + por qué)

### 1) Crear TGW
- Creas el “router regional”.
- Aún no conecta nada.

### 2) Crear 2 Attachments (1 por VPC)
- Attachment VPC-A → TGW
- Attachment VPC-B → TGW

> Attachment = “la VPC queda conectable al TGW”.  
> Todavía **no** hay conectividad hasta tocar rutas.

### 3) Rutas en las subnets (lado VPC)
En la route table de la subnet donde vive EC2-A:
- `10.1.0.0/16 → TGW`

En la route table de la subnet donde vive EC2-B:
- `10.0.0.0/16 → TGW`

### 4) Rutas en la TGW route table (lado TGW)
Asegúrate de que la TGW route table “aprende” ambos CIDRs:
- `10.0.0.0/16 → attachment de VPC-A`
- `10.1.0.0/16 → attachment de VPC-B`

Esto puede venir por **propagation** (automático) o requerir habilitar propagations.

### 5) Security Groups (para poder probar)
Para ping A→B:
- En SG de **EC2-B**: permitir inbound ICMP desde `10.0.0.0/16`

Para ping bidireccional (A↔B):
- También permitir inbound ICMP en **EC2-A** desde `10.1.0.0/16`

> Los SG son **stateful**: no necesitas duplicar outbound ICMP si mantienes “allow all outbound”.

### 6) Validación
Desde EC2-A:
- `ping <IP_privada_EC2-B>`

---

# Lab 2 — Hub & Spoke + Bastion (Prod, Dev, Shared)

## Objetivo
Montar un **hub & spoke** con TGW:
- 3 VPCs conectadas al mismo TGW
- **Prod** actúa como **bastion (jump host)** público
- Dev y Shared son privadas (sin IP pública)
- Validar conectividad:
  - ping “any ↔ any” (si SG lo permite)
  - ssh desde bastion hacia privadas (saltos)

## Arquitectura
```
Laptop → SSH → Bastion (Prod, público)
                 |
                 v
               TGW
             /     \
          Dev     Shared
```

## Subnets públicas vs privadas (criterio)
- Para TGW **da igual** técnicamente.
- Para “diseño limpio”:
  - Bastion en subnet **pública** (IGW + IP pública)
  - Dev/Shared en subnets **privadas** (sin IP pública)

## Steps clave

### 1) Attachments
- 1 attachment por VPC:
  - Prod → TGW
  - Dev → TGW
  - Shared → TGW

### 2) Rutas en subnets (VPC route tables)
En **cada** VPC, en la route table de la subnet donde está su EC2:
- Rutas hacia los CIDRs de las otras VPCs → TGW

Ejemplo:
- En Prod:
  - `10.1.0.0/16 → TGW`
  - `10.2.0.0/16 → TGW`
- En Dev:
  - `10.0.0.0/16 → TGW`
  - `10.2.0.0/16 → TGW`
- En Shared:
  - `10.0.0.0/16 → TGW`
  - `10.1.0.0/16 → TGW`

### 3) TGW route table (modo simple)
En Lab 2, lo más simple es:
- **una sola** TGW route table
- todos los attachments asociados + propagando

Resultado: **any ↔ any** (a nivel de routing).

### 4) Bastion / Jump host (cómo lo usaste)
- Te conectas desde el portátil al bastion por su **IP pública**.
- Desde bastion, accedes a Dev/Shared por **IP privada**.
- Desde dentro de AWS, usar IP pública para saltos internos suele fallar (intenta “salir a internet”).

#### Regla mental
- **Desde fuera** → IP pública (bastion).
- **Desde dentro** → IP privada (cualquier VPC).

### 5) SGs recomendados (lab)
**Bastion (Prod)**
- Inbound: SSH desde tu IP pública (o temporalmente 0.0.0.0/0 en lab).
- Outbound: allow all.

**Dev/Shared**
- Inbound SSH desde **CIDR de Prod** (por TGW no puedes “referenciar SG” cross-VPC como source; usa CIDR).
  - ejemplo: `10.0.0.0/16 → SSH 22`
- Inbound ICMP desde CIDRs remotos si quieres ping.
- Outbound: allow all.

### 6) Copiar claves y usar ssh desde bastion
En Windows PowerShell usaste `scp` y `ssh`.

**Copiar fichero al bastion (siempre con destino):**
```powershell
scp -i .\prueba2.pem .\prueba2.pem ec2-user@<IP_PUBLICA_BASTION>:/home/ec2-user/
```

**Permisos de la key en Linux (si no, SSH la ignora):**
```bash
chmod 400 prueba2.pem
# o chmod 600 prueba2.pem
```

**SSH desde bastion a instancia privada (por IP privada):**
```bash
ssh -i prueba2.pem ec2-user@10.2.0.200
```

> Te salió el error “UNPROTECTED PRIVATE KEY FILE” (0664). Se arregla con chmod 400/600.

### 7) IP pública dinámica vs Elastic IP (EIP)
Te diste cuenta de lo importante:
- SG **no** influye en que haya IP pública.
- Depende de la subnet (pública) y del flag **Auto-assign public IPv4**.

**Para usar IP pública dinámica:**
- No uses EIP.
- Asegura que la subnet del bastion tiene:
  - `0.0.0.0/0 → IGW`
  - Auto-assign public IPv4 = **Enabled**
- Si una instancia se creó sin IP pública, normalmente hay que **recrear** para que AWS asigne pública dinámica al launch.

---

# Lab 3 — Segmentación con TGW Route Tables (3 RT-TGW)

## Objetivo
Aplicar una política de conectividad típica de “shared services”:

- Dev ↔ Shared ✅
- Prod ↔ Shared ✅
- Dev ↔ Prod ❌
- (Y que Shared pueda iniciar tráfico hacia ambos)

La solución final que encontré y que funciona bien:
> **3 TGW Route Tables**, una por cada “punto de entrada” (attachment asociado).

## Por qué hacen falta 3 RT-TGW 
Porque el TGW enruta **según la route table asociada al attachment de origen**.

Si Shared solo puede estar asociado a 1 RT-TGW, y quieres:
- que Shared hable con Dev
- y también con Prod
- manteniendo Dev y Prod aislados

Entonces Shared necesita “su” tabla (RT-Shared) que conozca ambos, mientras Dev y Prod entran por tablas que **no** se conozcan entre sí.

## Diseño final (recomendado)

### TGW Route Tables
- **RT-Prod**: tabla de entrada para Prod
- **RT-Dev**: tabla de entrada para Dev
- **RT-Shared**: tabla de entrada para Shared

### Associations (quién entra por qué tabla)
- **Prod attachment → RT-Prod**
- **Dev attachment → RT-Dev**
- **Shared attachment → RT-Shared**

> Esta es la parte clave: **RT por attachment** cuando quieres restricciones.

### Propagations / Routes esperadas (qué conoce cada tabla)
#### RT-Prod (Prod solo ve Shared)
- Conoce: `10.2.0.0/16` (Shared)
- NO conoce: Dev

Resultado:
- Prod → Shared ✅
- Prod → Dev ❌

#### RT-Dev (Dev solo ve Shared)
- Conoce: `10.2.0.0/16` (Shared)
- NO conoce: Prod

Resultado:
- Dev → Shared ✅
- Dev → Prod ❌

#### RT-Shared (Shared ve a ambos)
- Conoce: `10.0.0.0/16` (Prod) y `10.1.0.0/16` (Dev)
- (y Shared local)

Resultado:
- Shared → Prod ✅
- Shared → Dev ✅

### Importante: las subnets siguen necesitando rutas
Aunque segmentes con TGW route tables, **las route tables de subnets** siguen teniendo que mandar CIDRs remotos al TGW.
Ejemplo real (lo que verificaste):
- En la subnet de Prod:
  - `10.1.0.0/16 → TGW`
  - `10.2.0.0/16 → TGW`
  - `0.0.0.0/0 → IGW` (por ser pública)
- En la subnet de Shared:
  - `10.0.0.0/16 → TGW`
  - `10.1.0.0/16 → TGW`
  - `10.2.0.0/16 → local`

## Validación (tests esperados)
Haz pings por IP privada:

1) Desde **Prod**:
- ping Shared ✅
- ping Dev ❌

2) Desde **Dev**:
- ping Shared ✅
- ping Prod ❌

3) Desde **Shared**:
- ping Prod ✅
- ping Dev ✅

> Para SSH: bastion (Prod) → Dev/Shared por IP privada, si SG de destino permite SSH desde CIDR de Prod.

---

## Costes y limpieza (para no pagar de más)

### Qué conservar para seguir estudiando
- VPCs, subnets, route tables, SGs: normalmente coste 0.
- TGW y attachments: tienen coste, pero en lab y poco tráfico suele ser controlado.

### Qué parar / borrar
- **EC2 en running**: parar cuando no uses.
- **Elastic IP**: si la dejas asignada a una instancia parada o no asociada, puede costar.  

### Recomendación práctica
- Mantén infraestructura lógica (VPC/TGW) si vas a seguir con labs.
- Para EC2: *start/stop* según necesidad.

---

## Takeaways de examen SAA-C03

- “Centralized connectivity”, “hub-and-spoke”, “transitive routing” → **Transit Gateway**.
- TGW **NO** da acceso a internet (eso es IGW/NAT).
- **Peering no es transitive**, TGW sí.
- **Segmentación**: usa múltiples **TGW route tables** + asociaciones por attachment.
- **SG stateful vs NACL stateless**: el examen ama esta diferencia.
- Desde dentro de AWS: **IPs privadas** para tráfico interno.
- Bastion/jump host: patrón típico para acceder a instancias privadas.

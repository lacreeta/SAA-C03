# VPC Endpoints — Labs Completos

## Fecha
29-01-2026

---

## Objetivo global

Diseñar y entender una arquitectura **100 % privada y segura** donde una EC2 en subnet privada:

- Accede a **S3** sin usar internet
- Se administra vía **SSM**:
  - Sin SSH
  - Sin bastion
  - Sin IP pública
- **NO** tiene salida a internet (si no se desea)

Demostrando claramente que:

- 🔌 **La conectividad** la proporcionan los **VPC Endpoints**
- 🔐 **Los permisos** los controla **IAM**

---

# Lab 0 — VPC privada sin internet (baseline)

## Objetivo

Crear una VPC **totalmente aislada de internet**, con una EC2 en subnet privada **sin salida**.  
Este lab es la base para entender **por qué necesitamos VPC Endpoints**.

---

## Pasos

### 1. Crear VPC

- Nombre: `lab-vpc-endpoints`
- IPv4 CIDR: `10.0.0.0/16`
- Tenancy: default

---

### 2. Crear subnet privada

- VPC: `lab-vpc-endpoints`
- AZ: por ejemplo `us-east-1a`
- Nombre: `private-subnet-a`
- CIDR: `10.0.1.0/24`
- Auto-assign public IPv4: **DISABLED**

---

### 3. Route Table privada

- Crear: `rt-private`
- VPC: `lab-vpc-endpoints`
- Rutas:
  - `10.0.0.0/16 → local`
- Asociar `private-subnet-a` a `rt-private`

> Opcional: hacerla *main*, pero lo importante es la asociación.

---

### 4. NO crear conectividad a internet

- ❌ Sin Internet Gateway
- ❌ Sin NAT Gateway

---

### 5. Security Group para la EC2

- Nombre: `sg-private-ec2`
- VPC: `lab-vpc-endpoints`
- Inbound: vacío
- Outbound:
  - `All traffic → 0.0.0.0/0`

---

### 6. Crear EC2 en subnet privada

- AMI: Amazon Linux
- Tipo: `t3.micro`
- VPC: `lab-vpc-endpoints`
- Subnet: `private-subnet-a`
- IP pública: **Disabled**
- Security Group: `sg-private-ec2`
- IAM role: ninguno (por ahora)

---

## Conclusión Lab 0

- La subnet es **privada** porque su route table **NO** tiene:
  - `0.0.0.0/0 → IGW`
- La EC2 no tiene:
  - IP pública
  - Ruta a internet

**Resultado:**  
❌ No hay acceso a internet  
❌ No hay acceso a S3  

👉 AWS **no regala conectividad** por estar “en la nube”.

---

# Lab 1 — Gateway Endpoint para S3 (sin NAT ni IGW)

## Objetivo

Permitir que la EC2 privada acceda a **S3** sin usar:

- Internet Gateway
- NAT Gateway
- IP pública

Usando **S3 Gateway VPC Endpoint**.

---

## Pasos

### 1. Estado previo

- EC2 en subnet privada
- Route table `rt-private`:
  - `10.0.0.0/16 → local`

---

### 2. Crear S3 Gateway Endpoint

- VPC → Endpoints → Create endpoint
- Service category: `AWS services`
- Service: `com.amazonaws.<region>.s3`
- Type: **Gateway**
- VPC: `lab-vpc-endpoints`
- Route tables: marcar `rt-private`
- Policy: **Full access** (para el lab)

---

### 3. Verificar route table

En `rt-private` aparece:

- `pl-XXXXXXX (S3 prefix list) → vpce-XXXXXXXX`

Y sigue **SIN**:

- `0.0.0.0/0 → IGW`
- `0.0.0.0/0 → NAT`

---

## Conclusión Lab 1

- La EC2 puede acceder a S3 **sin internet** (si IAM lo permite).
- El tráfico va **dentro de AWS**, no sale a internet.
- El Gateway Endpoint:
  - ❌ No crea ENIs
  - ❌ No usa Security Groups
  - ✅ Funciona solo vía **route table**

**Idea clave SAA-C03:**  
> Private access to S3 without internet → **S3 Gateway Endpoint**

---

# Lab 2 — NAT vs Endpoint (longest prefix match)

> Lab conceptual (realizado y luego limpiado)

## Objetivo

Demostrar que:

- Aunque exista un **NAT Gateway**
- El tráfico a **S3** sigue usando el **Gateway Endpoint**
- AWS aplica **longest prefix match**

---

## Resultado conceptual

- Tráfico a S3:
  - Coincide con prefix list → **Gateway Endpoint**
- Tráfico a internet:
  - `0.0.0.0/0` → **NAT Gateway**

**Idea clave:**  
> El Endpoint **no se ignora** aunque exista NAT.

---

# Lab 3 — Interface Endpoints + SSM (sin SSH, sin internet)

## Objetivo

Gestionar la EC2 privada usando **SSM**:

- Sin IP pública
- Sin SSH
- Sin bastion
- Sin NAT ni IGW

---

## Pasos

### 1. Habilitar DNS en la VPC

Requisito para **Private DNS**:

- `enableDnsSupport = true`
- `enableDnsHostnames = true`

---

### 2. IAM Role para la EC2

- Role type: EC2
- Policy: `AmazonSSMManagedInstanceCore`
- Nombre: `role-ec2-ssm`
- Asociar a la instancia EC2

---

### 3. Security Group para endpoints SSM

`sg-vpce-ssm`:

- Inbound:
  - HTTPS (443)
  - Source: SG de la EC2
- Outbound:
  - Allow all

---

### 4. Crear Interface Endpoints

Crear los **3 obligatorios**:

1. `com.amazonaws.<region>.ssm`
2. `com.amazonaws.<region>.ssmmessages`
3. `com.amazonaws.<region>.ec2messages`

Configuración:

- Type: Interface
- Subnet: `private-subnet-a`
- Security Group: `sg-vpce-ssm`
- Private DNS: **ENABLED**

---

### 5. Registro en SSM

- Inicialmente la EC2 no aparecía
- Se hizo **Stop / Start**
- El SSM Agent reintentó registro
- Apareció en **Session Manager**

---

### 6. Comprobaciones

```bash
curl google.com   # falla (no internet)
aws s3 ls         # depende de IAM
```

## Conclusión Lab 3

* Interface Endpoints:

    * Crean ENIs
    * Usan Security Groups
    * Requieren DNS

* Gestión 100 % privada posible con SSM

**Idea clave SAA-C03:**

    Manage private EC2 without internet → SSM + Interface Endpoints + IAM

# Lab 4 — Limpieza y arquitectura final
## Objetivo

* Quedarte con una arquitectura:

    * Mínima
    * Privada
    * Segura
    * Barata

* Estado final

    * Solo subnet privada
    * Solo rt-private con:

        * local
        * pl-s3 → vpce

    * Sin IGW
    * Sin NAT
    * EC2 gestionable por SSM
    * Acceso privado a S3


# Lab 5 — Endpoint ≠ Permisos (IAM manda)
## Objetivo

* Demostrar que:

    * Endpoint = conectividad
    * IAM = permisos

Error observado
```bash
aws s3 ls
# AccessDenied
```
👉 Red correcta, IAM incorrecto

**Solución (mínimo privilegio)**

Policy para un bucket concreto:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::111-testing-111"]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::111-testing-111/*"]
    }
  ]
}
```

## Conclusión Lab 5

* Endpoint sin IAM → AccessDenied
* No es problema de routing


# Resumen mental SAA-C03

* Privado + S3 → Gateway Endpoint + IAM
* Privado + SSM → Interface Endpoints + IAM + DNS
* Endpoint ≠ NAT
* Endpoint ≠ permisos
* Subnet pública innecesaria si no necesitas internet
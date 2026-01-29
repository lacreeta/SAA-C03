# VPC Endpoints (SAA-C03)

Apuntes orientados a **decisiones de arquitectura** y **trampas de examen**.

---

## Para qué sirven

Permiten que recursos en **subnets privadas** accedan a **servicios de AWS** sin:

* Internet Gateway (IGW)
* NAT Gateway
* IP pública

👉 El tráfico se mantiene en la red de AWS (privado), mejorando **seguridad** y (a menudo) **coste**.

---

## Tipos de VPC Endpoints

### 1) Gateway Endpoint

**Servicios (clave de examen):**

* **S3**
* **DynamoDB**

**Cómo funciona:**

* Se añade como destino en la **Route Table** de la subnet (normalmente privada).
* No crea ENIs.
* No usa Security Groups.

**Cuándo usar (examen):**

* “Acceder a S3/DynamoDB desde una subnet privada sin NAT”
* “Reducir costes de NAT Gateway para tráfico a S3”

**Qué debes recordar:**

* Gateway Endpoint = **Route Table**
* Solo para **S3/DynamoDB**

---

### 2) Interface Endpoint (AWS PrivateLink)

**Servicios:**

* La mayoría de servicios AWS (ej: SSM, EC2 API, CloudWatch, SNS, STS, ECR, KMS, etc.).

**Cómo funciona:**

* Crea una o varias **ENIs** (interfaces) dentro de tus subnets.
* Control de acceso por **Security Group** asociado al endpoint.
* Requiere que el DNS funcione bien (normalmente con **Private DNS** habilitado).

**Cuándo usar (examen):**

* “Acceder a SSM / CloudWatch / API de AWS desde subnets privadas sin internet”
* “No se permite NAT ni IGW”

**Coste (idea general):**

* Tiene coste por hora + tráfico (varía por región/servicio). En examen: lo importante es que **no es gratis** como concepto.

**Qué debes recordar:**

* Interface Endpoint = **ENI + SG**
* Aparece como “privado” dentro de tu VPC

---

## Cómo elegir en el examen

### Regla rápida

* Si el servicio es **S3 o DynamoDB** → **Gateway Endpoint**
* Si es “casi cualquier otro servicio AWS” → **Interface Endpoint (PrivateLink)**

---

## Frases típicas de examen y respuesta

* “Private access to AWS services without traversing the internet” → **VPC Endpoint**
* “Instances in private subnets must access S3 without NAT” → **S3 Gateway Endpoint**
* “No public internet, no NAT allowed, but needs to use SSM” → **Interface Endpoint para SSM**

---

## Trampas comunes

1. **Una subnet no se vuelve pública por tener endpoint**

* Pública/privada lo define el routing hacia IGW, no endpoints.

2. **Gateway Endpoint no usa Security Groups**

* Si una pregunta habla de “permitir por SG”, casi seguro es **Interface Endpoint**.

3. **Peering/TGW no sustituyen endpoints**

* Peering/TGW conectan redes entre VPCs; endpoints conectan **tu VPC con servicios AWS** de forma privada.

4. **Endpoint ≠ NAT**

* NAT sirve para salir a internet (o servicios públicos). Endpoint sirve para ir a **servicios AWS** de forma privada.

---

## Checklist mental (SAA-C03)

* ¿Es tráfico hacia **S3/DynamoDB** desde privado? → Gateway Endpoint
* ¿Es un servicio tipo **SSM/CloudWatch/STS/ECR/KMS** desde privado? → Interface Endpoint
* ¿La pregunta dice “sin internet / sin NAT / compliance”? → Endpoint casi seguro

## Laboratorio
👉 [VPC Endpoints — Laboratorio](endpoints-lab.md)
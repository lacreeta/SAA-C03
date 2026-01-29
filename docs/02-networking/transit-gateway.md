# Transit Gateway (SAA-C03)

Apuntes centrados en **arquitectura a escala**, **comparaciones de examen** y **cuándo es la respuesta correcta**.

---

## Qué es Transit Gateway (TGW)

AWS Transit Gateway es un **hub de red centralizado** que permite conectar:

- Múltiples VPCs
- Redes on-premises (VPN / Direct Connect)
- Todo con **routing transitive**

Arquitectura típica: **hub-and-spoke**.

---

## El problema que resuelve

Sin TGW:
- Muchas VPCs → muchos VPC Peerings
- Arquitectura compleja
- Difícil de mantener y escalar

Con TGW:
- Un punto central de routing
- Gestión simplificada
- Escala limpia

📌 Frase mental:
> “Transit Gateway es el router central de AWS”

---

## Características clave (examen)

- Routing **transitive** ✅
- Conecta **decenas o cientos de VPCs**
- Centraliza conectividad:
  - VPC ↔ VPC
  - VPC ↔ on-prem
- Más caro que peering, pero **mucho más escalable**

---

## Componentes básicos (modelo mental)

- **Transit Gateway**
  - El hub
- **Attachments**
  - VPC attachments
  - VPN attachments
  - Direct Connect attachments
- **Route Tables del TGW**
  - Deciden a dónde va el tráfico entre attachments

No confundir con route tables de las subnets.

---

## Caso típico de examen

> Empresa con múltiples VPCs (prod, dev, shared-services)  
> Necesita conectividad privada entre todas  
> Además conexión con on-prem

👉 **Transit Gateway**

---

## Comparaciones clave (MUY preguntadas)

### Transit Gateway vs VPC Peering

| Feature | VPC Peering | Transit Gateway |
|------|------------|----------------|
| Transitive routing | ❌ | ✅ |
| Escalabilidad | Baja | Alta |
| Arquitectura | Punto a punto | Hub-and-spoke |
| Gestión | Distribuida | Centralizada |
| Complejidad | Baja | Media |

---

### Transit Gateway vs NAT Gateway

- NAT:
  - Solo outbound a internet
  - No conecta redes
- TGW:
  - Conecta redes privadas
  - No da acceso a internet

📌 Si la pregunta habla de **conectividad entre VPCs**, NAT nunca es la respuesta.

---

### Transit Gateway vs VPC Endpoint

- Endpoint:
  - VPC → servicio AWS
- TGW:
  - VPC ↔ VPC / on-prem

---

## Cuándo usar Transit Gateway (respuesta correcta)

- Muchas VPCs
- Necesidad de routing transitive
- Arquitectura hub-and-spoke
- Conectividad centralizada
- Integración con on-prem

---

## Cuándo NO usar Transit Gateway

- Solo 2 VPCs
- Arquitectura simple
- No necesitas routing transitive

👉 En esos casos: **VPC Peering**.

---

## Trampas comunes del examen

1) **Pensar que TGW da acceso a internet**
- No. Para eso sigue siendo NAT/IGW.

2) **Confundir TGW con VPC Endpoint**
- TGW conecta redes
- Endpoint conecta servicios AWS

3) **Olvidar el concepto de “attachment”**
- Sin attachment, la VPC no está conectada al TGW

---

## Frases típicas del examen → respuesta

- “Centralized network connectivity” → **Transit Gateway**
- “Hub and spoke architecture” → **Transit Gateway**
- “Transitive routing required” → **Transit Gateway**
- “Many VPCs need to communicate” → **Transit Gateway**

---

## Checklist mental SAA-C03

- ¿Más de 2–3 VPCs? → TGW
- ¿Necesitas transitive routing? → TGW
- ¿Conectividad on-prem + VPCs? → TGW
- ¿Arquitectura simple punto a punto? → Peering


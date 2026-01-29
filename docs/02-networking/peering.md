# VPC Peering (SAA-C03)

Apuntes centrados en **decisiones de arquitectura**, **limitaciones reales** y **trampas clásicas de examen**.

---

## Qué es VPC Peering

VPC Peering es una conexión de red **privada** entre **dos VPCs** usando la infraestructura interna de AWS.

- El tráfico **no pasa por internet**
- No necesita:
  - Internet Gateway
  - NAT Gateway
- Se comporta como si las VPCs estuvieran en la misma red (a nivel IP)

---

## Características clave (memorizar)

- Comunicación **bidireccional**
- **NO transitive** ❌
- CIDRs **no pueden solaparse**
- Requiere **rutas explícitas** en ambas VPCs
- Puede ser:
  - Mismo account
  - Accounts distintos
  - Regiones distintas (inter-region peering)

---

## La limitación MÁS preguntada: no transitive

### Ejemplo típico de examen

VPC A <--> VPC B
VPC B <--> VPC C

👉 ¿Puede A comunicarse con C?

❌ **NO**

Aunque técnicamente haya camino físico, **AWS no permite routing transitive en peering**.

📌 Frase clave:
> “VPC Peering does not support transitive routing”

Remember this or fail the question 😄

---

## Cómo funciona a nivel práctico

### Pasos reales (modelo mental)
1. Crear peering connection
2. Aceptar peering (en la otra VPC / account)
3. Añadir rutas en:
   - Route table de VPC A → CIDR de VPC B
   - Route table de VPC B → CIDR de VPC A
4. Ajustar Security Groups / NACLs si aplica

Si falla algo → casi siempre es **routing o SG**, no el peering en sí.

---

## Cuándo usar VPC Peering (examen)

- Pocas VPCs (2–5)
- Arquitectura simple
- Necesidad de comunicación directa y privada
- Bajo coste y baja complejidad

📌 Ejemplo típico:
> VPC de aplicaciones necesita hablar con VPC de base de datos en otro account

---

## Cuándo NO usar VPC Peering

- Muchas VPCs (spaghetti de peerings)
- Necesidad de routing transitive
- Arquitectura hub-and-spoke
- Conectividad centralizada (shared services)

👉 Aquí aparece **Transit Gateway**.

---

## Comparaciones clave de examen

### VPC Peering vs Internet / NAT
- Peering: privado, interno AWS
- NAT/IGW: internet (aunque sea tráfico hacia AWS services públicos)

### VPC Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|------|------------|----------------|
| Escala | Baja | Alta |
| Transitive | ❌ | ✅ |
| Arquitectura | Punto a punto | Hub-and-spoke |
| Complejidad | Baja | Media |
| Coste | Bajo | Mayor (pero escalable) |

---

## Trampas comunes del examen

1) **Creer que peering = transitive**
- Falso, siempre no transitive.

2) **Olvidar las route tables**
- Peering creado pero sin rutas → no hay tráfico.

3) **CIDR solapado**
- Peering ni siquiera se puede crear.

4) **Confundir peering con endpoint**
- Peering conecta **VPC ↔ VPC**
- Endpoint conecta **VPC ↔ servicio AWS**

---

## Frases típicas del examen → respuesta

- “Private connectivity between two VPCs” → **VPC Peering**
- “VPCs in different accounts need to communicate” → **VPC Peering**
- “Transitive routing required” → ❌ NO Peering

---

## Checklist mental SAA-C03

- ¿Son solo 2 VPCs? → Peering
- ¿CIDRs no solapados? → OK
- ¿Necesitas transitive routing? → ❌ Peering
- ¿Escala grande / hub central? → Transit Gateway

---

## Laboratorio
- 👉 [VPC Peering - Laboratorio](peering-lab.md)
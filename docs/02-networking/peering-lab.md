# VPC Peering — Labs Prácticos (SAA-C03)

## Objetivo global

Entender **VPC Peering de forma práctica**, sabiendo:

- Qué hace y qué **NO** hace
- Cómo funciona el routing entre VPCs
- Cómo influyen los Security Groups
- Por qué el peering **no es transitivo**
- Errores y trampas típicas del examen SAA-C03

---

## Arquitectura base

- 2 VPCs en la misma región
- CIDRs no solapados
- 1 EC2 por VPC
- 1 VPC Peering Connection
- Routing explícito en ambas VPCs

---

# Lab 1 — Crear VPCs y EC2 (sin conectividad)

## Objetivo

Crear dos VPCs aisladas y comprobar que **no se comunican por defecto**.

---

## Pasos

### VPC-A
- Nombre: `vpc-a`
- CIDR: `10.0.0.0/16`

### VPC-B
- Nombre: `vpc-b`
- CIDR: `10.1.0.0/16`

---

### Subnets

- VPC-A:
  - `subnet-a-public`
  - CIDR: `10.0.1.0/24`
  - Route table:
    - `0.0.0.0/0 → IGW`

- VPC-B:
  - `subnet-b-private`
  - CIDR: `10.1.1.0/24`
  - Route table:
    - solo `local`

---

### EC2

#### EC2-A (pública)
- Subnet: `subnet-a-public`
- IP pública: enabled
- SG-A:
  - SSH (22) → 0.0.0.0/0

#### EC2-B (privada)
- Subnet: `subnet-b-private`
- IP pública: disabled
- SG-B:
  - Sin inbound ICMP todavía

---

## Resultado esperado

- SSH a EC2-A funciona
- No hay conectividad privada entre A y B

---

# Lab 2 — Crear y aceptar VPC Peering

## Objetivo

Crear el peering y comprobar que **sin routing no hay tráfico**.

---

## Pasos

1. Crear VPC Peering:
   - Requester: VPC-A
   - Accepter: VPC-B

2. Aceptar el peering desde VPC-B

3. Verificar estado:
```text
Active
```

---

## Resultado esperado

- El peering existe
- Las EC2 **no se comunican todavía**

---

# Lab 3 — Routing explícito entre VPCs

## Objetivo

Permitir conectividad privada mediante route tables.

---

## Pasos

### Route table en VPC-A
```text
10.1.0.0/16 → pcx-XXXX
```

### Route table en VPC-B
```text
10.0.0.0/16 → pcx-XXXX
```

---

## Prueba

Desde EC2-A:
```bash
ping <IP-privada-EC2-B>
```

---

## Resultado esperado

- El ping empieza a funcionar
- El tráfico va por IP privada

---

# Lab 4 — Security Groups y peering

## Objetivo

Demostrar que **routing correcto no basta** si los SG bloquean.

---

## Pasos

### SG-A (VPC-A)
- Inbound:
  - SSH (22) → 0.0.0.0/0
- Outbound:
  - All traffic

---

### SG-B (VPC-B)
- Inbound:
  - ICMP → CIDR VPC-A (`10.0.0.0/16`)
- Outbound:
  - All traffic

---

## Prueba

Desde EC2-A:
```bash
ping <IP-privada-EC2-B>
```

---

## Resultado esperado

- Ping funciona solo si SG-B permite ICMP

---

# Lab 5 — Ruta en estado Blackhole

## Objetivo

Entender el estado **blackhole** en route tables.

---

## Qué ocurre

- Si el peering está en `pending-acceptance`
- O ha sido eliminado

Las rutas aparecen como:
```text
blackhole
```

---

## Conclusión

> Blackhole = el recurso destino no está disponible

---

# Lab 6 — Peering NO transitivo (muy examen)

## Objetivo

Demostrar que **VPC Peering no es transitivo**.

---

## Setup

- VPC-A ↔ VPC-B
- VPC-B ↔ VPC-C
- NO hay peering A ↔ C

---

## Resultado

- A NO puede comunicarse con C
- Aunque B tenga rutas a ambos

---

## Conclusión

> VPC Peering is NOT transitive

---

# Lab 7 — Errores típicos y trampas SAA-C03

## Trampas comunes

- CIDRs solapados
- Olvidar aceptar el peering
- Falta de rutas en una de las VPCs
- SG bloqueando ICMP/TCP
- Probar con IP pública

---

## Checklist mental de examen

1. CIDRs no solapados
2. Peering activo
3. Rutas en ambas route tables
4. SG permiten tráfico
5. IP privada usada

---

## Resumen final

- VPC Peering conecta **redes privadas**
- Requiere routing explícito
- No es transitivo
- No comparte IGW, NAT ni endpoints
- Siempre usar IP privada para probar

---

## Resultado

Tras estos labs:
- Entiendes VPC Peering de verdad
- Sabes depurar problemas rápido
- Estás listo para las preguntas de examen

# Subnets — Labs Prácticos (SAA-C03)

## Objetivo global

Entender **cómo funcionan realmente las subnets en AWS**, practicando:

- Cómo el **CIDR** divide la VPC
- Cómo las **route tables** definen el comportamiento
- Qué hace (y qué NO hace) un **Internet Gateway**
- Diferencia real entre **subnet pública y privada**
- Errores y trampas típicas de examen

---

## Arquitectura base de los labs

- 1 VPC
- 2 subnets
- 2 route tables
- 1 Internet Gateway
- EC2 de prueba

---

# Lab 1 — Crear VPC pequeña y dividirla en subnets

## Objetivo

Entender que:
- El CIDR de la VPC es el **límite total**
- Las subnets **parten** ese espacio

### Pasos

#### Crear VPC
- Nombre: `lab-subnets`
- CIDR: `10.0.0.0/24`

#### Crear subnets
- `subnet-a` → `10.0.0.0/25`
- `subnet-b` → `10.0.0.128/25`

### Conclusión
- El CIDR no define si una subnet es pública o privada
- Una VPC pequeña limita el crecimiento

---

# Lab 2 — Route Tables y asociación real

## Objetivo

Entender que:
- Cada subnet usa **una** route table
- Una route table puede usarse por **varias** subnets

### Pasos

- Identificar la **main route table**
- Crear:
  - `rt-public`
  - `rt-private`
- Asociar:
  - `rt-public` → `subnet-a`
  - `rt-private` → `subnet-b`

---

# Lab 3 — Internet Gateway

## Objetivo

Ver que:
> Sin IGW no hay internet

### Pasos

- Crear IGW
- Attach a la VPC
- En `rt-public` añadir:
  - `0.0.0.0/0 → IGW`

---

# Lab 4 — EC2 pública vs privada

## Pasos

### EC2 pública
- Subnet: `subnet-a`
- IP pública: Enabled

```bash
curl google.com
```

### EC2 privada
- Subnet: `subnet-b`
- IP pública: Disabled

```bash
curl google.com
```

---

# Lab 5 — Intercambiar route tables

## Objetivo

Demostrar que la subnet no es pública ni privada por sí misma.

### Pasos

- Asociar `rt-public` a `subnet-b`
- Asociar `rt-private` a `subnet-a`

---

# Lab 6 — Límite del CIDR

## Objetivo

Ver qué pasa cuando el CIDR se agota.

### Pasos

- Intentar crear una tercera subnet

### Resultado
- No queda espacio disponible

---

## Resumen mental SAA-C03

- VPC = universo de IPs
- Subnet = partición
- Route table = comportamiento
- IGW = salida a internet
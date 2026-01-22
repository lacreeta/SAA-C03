# VPC — Laboratorio

## 22-01-2026

- VPC: default
- Región: us-east-1

### Subnets
- La VPC por defecto contiene varias subnets (una por AZ).
- Una subnet **no es pública o privada por sí misma**.
- El carácter público/privado lo determina la **route table asociada**.

### Route Table
- La route table define **a dónde se dirige el tráfico que sale de una subnet**.
- Regla clave:
  - Ruta `0.0.0.0/0 → Internet Gateway` → subnet pública
  - Sin esa ruta → subnet privada

### Internet Gateway
- El Internet Gateway está asociado a la VPC.
- Permite conectividad entre la VPC e internet.
- Solo se usa si la route table lo permite.

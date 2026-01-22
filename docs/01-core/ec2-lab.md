# EC2 — Laboratorio

## 22-01-2026

- Instancia: `t3.micro`
- Región: `us-east-1`
- IP pública asignada → salida a internet
- Subnet pública (ruta a IGW)
- Security Group vs Network ACL:
  - SG: stateful, instancia
  - NACL: stateless, subnet
- Cambio de tipo a `t3.small`
  - La instancia no se recrea
- Volumen root:
  - Tipo: gp3
  - Asociado a una AZ
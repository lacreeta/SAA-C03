# EC2

# EC2

## Para qué sirve
Instancias de cómputo flexibles para workloads que requieren control total del sistema operativo.

## Cuándo usar EC2 (examen)
- Aplicaciones legacy
- Necesidad de control del SO
- Requisitos específicos de CPU/RAM
- Workloads no event-driven

## Cuándo NO usar EC2
- Arquitecturas serverless → Lambda
- Microservicios gestionados → ECS / EKS
- Procesamiento esporádico → Lambda / Fargate

## Conceptos clave
- EC2 puede tener **IP pública o privada**
- La conectividad a internet depende de:
  - IP pública
  - Subnet pública
  - Ruta `0.0.0.0/0 → IGW`
- EC2 es **flexible**:
  - Se puede cambiar el tipo de instancia sin recrearla
- EC2 usa **EBS**, ligado a una **AZ**

## Seguridad
- **Security Groups**
  - Stateful
  - A nivel de instancia
- **Network ACLs**
  - Stateless
  - A nivel de subnet

## Trampas de examen
- Una EC2 en subnet privada **NO sale a internet** sin NAT
- Un volumen EBS **no puede moverse entre AZs**
- SG ≠ IAM (muchos los confunden)

## Laboratorio
👉 [EC2 — Laboratorio](ec2-lab.md)
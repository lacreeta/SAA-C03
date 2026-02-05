# RDS LAB — Arquitectura, Seguridad y Conectividad (SAA-C03)

## Objetivo del laboratorio

Entender cómo se despliega RDS correctamente desde el punto de vista de:
- Arquitectura
- Seguridad
- Conectividad
- Trampas de examen (SAA-C03)

Este laboratorio NO busca administrar bases de datos, sino aprender a decidir y diseñar bien.

## Alcance del lab

- RDS Single-AZ (Free Tier)
- RDS en subnets privadas
- Acceso solo desde una EC2
- Sin acceso desde internet
- Uso correcto de DB Subnet Group y Security Groups

## Prerrequisitos

- VPC creada
- 1 subnet pública
- 2 subnets privadas en AZ distintas
- 1 EC2 en subnet pública
- Acceso SSH a la EC2
- Free Tier activo

## Arquitectura final

VPC
- Public Subnet (AZ-a)
  - EC2 (cliente / bastion)
- Private Subnet A (AZ-a)
- Private Subnet B (AZ-b)
  - RDS (MySQL / PostgreSQL)

## Paso 1 — Security Group para RDS

- Crear SG llamado sg-rds
- Inbound:
  - Puerto 3306 (MySQL) o 5432 (PostgreSQL)
  - Source: Security Group de la EC2
- Outbound: allow all

Concepto clave:
RDS se protege con SG → SG, no con IPs.

## Paso 2 — DB Subnet Group

- Nombre: rds-private-subnets
- VPC: la del lab
- Subnets:
  - Private Subnet A
  - Private Subnet B

Nota de examen:
El DB Subnet Group siempre requiere subnets en al menos 2 AZ, incluso en Single-AZ.

## Paso 3 — Crear la instancia RDS

- Standard create
- Engine: MySQL o PostgreSQL
- Template: Free Tier
- DB identifier: rds-lab
- Instance class: db.t3.micro
- Storage: ≤ 20 GB, autoscaling OFF
- Connectivity:
  - DB Subnet Group: rds-private-subnets
  - Public access: NO
  - Security Group: sg-rds
- Backups: enabled (7 días)

## Paso 4 — Observaciones importantes

- RDS no tiene IP pública
- Se accede siempre mediante endpoint DNS
- Vive en subnets privadas
- Es un servicio gestionado (no SSH, no SO)

## Paso 5 — Conexión desde la EC2

MySQL:
sudo yum install mysql -y
mysql -h <endpoint> -u <user> -p

PostgreSQL:
sudo yum install postgresql15 -y
psql -h <endpoint> -U <user> -d postgres

Resultado esperado:
- Conecta desde EC2
- No conecta desde internet

## Trampas de examen

- Multi-AZ ≠ DB Subnet Group
- IP pública ≠ accesible
- RDS no se accede por IP
- RDS no es EC2

## Limpieza

- Eliminar la instancia RDS
- Eliminar el DB Subnet Group si no se reutiliza

## Conclusión

Este lab refuerza:
- Buen diseño de red
- Seguridad por defecto
- Mentalidad de arquitectura orientada a examen

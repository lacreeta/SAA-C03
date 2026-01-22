## Qué es una VPC
Red virtual aislada a nivel regional.

## Subnets
- División lógica de una VPC
- Asociadas a una AZ
- Pueden ser públicas o privadas según routing

## Route Tables
- Definen el destino del tráfico saliente
- Se asocian a subnets

## Internet Gateway
- Conecta la VPC con internet
- Necesario para tráfico entrante/saliente

## Alcance
- Una VPC es **regional**
- Las subnets son **zonales (AZ)**

**Implicación de examen**:
- No puedes mover una subnet entre AZs
- Alta disponibilidad = múltiples subnets en distintas AZs

## Regla clave de examen
Una subnet es pública **SOLO** si:
- Tiene una ruta 0.0.0.0/0
- Apunta a un Internet Gateway

## Trampas de examen
- Asignar IP pública ≠ subnet pública 
- IGW no se asocia a subnets
- Una VPC sin IGW **nunca** tiene acceso a internet
- Un IGW sin ruta en la route table **no se usa**

## Relación con EC2
Para que una EC2 sea accesible desde internet necesita:
- Estar en una subnet pública
- Tener una IP pública
- Security Group permitiendo el tráfico
- Un servicio escuchando en el puerto

La VPC solo resuelve la **conectividad de red**, no los permisos.

## Default VPC
- AWS crea una por región
- Tiene subnets públicas por defecto
- Incluye Internet Gateway y routing configurado
- Pensada para “funcionar sin pensar”

**Trampa**:
- Funciona bien para pruebas
- Mala práctica para producción

## Componentes y responsabilidades

| Componente | Decide qué |
|----------|------------|
| VPC | Aislamiento de red |
| Subnet | Dónde vive el recurso |
| Route Table | Dónde va el tráfico |
| IGW | Conexión a internet |
| SG | Qué tráfico se permite |
| NACL | Filtro a nivel subnet |

## Laboratorio
👉 [VPC — Laboratorio](vpc-lab.md)
# Routing, Internet Gateway & NAT Gateway

## Objetivo
Este documento resume cómo funciona el **routing en una VPC**, el papel del **Internet Gateway (IGW)** y la **NAT Gateway**, y cómo se determina si una subnet es pública o privada en AWS.

Está orientado a **entender los conceptos clave de arquitectura** y a preparar preguntas típicas del examen **AWS SAA-C03**.

---

## Conceptos clave

### ¿Qué determina si una subnet es pública o privada?
Una subnet es:

- **Pública** si su *route table* tiene una ruta:
  - `0.0.0.0/0 → Internet Gateway (IGW)`
- **Privada** si **no** tiene ninguna ruta hacia un IGW

> El ajuste **auto-assign public IPv4** NO define si una subnet es pública o privada.

---

## Internet Gateway (IGW)

El **Internet Gateway** permite la comunicación directa entre una VPC e internet.

Características:
- Se asocia a una **VPC**
- Permite tráfico **entrante y saliente**
- Requiere:
  - Subnet con route table que apunte al IGW
  - Instancias con IP pública o Elastic IP

Uso típico:
- Subnets públicas
- Load Balancers públicos
- Bastion Hosts

---

## NAT Gateway

La **NAT Gateway** permite que instancias en **subnets privadas** accedan a internet **sin ser accesibles desde fuera**.

Características:
- Debe estar en una **subnet pública**
- Requiere una **Elastic IP**
- Solo permite **tráfico saliente iniciado desde la subnet privada**
- Realiza **NAT de origen**

Routing típico:
- Subnet privada:
  - `0.0.0.0/0 → NAT Gateway`
- Subnet pública:
  - `0.0.0.0/0 → IGW`

Uso típico:
- EC2 privadas que necesitan:
  - Actualizaciones del sistema
  - Acceso a APIs externas
  - Descarga de paquetes

---

## Comparación rápida: IGW vs NAT Gateway

| Característica | IGW | NAT Gateway |
|---------------|-----|-------------|
| Tráfico entrante | Sí | No |
| Tráfico saliente | Sí | Sí |
| Uso principal | Subnets públicas | Subnets privadas |
| Requiere IP pública en EC2 | Sí | No |
| Alta disponibilidad | Sí (gestionado por AWS) | Sí (por AZ) |

---

## Acceso a instancias en subnets privadas

Las instancias en subnets privadas **no son accesibles directamente desde internet**.

Opciones habituales:
- **SSM Session Manager** (recomendado)
- **Bastion Host** en subnet pública
- **EC2 Instance Connect Endpoint**

---

## Laboratorio práctico

El laboratorio completo con pruebas reales de routing, IGW y NAT Gateway se encuentra en:

- 👉 [Laboratorio](routing-lab.md)

Incluye:
- Creación de subnets públicas y privadas
- Configuración de route tables
- Uso de NAT Gateway
- Acceso a EC2 en subnet privada
- Limpieza de recursos

---

## Key Takeaways (Examen)

- El routing define la conectividad, no la subnet en sí
- IGW = acceso directo a internet
- NAT Gateway = salida controlada desde subnets privadas
- Subnet pública ≠ auto-assign public IPv4
- Subnet privada ≠ sin internet (puede usar NAT)

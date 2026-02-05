# DynamoDB LABS — Core + VPC Endpoint (SAA-C03)

## Objetivo general

Consolidar el uso de DynamoDB desde una perspectiva de arquitectura AWS (SAA-C03), entendiendo:
- Cómo se accede a DynamoDB realmente
- Qué papel juega IAM
- Por qué DynamoDB no vive en una VPC
- Cómo los VPC Endpoints permiten acceso privado sin internet

Incluye DOS LABS:
1. DynamoDB básico
2. DynamoDB con VPC Endpoint (Gateway)

---

# LAB 1 — DynamoDB Básico

## Objetivo del lab

Entender los conceptos fundamentales de DynamoDB:
- Diseño de claves
- Query vs Scan
- TTL
- Acceso por API (no red)

---

## Arquitectura conceptual

EC2 -> AWS API (HTTPS) -> DynamoDB (servicio regional, fuera de la VPC)

---

## Paso 1 — Crear la tabla

- Table name: users
- Partition key: user_id (String)
- Sort key: created_at (Number)
- Capacity mode: On-Demand

Concepto de examen:
Si no conoces el patrón de tráfico → On-Demand.

---

## Paso 2 — Insertar items

Ejemplo:

{
  "user_id": { "S": "u123" },
  "created_at": { "N": "1700000000" },
  "email": { "S": "user1@test.com" }
}

---

## Paso 3 — Query vs Scan

Query:
- Usa Partition Key
- Eficiente

Scan:
- Recorre toda la tabla
- Costoso

Trampa de examen:
Query siempre es mejor que Scan.

---

## Paso 4 — TTL (Time To Live)

- Activar TTL
- Attribute name: ttl
- Eliminación automática de items

---

## Paso 5 — Acceso desde EC2

- Usar IAM Role (no access keys)
- Permisos DynamoDB
- Acceso vía AWS CLI o SDK

---

# LAB 2 — DynamoDB + VPC Endpoint (Gateway)

## Objetivo del lab

Acceder a DynamoDB desde una VPC:
- Sin Internet
- Sin NAT Gateway
- De forma privada

---

## Qué es un VPC Endpoint

Un VPC Endpoint permite que recursos en una VPC accedan a servicios AWS sin salir a internet.

Tipos:
- Gateway Endpoint: S3, DynamoDB
- Interface Endpoint: otros servicios

---

## Arquitectura final

EC2 -> Gateway VPC Endpoint -> DynamoDB

---

## Paso 1 — Crear el VPC Endpoint

- Servicio: com.amazonaws.us-east-1.dynamodb
- Tipo: Gateway
- Asociar route table de la EC2

---

## Paso 2 — Efecto del endpoint

Se añade una ruta a DynamoDB apuntando al endpoint.
El tráfico deja de usar Internet/NAT.

---

## Paso 3 — Acceso desde EC2

Comandos AWS CLI funcionan igual.
La diferencia es arquitectónica, no de aplicación.

---

## Preguntas típicas de examen

- Acceso privado a DynamoDB → Gateway VPC Endpoint
- Reducir costes de NAT → Gateway VPC Endpoint

---

## Conclusión

DynamoDB:
- No vive en VPC
- No usa SG
- Se accede por API
- Puede integrarse con VPC mediante VPC Endpoints

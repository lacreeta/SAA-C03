# Elastic Load Balancing (ELB)

## Qué es ELB

Elastic Load Balancing es un servicio gestionado que **distribuye el tráfico entrante** entre múltiples targets (por ejemplo, instancias EC2) para mejorar:

* Alta disponibilidad
* Escalabilidad
* Tolerancia a fallos

ELB elimina la necesidad de exponer instancias individuales y gestionar balanceo manual.

---

## Alcance y arquitectura

* ELB es un servicio **REGIONAL**.
* Opera en **múltiples Availability Zones**.
* Se asocia a **subnets** dentro de una VPC.

**Implicación de examen**:

* Alta disponibilidad = múltiples AZs + ELB.
* Un ELB no vive en una AZ concreta.

---

## Tipos de Load Balancer (MUY IMPORTANTE)

### Application Load Balancer (ALB)

* Capa 7 (HTTP / HTTPS).
* Entiende el tráfico web.
* Permite **routing avanzado**:

  * Por path (`/api`, `/images`)
  * Por host (`app.example.com`)

**Cuándo usar ALB**:

* Aplicaciones web
* Microservicios
* Routing basado en URL

---

### Network Load Balancer (NLB)

* Capa 4 (TCP / UDP).
* Extremadamente rápido y de baja latencia.
* Maneja millones de requests por segundo.

**Cuándo usar NLB**:

* Protocolos no HTTP
* Requisitos de latencia muy baja
* IP estática

---

### Classic Load Balancer (CLB)

* Servicio legacy.
* Mezcla de L4 y L7.

**Examen**:

* Saber que existe.
* No usarlo en arquitecturas nuevas.

---

## Componentes clave

### Listener

* Define:

  * Puerto
  * Protocolo
* Escucha el tráfico entrante.

### Target Group

* Conjunto de targets:

  * EC2
  * IPs
  * Lambda (solo ALB)
* Configura:

  * Health checks
  * Puerto destino

### Health Checks

* ELB comprueba periódicamente si un target está sano.
* Si falla:

  * El target deja de recibir tráfico.

**Trampa de examen**:

* Una instancia puede estar RUNNING pero fuera del target group por health checks fallidos.

---

## Seguridad

### Subnets

* ELB suele estar en **subnets públicas**.
* Targets suelen estar en **subnets privadas**.

### Security Groups

* **ALB**:

  * Tiene Security Group.
* **NLB**:

  * NO usa Security Groups.

**Implicación de examen**:

* Tráfico externo → ALB SG → EC2 SG.

---

## Integración con Auto Scaling

* ELB distribuye tráfico.
* Auto Scaling ajusta el número de instancias.

Funcionan juntos para:

* Alta disponibilidad
* Escalabilidad automática

---

## Casos típicos de examen

* Alta disponibilidad sin gestionar IPs → ELB
* Routing por URL → ALB
* Tráfico TCP de baja latencia → NLB
* Aplicación web escalable → ALB + Auto Scaling

---

## Trampas de examen

* ELB no es global (es regional).
* NLB no soporta routing por path.
* ALB no sirve tráfico no HTTP.
* Health checks mal configurados pueden dejar targets fuera.

---

## Relación con otros servicios

| Servicio     | Relación con ELB       |
| ------------ | ---------------------- |
| EC2          | Targets comunes        |
| Auto Scaling | Escalado automático    |
| VPC          | Networking             |
| ACM          | Certificados TLS (ALB) |

---

## Resumen mental SAA-C03

* ELB = alta disponibilidad + escalabilidad.
* ALB para HTTP/HTTPS.
* NLB para TCP/UDP.
* Siempre pensar en AZs, health checks y security groups.

## Laboratorio
👉 [ELB — Laboratorio](elb-lab.md)
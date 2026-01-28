# Auto Scaling (EC2 Auto Scaling)

## Qué es Auto Scaling

Auto Scaling es un servicio que **crea y elimina instancias EC2 automáticamente** en función de la demanda, fallos o políticas definidas.

Su objetivo principal es:

* Mantener la **capacidad adecuada**
* Garantizar **alta disponibilidad**
* Evitar intervención manual

Auto Scaling **no sustituye** a ELB:

* ELB reparte tráfico
* Auto Scaling ajusta el número de instancias

Juntos forman un patrón básico de arquitectura altamente disponible.

---

## Componentes principales

### Launch Template (o Launch Configuration)

Define **cómo debe ser una instancia** cuando Auto Scaling la crea:

* AMI
* Tipo de instancia
* Security Groups
* User Data

📌 En arquitecturas modernas se usa **Launch Template**.

---

### Auto Scaling Group (ASG)

Es el grupo que gestiona las instancias.

Parámetros clave:

* **Minimum capacity** → mínimo de instancias
* **Desired capacity** → instancias deseadas
* **Maximum capacity** → límite superior

El ASG siempre intenta mantener el estado deseado.

---

### Scaling Policies

Definen **cuándo escalar**:

* Scale out (añadir instancias)
* Scale in (eliminar instancias)

Ejemplos:

* CPU > 70%
* Número de requests

---

## Integración con ELB

* El ASG se asocia a un **Target Group**
* Las instancias creadas se registran automáticamente
* ELB solo envía tráfico a instancias **healthy**

📌 Si una instancia falla:

* ELB deja de enviar tráfico
* Auto Scaling la reemplaza

---

## Health Checks

Auto Scaling puede usar:

* EC2 health checks
* ELB health checks (recomendado)

Con ELB:

* Si el health check falla → la instancia se reemplaza

---

## Casos típicos de examen

* Tráfico variable → Auto Scaling
* Alta disponibilidad → ASG + múltiples AZs
* Recuperación automática ante fallos

---

## Trampas de examen

* Auto Scaling **no balancea tráfico**
* ELB **no crea instancias**
* Desired capacity puede ser distinto de min/max
* Sin Launch Template no hay Auto Scaling

---

## Resumen mental SAA-C03

* ELB = tráfico
* Auto Scaling = capacidad
* Launch Template = cómo nace una EC2
* ASG = cuántas EC2 hay

## Laboratorio
👉 [Auto Scaling — Laboratorio](autoscaling-lab.md)
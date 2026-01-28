## Auto Scaling — Lab 1 (sin ELB)

## Fecha

28-01-2026

---

## Objetivo del laboratorio

Comprender el funcionamiento básico de **EC2 Auto Scaling** sin integración con Load Balancers, observando cómo el servicio:

* Crea instancias automáticamente
* Mantiene el número deseado de instancias
* Reemplaza instancias fallidas sin intervención manual

---

## Arquitectura del lab

* Launch Template
* Auto Scaling Group (ASG)
* Múltiples Availability Zones
* Sin ELB

---

## Launch Template

Se creó un **Launch Template** que define cómo deben crearse las instancias EC2 gestionadas por el Auto Scaling Group.

El Launch Template incluye:

* AMI de Amazon Linux
* Tipo de instancia micro
* Security Group propio del ASG
* User Data para instalar un servidor web y servir contenido HTTP

Este template actúa como el **molde** que Auto Scaling utiliza cada vez que necesita crear o reemplazar una instancia.

---

## Auto Scaling Group

Se creó un **Auto Scaling Group** usando el Launch Template anterior, configurado en múltiples Availability Zones para alta disponibilidad.

Parámetros clave:

* **Minimum capacity**: 1
* **Desired capacity**: 1
* **Maximum capacity**: 2

El ASG intenta en todo momento mantener el valor de *desired capacity*, respetando los límites definidos por *min* y *max*.

---

## Pruebas realizadas

### Escalado manual

* Se incrementó la **desired capacity** de 1 a 2
* El ASG creó automáticamente una nueva instancia EC2

### Auto-healing

* Se terminó manualmente una instancia EC2
* El ASG detectó que la desired capacity no se cumplía
* Se creó automáticamente una nueva instancia de reemplazo

### Escalado hacia abajo

* Se redujo la **desired capacity** a 1
* El ASG eliminó una instancia manteniendo el mínimo configurado

---

## Observaciones clave

* Las instancias EC2 **no se crean manualmente**, siempre son gestionadas por el ASG
* El ASG es el único componente que decide cuántas instancias deben existir
* El Launch Template define completamente cómo nace una EC2

---

## Implicaciones de examen (SAA-C03)

* Auto Scaling permite **alta disponibilidad** sin intervención manual
* El reemplazo automático de instancias se conoce como **auto-healing**
* ELB no es necesario para que Auto Scaling funcione
* Sin Launch Template (o Launch Configuration) no es posible usar Auto Scaling

---

## Conclusión

Este laboratorio demuestra que Auto Scaling es responsable de la **gestión de capacidad**, manteniendo el número correcto de instancias EC2 incluso ante fallos, y sienta las bases para su integración posterior con Elastic Load Balancing.

## Replicación Lab 2-ELB

El laboratorio de ELB fue replicado sin guía, confirmando la correcta asimilación de los conceptos de balanceo, target groups y security groups.

## ELB + Auto Scaling — Lab 3 (integración completa)

## Fecha

28-01-2026

---

## Objetivo del laboratorio

Construir una arquitectura **altamente disponible y autoescalable** donde:

* El tráfico se balancea mediante un **Application Load Balancer (ALB)**
* Las instancias EC2 se crean y destruyen automáticamente con **Auto Scaling**
* No existe gestión manual de instancias

---

## Arquitectura implementada

```
Usuario
  ↓
Application Load Balancer
  ↓
Target Group
  ↓
Auto Scaling Group
  ↓
EC2 (creadas dinámicamente)
```

---

## Componentes

* **Launch Template**

  * Define cómo nacen las EC2
  * Incluye AMI, tipo de instancia, Security Group y User Data

* **Auto Scaling Group (ASG)**

  * Gestiona el número de instancias
  * Mantiene la desired capacity
  * Reemplaza instancias fallidas automáticamente

* **Target Group**

  * Punto de unión entre ALB y ASG
  * Gestiona health checks y registro de instancias

* **Application Load Balancer (ALB)**

  * Recibe tráfico HTTP desde internet
  * Reenvía el tráfico al Target Group

---

## Funcionamiento observado

* El usuario accede únicamente al **DNS del ALB**
* El contenido servido proviene del **User Data del Launch Template**
* Las instancias EC2 no se crean manualmente
* Al escalar o eliminar instancias, el servicio sigue funcionando

---

## Pruebas realizadas

* Incremento y decremento de la **desired capacity**
* Terminación manual de instancias EC2

En todos los casos:

* El ASG creó instancias de reemplazo
* El ALB solo envió tráfico a instancias healthy

---

## Implicaciones de examen (SAA-C03)

* Alta disponibilidad sin gestión manual → **ALB + Auto Scaling**
* ELB no crea instancias, Auto Scaling no balancea tráfico
* El ALB siempre apunta a un **Target Group**, nunca directamente a EC2

---

## Conclusión

Este laboratorio demuestra el patrón clásico de arquitectura en AWS para aplicaciones web escalables y altamente disponibles, basado en la integración de **ELB y Auto Scaling**.


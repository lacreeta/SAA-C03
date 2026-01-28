## ELB — Labs prácticos

## Fecha

28-01-2026

---

## Objetivo del laboratorio

Configurar un **Application Load Balancer (ALB)** que distribuya tráfico HTTP entre dos instancias EC2 en distintas Availability Zones, entendiendo el papel de **User Data**, **Security Groups** y **Target Groups** dentro de una arquitectura altamente disponible.

---

## Creación de las instancias EC2

Se crearon **dos instancias EC2**, cada una con un **User Data distinto**.

El *User Data* es un script que AWS ejecuta **una sola vez, en el primer arranque de la instancia**, y se utiliza para preparar automáticamente la máquina (por ejemplo, instalar software y arrancar servicios). En este caso, se usó para instalar un servidor web y devolver un contenido distinto en cada instancia, lo que permite comprobar visualmente el balanceo de tráfico.

---

## Security Groups en las instancias EC2

Ambas instancias EC2 utilizan **el mismo Security Group**, lo cual es una buena práctica tanto a nivel real como de examen.

### Razones

1. **Mismo rol**

* Mismo tipo de instancia
* Misma aplicación
* Mismo puerto expuesto (HTTP 80)

👉 Mismo rol implica **misma política de seguridad**.

2. **ELB trabaja a nivel de grupo**

* El Load Balancer envía tráfico a un **Target Group**
* Todas las instancias del Target Group deben aceptar el mismo tipo de tráfico

Usar distintos Security Groups para instancias idénticas añade:

* Más complejidad
* Más posibilidades de error
* Ningún beneficio real

3. **Patrón habitual (examen)**

* Un Security Group para el **ALB**
* Un Security Group para las **instancias EC2**
* Ambos reutilizables y fácilmente mantenibles

⚠️ Importante: aunque las EC2 compartan el mismo Security Group, **no es el mismo Security Group que el del ALB**.

---

## Target Group

Se creó un **Target Group** con:

* Protocolo: HTTP
* Puerto: 80
* Targets: las dos instancias EC2

### ¿Para qué sirve un Target Group?

El Target Group es el componente que define:

* **A quién envía tráfico el Load Balancer**
* **Cómo comprobar si los destinos están sanos (health checks)**

El ALB **no envía tráfico directamente a instancias EC2**, sino al Target Group. El Target Group decide:

* Qué instancias reciben tráfico
* Cuáles quedan fuera si fallan los health checks

Este desacoplamiento permite:

* Cambiar instancias sin tocar el ALB
* Integrarse fácilmente con Auto Scaling

---

## Application Load Balancer

Se configuró un **Application Load Balancer internet-facing**, desplegado en **dos Availability Zones**, y asociado al Target Group creado previamente.

* El ALB recibe tráfico HTTP desde internet
* Distribuye el tráfico entre las instancias EC2 sanas del Target Group
* El usuario final accede únicamente al **DNS del ALB**, sin conocer las instancias subyacentes

Al acceder repetidamente al DNS del ALB, se observó que la respuesta alternaba entre ambas instancias, demostrando:

* Balanceo de carga
* Alta disponibilidad
* Correcto funcionamiento de los health checks

---

## Conclusiones del laboratorio

Este laboratorio demuestra:

* Cómo ELB permite balancear tráfico sin gestionar IPs manualmente
* La importancia de los Target Groups como intermediarios entre el Load Balancer y las instancias
* El uso de Security Groups separados para ELB y EC2
* Un patrón de arquitectura típico de examen: **ALB + múltiples AZs + instancias homogéneas**

Este diseño sienta la base para integrar **Auto Scaling** y construir aplicaciones web altamente disponibles y escalables.

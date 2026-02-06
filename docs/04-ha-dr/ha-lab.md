# HA Lab (Alta Disponibilidad) — ALB Multi-AZ (SAA-C03)

> Objetivo: practicar **High Availability real dentro de una región** usando el patrón clásico:
>
> **Internet → ALB → (Targets en 2 AZ)**
>
> Este lab está escrito para que entiendas **qué estás eligiendo y por qué**, y para que lo puedas repetir sin dudas.

---

## Qué vas a aprender (lo que “debe quedarte” para el examen)

1. **HA ≠ DR**  
   - HA: el servicio **sigue funcionando** si cae una **instancia** o una **AZ** (dentro de la región).
   - DR: recuperas tras desastre (caída regional, pérdida de datos, etc.). En DR vimos Route 53 failover y backups.

2. Un **ALB** (Application Load Balancer) te da HA porque:
   - Reparte tráfico entre targets.
   - Hace **health checks** y **deja de mandar tráfico** a targets unhealthy.
   - Es **Multi-AZ** si lo pones en subnets de **2+ AZ**.

3. La “ventana de error” (p. ej., 502) al parar un backend es normal y depende de:
   - `HealthCheckIntervalSeconds`
   - `UnhealthyThresholdCount`
   - TTL/cachés NO aplican aquí (eso era DNS / Route 53). Aquí es L7 con health checks.

---

## Arquitectura final (patrón de examen)

```
Usuarios (Internet)
      |
      v
+------------------------+
| ALB (Internet-facing)  |
| Listener 80/443        |
+------------------------+
      |
      v
+------------------------------+
| Target Group (HTTP:80)       |
| Health checks: GET /         |
+------------------------------+
   |                      |
   v                      v
EC2-AZ1 (Apache)      EC2-AZ2 (Apache)
(healthy)             (healthy)
```

**Qué es “HA” aquí:** si EC2-AZ1 cae, el ALB enruta solo a EC2-AZ2 (y viceversa).

---

# Pre-requisitos (para no pelearte con networking)

### Necesitas:
- 1 VPC con Internet Gateway (IGW).
- **2 subnets públicas en AZ distintas** (p. ej. `eu-west-1a` y `eu-west-1b`).
- Route tables correctas: `0.0.0.0/0 → IGW`.
- 2 instancias EC2 (opcional; puedes crearlas durante el lab) con Apache en el puerto 80.

> Nota: En arquitectura “real” el web tier suele ir en **subnets privadas** y el ALB en públicas.  
> Para el lab, las EC2 pueden estar públicas; lo importante es entender el **comportamiento HA** del ALB.

---

# Parte A — Preparar Security Groups (crítico y muy de examen)

## 1) SG del ALB (entrada desde internet)
Crea un Security Group: `sg-alb-web`

**Inbound**
- HTTP 80 desde `0.0.0.0/0`
- (Opcional recomendado) HTTPS 443 desde `0.0.0.0/0` si quisieras TLS

**Outbound**
- Allow all (default)

### Por qué:
- El ALB es el “front door” público.
- Si no abres 80/443, no puedes llegar.

---

## 2) SG de las EC2 (entrada SOLO desde el ALB)
En el SG de tus instancias (o crea uno nuevo `sg-ec2-web`):

**Inbound**
- HTTP 80 **source = sg-alb-web** (NO 0.0.0.0/0)
- SSH 22 desde tu IP (solo si lo necesitas)

### Por qué (esto es importante):
- En un diseño correcto, nadie debería pegarle directo a las instancias: **solo el ALB**.
- Reduce superficie de ataque.
- Te fuerza a pensar en “tiers”: internet → ALB → app.

---

# Parte B — Crear/validar las EC2 (targets)

> Si ya tienes EC2 con Apache funcionando, pasa a la Parte C.

## Opción recomendada: User Data (automatiza y evita errores)
Lanza 2 EC2 (una en cada subnet/AZ) con este User Data.

### EC2-A (AZ 1)
```bash
#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
echo "Hello from EC2-A (AZ1)" > /var/www/html/index.html
```

### EC2-B (AZ 2)
```bash
#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
echo "Hello from EC2-B (AZ2)" > /var/www/html/index.html
```

### Por qué poner mensajes distintos
En un sistema real el contenido debería ser el mismo, pero aquí sirve para:
- Ver que el ALB realmente reparte
- Ver qué target está respondiendo
- Hacer el test de “si cae uno, sigo viendo la web”

---

# Parte C — Crear Target Group (donde viven los backends)

Ve a **EC2 → Target Groups → Create target group**.

## Opciones (y por qué)

### Target type: `Instances`
**Por qué:**
- Estás registrando instancias EC2 directamente.
- Alternativas:
  - `IP` (útil si balanceas a IPs (on-prem, pods, endpoints))
  - `Lambda` (para serverless)

### Protocol / Port: `HTTP : 80`
**Por qué:**
- Apache responde en 80 (lab simple).
- En real, podría ser 443 interno, o app en 8080, etc.

### VPC: tu VPC
**Por qué:**
- El ALB y los targets deben estar en la misma VPC.

### Health checks
- Protocol: HTTP
- Path: `/`
- Matcher: `200` (default suele ser 200–399, según configuración)

**Por qué:**
- El ALB decide “healthy/unhealthy” con esto.
- Si paras `httpd`, el health check falla y el target se marca unhealthy.

## Registrar targets
- Selecciona tus 2 EC2
- Asegúrate de registrarlas en el puerto **80**
- Create target group

### Validación
En el target group → pestaña **Targets**:
- Deben aparecer como **healthy** tras 30–90 segundos.

---

# Parte D — Crear el ALB (Application Load Balancer)

Ve a **EC2 → Load Balancers → Create → Application Load Balancer**.

## Opciones clave (y por qué)

### Scheme: `Internet-facing`
**Por qué:**
- Quieres entrar desde tu navegador.
- `Internal` sería solo para dentro de VPC/peering/VPN.

### IP address type: `IPv4`
**Por qué:**
- Simplicidad para lab. (Dualstack si quieres IPv6 también.)

### Network mapping (subnets)
Selecciona **2 subnets públicas en AZ distintas**.

**Por qué:**
- Esto hace al ALB **Multi-AZ**.
- Si cae una AZ, el ALB sigue disponible en la otra AZ.

### Security group
Asigna `sg-alb-web`.

**Por qué:**
- Es el SG que acepta tráfico del mundo en 80.

## Listener
- Listener: HTTP 80
- Default action: **Forward to** tu Target Group

**Por qué:**
- El listener define el “puerto de entrada”.
- La action determina qué hacer con las requests (forward, redirect, fixed response…).

Crea el ALB.

---

# Parte E — Probar (y ver HA “de verdad”)

## 1) Probar acceso
Copia el **DNS name** del ALB, por ejemplo:
`alb-ha-lab-123456.eu-west-1.elb.amazonaws.com`

Ábrelo en el navegador:
- Deberías ver “Hello from EC2-A…” o “Hello from EC2-B…”

## 2) Ver que balancea
Refresca varias veces:
- debería alternar entre A/B (no siempre 1:1 exacto).

> Nota: si te parece que “se queda” en una, puede ser caching del navegador o keep-alive.  
> Prueba en incógnito o con `curl` repetido.

Ejemplo (desde tu PC o desde otra EC2):
```bash
curl http://<DNS-DEL-ALB>
```

## 3) Simular caída (la prueba HA)
En EC2-A:
```bash
sudo systemctl stop httpd
```

### Qué esperar
- Durante un breve periodo puede aparecer **502** al refrescar:
  - ALB aún no ha marcado el target como unhealthy.
- En 30–90s (depende de interval/threshold) el target se marca **unhealthy** y deja de recibir tráfico.
- Luego, refrescas y solo verás EC2-B (healthy).

### Validación “de examen”
Ve al Target Group → Targets:
- EC2-A pasa a **unhealthy**
- EC2-B sigue **healthy**

**Eso es HA**: el sistema sigue sirviendo sin cambiar DNS ni manualmente tocar rutas.

## 4) Recuperar el servicio
En EC2-A:
```bash
sudo systemctl start httpd
```

Espera a que vuelva a **healthy** y comprobarás que se reincorpora al balanceo.

---

# Parámetros de health check (para entender la “ventana 502”)

En el Target Group (Health checks):

- **Interval** (p. ej. 30s): cada cuánto se comprueba.
- **Unhealthy threshold** (p. ej. 2): cuántos fallos seguidos para marcar unhealthy.
- **Healthy threshold** (p. ej. 5): cuántos éxitos seguidos para volver a healthy.

### Regla mental
Tiempo típico hasta “sacar” un target:
- `Interval × UnhealthyThreshold` (aprox.)

Ejemplo:
- 30s × 2 = ~60s

> Ajustar esto reduce “ventana de error”, pero puede causar “flapping” si tu app es inestable.

---

# Trampas típicas (SAA-C03)

## 1) “Dos EC2 en dos subnets” sin ALB ≠ HA
Sin un componente que:
- distribuya tráfico
- haga health checks
- y saque targets malos
no tienes alta disponibilidad automática.

## 2) Multi-AZ ≠ Multi-Region
- HA intra-región: Multi-AZ + ALB/ASG.
- DR regional: Route 53 failover / backups / warm standby / active-active.

## 3) Seguridad: no abras las EC2 al mundo si tienes ALB
- Inbound 80 en EC2 desde **SG del ALB**, no desde 0.0.0.0/0.

## 4) 502 vs 504 vs 503 (muy rápido)
- **502**: el backend respondió mal o no hubo respuesta HTTP válida.
- **504**: timeout (ALB no recibe respuesta a tiempo).
- **503**: no hay targets healthy (o el LB no puede enrutar).

---

# Extensión opcional (muy recomendable): Auto Scaling Group (ASG) para “HA + self-healing”

> ALB te evita mandar tráfico a un backend caído.  
> **ASG** además puede **reemplazar** instancias para recuperar capacidad.

Patrón de examen:
- ALB + ASG (min=2) en 2 AZ
- Si muere una instancia, ASG crea otra automáticamente

No lo desarrollamos aquí para no alargar, pero si quieres te hago el lab ASG en otro .md.

---

# Limpieza (para no pagar)

1. **Terminate EC2** (las del lab)
2. Delete **ALB**
3. Delete **Target Group**
4. Delete Security Groups (si no los usas)
5. Revisa **EIP** (si asignaste alguna)

---

# Checklist rápida

- [ ] SG del ALB: 80 abierto al mundo
- [ ] SG de EC2: 80 solo desde SG del ALB
- [ ] Target Group HTTP:80 con health check `/`
- [ ] 2 EC2 registradas (healthy)
- [ ] ALB en 2 subnets (2 AZ) + listener 80 → TG
- [ ] Refresco muestra A/B
- [ ] `stop httpd` en una EC2 → target unhealthy → el ALB sirve solo desde la otra

---

## Mini-resumen para memorizar (examen)
- **HA intra-región**: **ALB + targets en 2+ AZ** (y normalmente ASG).
- Health checks = “cerebro” del LB.
- ALB (L7) para HTTP/HTTPS.
- Route 53 failover = más DR que HA (DNS + TTL), no instantáneo.

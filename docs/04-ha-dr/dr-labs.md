# Labs de Disaster Recovery (DR) — SAA-C03 (Backup & Restore + Route 53 Failover)

> Documento “de apuntes + guía práctica” para repetir los **2 labs clave** de DR en AWS:
> 1) **Backup & Restore (Cold DR)** con **EBS snapshots cross-region**
> 2) **Route 53 Failover (Active/Passive)** con **Health Checks + Failover routing**
>
> Enfoque: **entender DR para el examen** (RTO/RPO, decisiones, trampas) y a la vez poder reproducirlo sin dudas.

---

## Objetivos (lo que debes poder explicar al terminar)

### Lab 1 — Backup & Restore
- **Backup ≠ HA** (backup no te deja “sirviendo” automáticamente).
- **DR regional** = backups **fuera de la región** (copia cross-region).
- **RPO** depende de la frecuencia del backup/snapshot.
- **RTO** alto: restauración manual (infra + attach + mount + ajustes).

### Lab 2 — Route 53 Failover
- Route 53 **no balancea tráfico**: decide **qué respuesta DNS** entregar.
- Failover depende de:
  - **Health checks**
  - **TTL**
  - **cachés DNS**
- **DR Active/Passive** típico = Route 53 Failover.
- Failover **NO instantáneo** (a diferencia de otras opciones como Global Accelerator en algunos escenarios).

---

# Lab 1 — DR: Backup & Restore (Cold Recovery) con EBS Snapshot Cross-Region

## Arquitectura (mínima y deliberadamente simple)
- EC2 (Región A) con su **root EBS**
- Snapshot del EBS en Región A
- Copia del snapshot a Región B (la “región de DR”)
- “Desastre”: terminamos EC2 en Región A
- Restore en Región B: volumen desde snapshot + EC2 nueva + attach + mount

```
EC2 (Region A)
 └── EBS (root)  → datos
      └── Snapshot (Region A)
            └── Copy Snapshot (Region B)
                   └── Create Volume (Region B)
                         └── Attach to EC2 (Region B) + Mount + Validate data
```

## Coste y control
- EC2 t3.micro/t2.micro + EBS 8–10GB barato.
- Snapshots cuestan por almacenamiento (incremental).
- Copia cross-region añade coste de almacenamiento en la región destino.
- **Haz cleanup** al final para no dejar nada colgado.

---

## Paso 0 — Pre-requisitos
- Tener permisos para EC2/EBS/Route 53.
- Una key pair (si vas a entrar por SSH).
- Security Group con **SSH** desde tu IP (opcional) y, para el Lab 2, **HTTP 80**.

---

## Parte 1 — Crear “producción” (Región A)
1. Elige una región como **Región A** (ejemplo: `us-east-1` o la que uses de base).
2. Lanza una EC2 Amazon Linux 2023 (t3.micro).
3. Conéctate (SSH) y crea un dato “crítico”:
   ```bash
   echo "VERSION 1 - datos críticos" | sudo tee /home/ec2-user/data.txt
   cat /home/ec2-user/data.txt
   ```

**Qué estás simulando:** datos que no quieres perder.

---

## Parte 2 — Backup: snapshot + copia cross-region
4. Consola EC2 → **Volumes** → selecciona el root volume → **Create snapshot**
   - Descripción: `DR-Lab-Backup-v1`
5. Ve a EC2 → **Snapshots**, selecciona el snapshot y **Copy snapshot** a otra región (Región B).
   - Esto crea el “artefacto de DR”.

**Clave de examen:**  
Si la región A cae, **lo único “útil” para DR regional** es lo que esté **fuera** de la región A (la copia en región B).

---

## Parte 3 — Simular desastre (Región A)
6. “Catástrofe”: **Termina** la EC2 de la región A.
   - (En el mundo real, una caída regional implica que no puedes acceder a recursos en esa región; en consola a veces “siguen ahí”, pero DR asume inaccesibles.)

---

## Parte 4 — Restore en la región B (región de DR)
> Todo lo siguiente se hace en **Región B**.

7. EC2 → Snapshots → selecciona el snapshot copiado (en región B) → **Create volume**
   - Importante: el volumen queda en **una AZ** concreta.
8. Lanza una EC2 nueva en región B **en la misma AZ** que el volumen (para poder adjuntarlo).
9. EC2 → Volumes → selecciona el volumen → **Attach volume** a la EC2 nueva.

---

## Parte 5 — Ver el disco y montarlo (Linux)
10. Entra por SSH a la EC2 nueva y mira los discos:
   ```bash
   lsblk
   ```
   Verás dos discos tipo:
   - `nvme0n1` (root de la EC2 nueva)
   - `nvme1n1` (volumen restaurado del snapshot)

11. Crea un punto de montaje:
   ```bash
   sudo mkdir -p /mnt/dr-volume
   ```

12. **Montaje** (ojo con XFS y UUID duplicado)
   - Muchos root volumes de Amazon Linux están en **XFS**.
   - Si el volumen es un **clon** del root original, puede fallar por UUID duplicado.
   - Solución correcta para este lab:
     ```bash
     sudo mount -o nouuid -t xfs /dev/nvme1n1p1 /mnt/dr-volume
     ```

13. Verifica el archivo restaurado:
   ```bash
   ls /mnt/dr-volume/home/ec2-user/
   cat /mnt/dr-volume/home/ec2-user/data.txt
   ```

### Si te da error al montar: checklist de troubleshooting
- ¿El volumen está adjunto? → `lsblk` lo muestra.
- ¿Qué filesystem es?
  ```bash
  sudo file -s /dev/nvme1n1p1
  ```
- Si en `dmesg` ves:
  `Filesystem has duplicate UUID ... can't mount`
  → usa `-o nouuid`.

**Qué aprendiste (examen):**
- Backup & Restore tiene **RTO alto** porque hay intervención manual.
- RPO depende del último snapshot.
- DR regional requiere **copia cross-region**.

---

## Limpieza (Lab 1)
En región A y B:
- Terminar EC2 creadas
- Borrar volúmenes extra (los que no sean root de instancias que aún existan)
- Borrar snapshots (si no los necesitas)
- Borrar copias cross-region también

---

# Lab 2 — Route 53 Failover (Active/Passive) con Health Checks

## Arquitectura
Dos endpoints HTTP:
- **Primary** (EC2 + Apache)
- **Secondary** (EC2 + Apache)

Route 53:
- 2 **Health Checks** (uno por endpoint)
- 2 **Records A** con política **Failover** (Primary/Secondary)
- TTL bajo (30–60s) para acelerar la conmutación (con trade-off de caché/queries)

```
Client
  ↓ DNS query
Route 53 (Failover)
  ├─ Primary record → IP_PRIMARY (si Healthy)
  └─ Secondary record → IP_SECONDARY (si Primary Unhealthy)
```

---

## Parte 0 — Nota importante (sin dominio propio)
No necesitas comprar dominio.
Crea una **Public Hosted Zone** “fake”, por ejemplo:
- `lab-dr.internal`

No será resoluble públicamente “de forma normal”, pero:
- Puedes consultar directamente a los **Name Servers (NS)** de la hosted zone
- Eso basta para probar el failover.

---

## Parte 1 — Crear endpoints (2 EC2 con Apache)
Crea 2 EC2 (pueden estar en misma región o en dos regiones; el comportamiento DNS es el mismo para el lab).

### Security Group mínimo
- Inbound:
  - HTTP 80 desde `0.0.0.0/0`
  - SSH 22 desde tu IP (opcional)

### Recomendado: User Data (para automatizar)
**PRIMARY** (en User Data):
```bash
#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl start httpd
systemctl enable httpd
echo "PRIMARY - Region A" > /var/www/html/index.html
```

**SECONDARY** (en User Data):
```bash
#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl start httpd
systemctl enable httpd
echo "SECONDARY - Region B" > /var/www/html/index.html
```

Valida por IP pública:
- `http://IP_PRIMARY`
- `http://IP_SECONDARY`

---

## Parte 2 — Hosted Zone
Route 53 → Hosted Zones → Create hosted zone
- Domain name: `lab-dr.internal`
- Type: **Public Hosted Zone**

Guarda los NS, por ejemplo:
- `ns-1282.awsdns-32.org`
- etc.

---

## Parte 3 — Health Checks
Route 53 → Health checks → Create

### Health check PRIMARY
- Name: `hc-primary`
- Endpoint: HTTP
- IP address: **Public IPv4 de la EC2 PRIMARY**
- Port: 80
- Path: `/`
- Interval: 30s
- Failure threshold: 2

### Health check SECONDARY
- Name: `hc-secondary`
- Igual, pero con IP de EC2 SECONDARY

Espera a ver ambos en **Healthy**.

**Trampa típica:**  
Si falla health check:
- SG no permite 80
- Apache no está levantado
- endpoint no es público

---

## Parte 4 — Registros DNS con Failover
En la hosted zone `lab-dr.internal`, crea **dos records A** con el **mismo nombre** `app`:

### Record PRIMARY
- Record name: `app`
- Type: A
- Value: `IP_PRIMARY`
- TTL: 30
- Routing policy: **Failover**
- Failover record type: **Primary**
- Health check: `hc-primary`

### Record SECONDARY
- Record name: `app`
- Type: A
- Value: `IP_SECONDARY`
- TTL: 30
- Routing policy: **Failover**
- Failover record type: **Secondary**
- Health check: `hc-secondary`

---

## Parte 5 — Probar la resolución DNS (Windows sin `dig`)
En Windows usa `nslookup` apuntando al NS de Route 53:

```powershell
nslookup app.lab-dr.internal ns-1282.awsdns-32.org
```

Debe devolverte **IP_PRIMARY** mientras el primary esté healthy.

---

## Parte 6 — Simular caída del Primary y ver failover
1. En la EC2 PRIMARY:
   ```bash
   sudo systemctl stop httpd
   ```
   (o apaga la instancia)

2. Espera 60–120s (health check interval + threshold + TTL).

3. Repite en Windows:
   ```powershell
   nslookup app.lab-dr.internal ns-1282.awsdns-32.org
   ```

Ahora debe devolver **IP_SECONDARY**.

✅ Failover confirmado.

---

## Qué aprendiste (examen)
- Route 53 “enruta” **respuestas DNS**, no tráfico.
- Failover depende de:
  - Health checks
  - TTL
  - caché del cliente / resolver
- “Redirigir tráfico a otra región automáticamente” → **Route 53 Failover**
- “Failover en segundos + IP Anycast estática global” → considera **Global Accelerator** (si el enunciado lo sugiere).

---

## Limpieza (Lab 2)
- Termina ambas EC2
- Borra health checks
- Borra records
- Borra hosted zone “fake”

---

# Checklist rápido (para repetir sin leer todo)

## Lab 1
- [ ] EC2 + data.txt en región A
- [ ] Snapshot en región A
- [ ] Copy snapshot a región B
- [ ] Terminar EC2 región A
- [ ] Create volume desde snapshot en región B
- [ ] EC2 en región B misma AZ
- [ ] Attach volume
- [ ] `lsblk`
- [ ] `mount -o nouuid -t xfs /dev/nvme1n1p1 /mnt/dr-volume`
- [ ] `cat /mnt/dr-volume/home/ec2-user/data.txt`

## Lab 2
- [ ] 2 EC2 con Apache (User Data)
- [ ] Hosted zone `lab-dr.internal`
- [ ] 2 health checks (IP pública)
- [ ] 2 A records Failover (TTL 30) + health check asociado
- [ ] `nslookup app.lab-dr.internal <NS>`
- [ ] Stop httpd primary
- [ ] nslookup cambia a secondary

---

# Mini-resumen “de examen” (para memorizar)
- **Backup & Restore**: barato, simple, **RTO alto**, RPO depende de backups.
- **Pilot Light**: datos + mínimo encendido, escalar bajo demanda.
- **Warm Standby**: entorno reducido siempre activo, RTO menor.
- **Active-Active**: dos regiones activas, RTO/RPO mínimos, caro y complejo.
- **Route 53 Failover**: DR Active/Passive por DNS (no instantáneo).

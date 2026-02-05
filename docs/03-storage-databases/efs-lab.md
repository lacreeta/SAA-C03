# EFS LAB — Filesystem compartido multi-AZ (SAA-C03)

## Objetivo del laboratorio

Entender qué es EFS y cuándo usarlo desde una perspectiva de arquitectura AWS (SAA-C03).

---

## Arquitectura del lab

VPC
- Subnet A (AZ-a)
  - EC2-A
- Subnet B (AZ-b)
  - EC2-B
- EFS (regional, multi-AZ)

---

## Prerrequisitos

- VPC existente
- 2 subnets en AZ distintas
- 2 EC2 Linux
- Acceso SSH

---

## Paso 1 — Crear EFS

- EFS → Create file system
- VPC: la del lab
- Configuración por defecto

---

## Paso 2 — Security Group del EFS

Inbound:
- NFS (2049)
- Source: SG de las EC2

Outbound:
- Allow all

---

## Paso 3 — Montaje en las EC2

En ambas EC2:

```bash
sudo yum install -y amazon-efs-utils
sudo mkdir -p /mnt/efs
sudo mount -t efs <efs-id>:/ /mnt/efs
```

---

## Paso 4 — Prueba

EC2-A:
```bash
echo "hola desde A" > /mnt/efs/test.txt
```

EC2-B:
```bash
cat /mnt/efs/test.txt
```

---

## Trampas de examen

- EFS es compartido
- Multi-AZ
- No es EBS
- No es S3

---

## Limpieza

- Eliminar EFS
- Terminar una EC2

---

## Conclusión

EFS es la solución correcta para filesystem compartido en Linux multi-AZ.


## Laboratorio (solo EFS)
👉 [EFS — Laboratorio](efs-lab.md)
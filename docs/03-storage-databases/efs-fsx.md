# EFS y FSx — SAA-C03

> Este documento cubre **EFS** (NFS gestionado) y **FSx** (file systems especializados). El examen suele preguntar **cuándo usar cada uno**.

---

# 1) EFS (Elastic File System)

## Qué es
**Amazon EFS** es un sistema de archivos **NFS** totalmente gestionado:
- Filesystem compartido
- Acceso concurrente desde múltiples instancias/containers
- Orientado a Linux (NFS)

📌 Examen: cuando necesitas **shared file storage** para **varios servidores** en múltiples AZ.

---

## Características clave
- **Regional (multi-AZ)**
- Se monta desde:
  - EC2
  - ECS/EKS (cuando aplica)
  - On-prem (si hay conectividad)
- **Escala automáticamente** (capacidad)

### Caso clásico
- App en varias instancias que necesita leer/escribir el **mismo** directorio de uploads.
➡️ **EFS**

---

## Cuándo usar EFS
- “Filesystem compartido para instancias Linux”
- “Varios servidores en distintas AZ deben acceder a los mismos ficheros”
- “No quiero administrar NFS servers”

---

## Cuándo NO usar EFS
- Windows file shares (SMB)
- HPC extremo (mejor FSx for Lustre)
- Si realmente lo que quieres es objetos (S3) y no un filesystem POSIX

---

## Seguridad (alto nivel)
- Mount targets en subnets
- Control por Security Groups
- Cifrado en reposo con KMS (si se habilita)
- Cifrado en tránsito (TLS) según configuración

---

# 2) FSx (File Systems especializados)

## Idea central
**FSx** es una familia de file systems para necesidades específicas donde EFS no encaja.

---

## 2.1 FSx for Windows File Server (SMB)
### Qué es
File share para **Windows** (SMB), con integración típica con **Active Directory**.

### Cuándo usar (examen)
- “Necesito un file share SMB para Windows servers”
- “Aplicación Windows requiere file share compartido”
➡️ **FSx for Windows File Server**

---

## 2.2 FSx for Lustre (HPC)
### Qué es
File system de **alto rendimiento** para workloads HPC (throughput/IO muy alto).

### Cuándo usar (examen)
- “Simulaciones / HPC / ML con throughput altísimo”
- “Quiero un filesystem rápido acoplado a datasets en S3”
➡️ **FSx for Lustre**

---

## 2.3 FSx for NetApp ONTAP
### Qué es
File system basado en **NetApp ONTAP** (features enterprise: snapshots/clones, migraciones, etc.).

### Cuándo usar (examen)
- “Migración desde NetApp ONTAP”
- “Necesito features enterprise específicas de ONTAP”
➡️ **FSx for ONTAP**

---

## 2.4 FSx for OpenZFS
### Qué es
File system basado en **OpenZFS**, útil para compatibilidad/migración desde ZFS.
➡️ **FSx for OpenZFS**

---

# 3) Cómo elegir rápido (reglas de examen)
- **EFS**: Linux + NFS + multi-AZ + compartido
- **FSx Windows**: Windows + SMB + AD
- **FSx Lustre**: HPC + máximo rendimiento
- **FSx ONTAP/OpenZFS**: migraciones / necesidades específicas

---

# 4) Trampas típicas
- **S3 no es filesystem**
- **EBS no es shared** como filesystem típico entre múltiples instancias (en preguntas de “shared storage” suele ser trampa)
- “Windows file share” → no es EFS

---

# 5) Mini-resumen
- **EFS**: NFS para Linux, compartido, multi-AZ.
- **FSx Windows**: SMB para Windows.
- **FSx Lustre**: HPC, ultra rendimiento.
- **FSx ONTAP/OpenZFS**: enterprise/migración.

## Laboratorio (solo EFS)
👉 [EFS — Laboratorio](efs-lab.md)
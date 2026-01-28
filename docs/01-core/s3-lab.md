## S3 — Labs prácticos

## Fecha

27-01-2026

---

## Lab 1 — Creación de bucket y objetos

En este primer laboratorio se creó un **bucket S3** y se subieron varios ficheros, que en S3 se denominan **objetos**.

### Observaciones clave

* Un **bucket** es el contenedor lógico de los objetos (no equivale exactamente a un disco duro, sino a un contenedor de object storage).
* Los **objetos** son los ficheros almacenados dentro del bucket, identificados por una *object key*.
* El bucket se crea en una **región concreta**, aunque S3 sea un servicio regional y altamente distribuido internamente entre múltiples AZs.

### Conceptos reforzados

* S3 es **object storage**, no block storage ni filesystem.
* No se elige AZ: la durabilidad y replicación están gestionadas por AWS.

---

## Lab 2 — Versioning

En este laboratorio se activó el **versioning** del bucket.

### Pasos realizados

* Activación de versioning.
* Subida repetida del mismo objeto.
* Eliminación del objeto.
* Visualización de versiones usando la opción **Show versions**.

### Observaciones clave

* Al borrar un objeto versionado, **no se elimina realmente**, sino que se crea un **delete marker**.
* Las versiones anteriores del objeto siguen existiendo y pueden recuperarse.

### Implicación de examen

* Versioning protege contra borrados accidentales.
* Para eliminar definitivamente un objeto versionado es necesario borrar **todas sus versiones**.

---

## Lab 3 — Block Public Access y acceso público

En este laboratorio se exploró el funcionamiento de **Block Public Access**.

### Pasos realizados

* Se intentó hacer el bucket público manteniendo Block Public Access habilitado.
* AWS no permitió el acceso público.
* Se desactivó Block Public Access.
* Se pudo acceder a los objetos mediante su URL pública.

### Conclusión clave

> Aunque una bucket policy permita acceso público, si **Block Public Access está habilitado**, el acceso público sigue estando bloqueado.

### Conceptos reforzados

* Block Public Access actúa como una **capa de seguridad superior** a bucket policies y ACLs.
* El acceso público en S3 se controla mediante bucket policies, pero puede ser completamente anulado por Block Public Access.

---

## Lab 4 — Bucket privado con acceso controlado (IAM + Bucket Policy)

En este laboratorio se configuró un acceso **privado y controlado** a un bucket S3.

### Escenario

* Bucket privado.
* Block Public Access habilitado.
* Acceso concedido únicamente a un **IAM user específico**.

### Pasos realizados

* Creación de un IAM user sin permisos.
* Asignación de una **policy mínima custom** para permitir listar buckets (`s3:ListAllMyBuckets`).
* Creación de una **bucket policy (resource-based)** que permite únicamente:

  * `s3:ListBucket`
  * `s3:GetObject`

### Resultado

* El usuario IAM puede ver el bucket.
* Puede listar y leer los objetos.
* No puede subir ni borrar objetos.
* Ningún otro usuario tiene acceso.

### Conceptos reforzados

* Diferencia entre **IAM policies (identity-based)** y **bucket policies (resource-based)**.
* Separación entre permisos para listar recursos y permisos para acceder a su contenido.
* Aplicación del principio de **least privilege**.

### Implicación de examen

> Que un usuario no vea un bucket en la consola no significa que no tenga acceso a los objetos, y viceversa.

---

## Lab 5 — Lifecycle Rules (optimización de costes)

En este laboratorio se configuraron **Lifecycle Rules** para optimizar costes de almacenamiento.

### Configuración aplicada

* Día 0: los objetos se almacenan en **S3 Standard**.
* Día 30: transición automática a **S3 Standard-IA**.
* Día 90: transición automática a **Glacier Flexible Retrieval**.
* Día 365: expiración automática de los objetos.

### Conceptos reforzados

* Lifecycle rules permiten **automatizar el paso del tiempo** sobre los datos.
* No requieren cambios en la aplicación.
* Son clave para optimización de costes en datos con acceso decreciente.

### Implicación de examen

* Muy común en preguntas de reducción de costes.
* Lifecycle ≠ backup ≠ replicación.

---

## Lab 6 — Static Website Hosting

En este laboratorio se configuró un bucket S3 para servir una **web estática**.

### Pasos realizados

* Subida de ficheros estáticos (HTML).
* Activación de **Static Website Hosting**.
* Desactivación de Block Public Access.
* Creación de una bucket policy pública de solo lectura (`s3:GetObject`).

### Resultado

* El contenido se sirve directamente desde S3 mediante un endpoint web.
* No se utilizan servidores ni instancias EC2.

### Conceptos reforzados

* S3 puede actuar como **frontend** para contenido estático.
* No soporta backend ni ejecución de código.
* Para HTTPS y distribución global se requiere **CloudFront**.

### Implicación de examen

> Web estática barata, altamente disponible y sin servidores → S3 Static Website Hosting.

---

## Conclusiones generales

Estos laboratorios cubren los aspectos clave de S3 relevantes para el examen **AWS SAA-C03**:

* Almacenamiento duradero y escalable.
* Seguridad por capas (IAM, bucket policies, Block Public Access).
* Optimización de costes mediante lifecycle rules.
* Casos de uso reales de arquitectura.

S3 es un servicio fundamental en AWS y aparece de forma recurrente en preguntas de diseño y toma de decisiones arquitectónicas.

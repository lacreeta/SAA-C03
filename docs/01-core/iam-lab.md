## 23-01-2026

### Objetivo

Aplicar buenas prácticas de **IAM**: no usar el usuario root, entender la diferencia entre **autenticación y autorización**, y aplicar el **principio de mínimo privilegio (least privilege)**.

---

### Usuario IAM sin credenciales

* Un usuario IAM recién creado **no tiene credenciales por defecto**.
* Sin *AWS Management Console access* (username + password) **no puede iniciar sesión**.
* El error ocurre por **autenticación**, no por permisos.

**Conclusión clave:**
Un usuario IAM no puede acceder a la consola si no se configuran explícitamente credenciales de consola.

---

### Edición de usuarios IAM

* Un usuario IAM **sí puede editarse desde la consola**:

  * Añadir o quitar permisos
  * Activar/desactivar acceso a consola
  * Gestionar access keys
  * Activar MFA
* **Renombrar un usuario IAM no está soportado en la consola**.

  * Solo es posible mediante **AWS CLI, API o PowerShell**.

**Implicación práctica:**
No es necesario borrar un usuario para añadir credenciales o permisos; solo para cambiar su nombre.

---

### Usuario IAM con acceso a consola pero sin permisos

* Usuario creado con *AWS Management Console access*.
* Puede iniciar sesión correctamente.
* No puede acceder ni utilizar ningún recurso.
* Intento de crear una instancia EC2 → **AccessDenied**.

**Conclusión clave:**
Autenticación correcta ≠ autorización para usar recursos.

---

### Uso de grupos (Groups)

* Se creó un **IAM Group** para asignar permisos sin modificar directamente al usuario.
* Al añadir el usuario al grupo, **heredó los permisos**.
* Al eliminarlo del grupo, **perdió el acceso inmediatamente**.

**Buenas prácticas:**

* Los permisos a usuarios se gestionan preferiblemente mediante **groups**.

---

### Policy restringida (Least Privilege)

* Se creó una policy personalizada que permite solo:

  * `s3:GetObject`
  * En **un bucket S3 específico**
* El usuario:

  * Puede descargar objetos existentes.
  * No puede subir, borrar objetos ni crear buckets.

**Conclusión clave:**
Restringir una policy implica **limitar acciones y recursos**, no solo quitar permisos.

---

### Deny explícito

* Se añadió una segunda policy con:

  * `Effect: Deny`
  * `s3:GetObject` sobre el mismo bucket
* A pesar de existir una policy `Allow`, el acceso fue denegado.

**Regla clave de IAM:**
Un **Deny explícito siempre tiene prioridad** sobre cualquier Allow.

---

### Resumen de aprendizaje

* Los usuarios IAM no tienen permisos ni credenciales por defecto.
* Login (autenticación) y permisos (autorización) son conceptos distintos.
* Los grupos facilitan la gestión de permisos.
* El principio de mínimo privilegio es fundamental.
* Un Deny explícito invalida cualquier Allow.

# Security Labs --- AWS SAA-C03 (Free Tier Safe)

> Objetivo: consolidar los conceptos más preguntados de seguridad en el
> SAA-C03. Enfoque práctico + mentalidad de examen. Todos los labs están
> diseñados para ser seguros dentro del Free Tier.

------------------------------------------------------------------------

# LAB 1 --- IAM Profundo (Users, Roles, Policies, Deny)

## Objetivo Arquitectónico

Entender realmente:

-   Identity-based policy vs Resource-based policy
-   Explicit Deny \> Allow
-   Users vs Roles
-   AssumeRole
-   Principio de least privilege

------------------------------------------------------------------------

## Parte 1 --- Users y Policies

### Qué harás

-   Crear un IAM User sin permisos.
-   Crear una policy custom que permita solo listar buckets S3.
-   Adjuntar la policy al usuario.
-   Verificar acceso.

### Qué debes entender

-   IAM es global.
-   Las policies identity-based se adjuntan a usuarios/roles.
-   Si no hay Allow explícito → acceso denegado.

------------------------------------------------------------------------

## Parte 2 --- Explicit Deny

### Qué harás

-   Crear una policy que permita acceso S3 completo.
-   Añadir una policy adicional con Explicit Deny sobre un bucket
    específico.
-   Probar acceso.

### Qué debes entender

-   Explicit Deny siempre gana.
-   AWS evalúa todas las policies juntas.

------------------------------------------------------------------------

## Parte 3 --- Roles y AssumeRole

### Qué harás

-   Crear un IAM Role para EC2 o Lambda.
-   Entender la Trust Policy.
-   Probar asumir el role desde otro usuario.

### Qué debes entender

-   Roles se usan para servicios.
-   STS permite AssumeRole temporal.
-   No se deben usar Access Keys en producción cuando existe Role.

------------------------------------------------------------------------

## Trampas de examen

-   SG ≠ IAM
-   Resource-based policy puede otorgar acceso cross-account.
-   Falta de permiso ≠ Deny explícito.
-   Root user no debe usarse.

------------------------------------------------------------------------

# LAB 2 --- KMS + S3 (Cifrado real)

## Objetivo Arquitectónico

Comprender:

-   AWS-managed key vs Customer-managed key
-   SSE-S3 vs SSE-KMS
-   Key policy
-   Rotación automática

------------------------------------------------------------------------

## Parte 1 --- Crear CMK

### Qué harás

-   Crear una Customer Managed Key.
-   Activar rotación automática.
-   Revisar Key Policy.

### Qué debes entender

-   KMS no guarda datos.
-   KMS protege claves, no objetos.
-   Las key policies controlan acceso a la clave.

------------------------------------------------------------------------

## Parte 2 --- Cifrar objeto en S3 con SSE-KMS

### Qué harás

-   Crear bucket S3.
-   Subir objeto usando SSE-KMS.
-   Seleccionar tu CMK.

### Qué debes observar

-   El objeto indica "AWS KMS" como método de cifrado.
-   Si el usuario no tiene permiso kms:Decrypt → no podrá leer.

------------------------------------------------------------------------

## Trampas de examen

-   AWS-managed keys no se pueden modificar.
-   CMK permite control fino y cross-account.
-   Si falla el acceso, puede ser problema de key policy y no de IAM.

------------------------------------------------------------------------

# LAB 3 --- Secrets Manager vs SSM Parameter Store

## Objetivo Arquitectónico

Comprender diferencias reales entre:

-   Secrets Manager
-   SSM Parameter Store

------------------------------------------------------------------------

## Parte 1 --- Crear parámetro en SSM

### Qué harás

-   Crear parámetro tipo SecureString.
-   Asociarlo a KMS.

### Qué debes entender

-   SSM es más barato.
-   No tiene rotación automática integrada.
-   Bueno para configuración simple.

------------------------------------------------------------------------

## Parte 2 --- Crear secreto en Secrets Manager

### Qué harás

-   Crear secreto con credenciales simuladas.
-   Ver opciones de rotación.

### Qué debes entender

-   Secrets Manager soporta rotación automática.
-   Más caro que SSM.
-   Mejor integración con RDS.

------------------------------------------------------------------------

## Comparativa Mental

  Servicio              Cuándo usar
  --------------------- --------------------------------------------
  SSM Parameter Store   Configuración simple, bajo coste
  Secrets Manager       Credenciales críticas, rotación automática

------------------------------------------------------------------------

## Trampas de examen

-   Rotación automática → Secrets Manager.
-   Acceso a secretos lo controla IAM.
-   Secrets Manager es regional.
-   Ambos pueden usar KMS.

------------------------------------------------------------------------

# Resumen Final Security

Si el enunciado habla de:

-   Least privilege → IAM
-   Control fino de cifrado → KMS CMK
-   Rotación automática → Secrets Manager
-   Configuración barata → SSM
-   Cross-account acceso → resource-based policy o KMS key policy

------------------------------------------------------------------------

Estos 3 labs cubren el núcleo real de seguridad para SAA-C03.

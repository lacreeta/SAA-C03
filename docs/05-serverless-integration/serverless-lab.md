# Serverless --- Labs Prácticos (SAA-C03)

> Objetivo: entender cómo se conectan los servicios serverless y cuándo
> elegir cada uno. Enfoque orientado a arquitectura y examen.

------------------------------------------------------------------------

# LAB 1 --- SNS + SQS + Lambda

## Fan-out, desacoplamiento, retries y DLQ

## Objetivo

Comprender: - SNS como pub/sub - SQS como buffer desacoplado - Lambda
como consumidor - Retries automáticos - DLQ - Flujo ante fallos

## Arquitectura

Productor → SNS → SQS → Lambda → DLQ

## Qué está ocurriendo realmente

1.  SNS distribuye el mensaje.
2.  SQS lo almacena de forma duradera.
3.  Lambda hace polling.
4.  Si falla:
    -   El mensaje vuelve a estar disponible.
    -   Tras X intentos → va a DLQ.

## Casos de uso reales

-   Procesamiento asíncrono
-   Microservicios desacoplados
-   Fan-out de eventos
-   Procesamiento en segundo plano

## Trampas de examen

-   SNS no es una cola.
-   SQS no escucha eventos AWS automáticamente.
-   Lambda necesita permisos SQS.
-   Visibility timeout mal configurado = duplicados.

------------------------------------------------------------------------

# LAB 2 --- EventBridge

## Automatización basada en eventos AWS

## Objetivo

Entender cuándo usar EventBridge en vez de SQS o SNS.

## Arquitectura

Evento AWS → EventBridge Rule → Target

## Qué ocurre realmente

AWS genera eventos automáticamente (ej: cambio estado EC2). EventBridge
los filtra y los enruta.

No hay polling. No hay productor manual.

## Filtrado específico (ejemplo)

``` json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "instance-id": ["i-1234567890abcdef0"],
    "state": ["stopped"]
  }
}
```

## Casos de uso

-   Automatización operativa
-   Auditoría reactiva
-   Seguridad
-   Integraciones entre servicios AWS

## Trampas de examen

-   EventBridge es regional.
-   No modifica mensajes.
-   No es un sistema de almacenamiento.
-   Para transformar mensaje → usar Lambda intermedia.

------------------------------------------------------------------------

# LAB 3 --- Step Functions

## Orquestación y control de flujo

## Objetivo

Entender la diferencia entre orquestación y simple disparo de eventos.

## Arquitectura

Lambda A → Lambda B\
Con Retry y Catch

## Qué ocurre realmente

-   Step Functions controla el orden.
-   Gestiona reintentos.
-   Maneja errores centralmente.
-   Permite visualizar el flujo.

## Conceptos clave

-   Retry configurable
-   Backoff exponencial
-   Catch para manejar fallos
-   Flujo secuencial controlado

## Casos de uso reales

-   Procesos de negocio con múltiples pasos
-   Validaciones secuenciales
-   Workflows complejos
-   Procesamiento con lógica condicional

## Trampas de examen

-   Step Functions ≠ SNS
-   Step Functions ≠ EventBridge
-   No es solo disparar funciones, es controlar el flujo.

------------------------------------------------------------------------

# Comparativa Mental Final

  Servicio         Rol Principal
  ---------------- ----------------------
  SNS              Distribuir
  SQS              Desacoplar
  Lambda           Ejecutar lógica
  EventBridge      Detectar eventos AWS
  Step Functions   Orquestar

------------------------------------------------------------------------

Si entiendes cuándo usar cada uno y por qué, dominas el núcleo
serverless del SAA-C03.

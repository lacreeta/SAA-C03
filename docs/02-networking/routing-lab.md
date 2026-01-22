# Routing, IGW & NAT Gateway Lab

## Fecha
22-01-2026

## Objetivo
Entender cómo funcionan las **subnets públicas y privadas**, el **Internet Gateway (IGW)** y la **NAT Gateway**, y cómo el *routing* determina realmente la conectividad a internet en una VPC.

---

## Análisis de la VPC por defecto

Comencé analizando la **VPC por defecto** y sus subnets:

- La VPC tenía **3 subnets**, todas ellas **públicas**
- Todas estaban asociadas a una **route table** con la ruta:
  - 0.0.0.0/0 → Internet Gateway (IGW)
- Además, tenían **auto-assign public IPv4 ENABLED**

**Conclusión:**  
Una subnet es pública **porque su route table tiene una ruta hacia un IGW**, no por el auto-assign public IPv4.

---

## Creación de una subnet privada

- Creé una nueva subnet con **auto-assign public IPv4 DISABLED**
- A pesar de ello, la subnet seguía siendo pública porque:
  - Estaba asociada a la **route table por defecto**, que tenía `0.0.0.0/0 → IGW`

Para corregirlo:

- Creé una **nueva route table**
- Eliminé cualquier ruta hacia el IGW
- Asocié esta route table a la subnet creada

**Resultado:**  
La subnet pasó a ser **realmente privada**.

---

## Creación y uso de la NAT Gateway

- Creé una **NAT Gateway**:
  - Ubicada en una **subnet pública**
  - Asociada a una **Elastic IP**
  - Actualicé la route table de la subnet privada con: 
    - 0.0.0.0/0 → NAT Gateway

**Comportamiento esperado:**

- Las instancias en la subnet privada pueden **iniciar tráfico hacia internet**
- No pueden **recibir tráfico entrante iniciado desde internet**
- La NAT Gateway realiza **NAT de origen**, usando su Elastic IP para el tráfico saliente

---

## Pruebas con EC2 en subnet privada

- Creé una **instancia EC2** en la subnet privada
- Intenté conectarme por **SSH directamente**, sin éxito (comportamiento esperado)
- Verifiqué que la NAT Gateway funcionaba correctamente

### Acceso a la instancia

- Utilicé **EC2 Instance Connect Endpoint**
- Para ello:
  - Creé el endpoint usando el mismo **Security Group** que la EC2
  - Modifiqué la **Inbound Rule de SSH** para permitir acceso

Finalmente:

- Pude acceder a la instancia
- Desde la EC2, realicé pruebas de conectividad (ping/salida a internet)
- La salida a internet funcionó correctamente

---

## Limpieza de recursos

Tras finalizar las pruebas:

- Eliminé la NAT Gateway
- Eliminé subnets y route tables creadas
- Dejé la VPC en su estado por defecto

**Objetivo:** evitar costes innecesarios.

---

## Key Takeaways (Examen)

- Una subnet es pública o privada **según su route table**
- `auto-assign public IPv4` **no define** si una subnet es pública
- El **IGW** permite tráfico directo entre la VPC e internet
- La **NAT Gateway** permite solo **tráfico saliente iniciado desde subnets privadas**
- Las instancias en subnets privadas **no aceptan tráfico entrante desde internet**

# 📈 E-commerce Data Engineering: Microsoft Fabric Architecture

> **Nota de Confidencialidad:**  
> Este proyecto fue desarrollado para cliente de e-commerce bajo acuerdo 
> de confidencialidad. Los datos, nombres y métricas específicas han sido 
> anonimizados. La arquitectura técnica y metodología mostradas reflejan 
> la implementación real.

[![View Dashboard](https://img.shields.io/badge/View-Live_Dashboard-blue)](https://app.powerbi.com/view?r=eyJrIjoiMzJiODdjNTAtYmZiNS00NTM0LWEwZTQtODg1ZGU3NzYwMWI1IiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9)
[![Fabric](https://img.shields.io/badge/Microsoft-Fabric-purple)]()
[![Power BI](https://img.shields.io/badge/Power-BI-yellow)]()

Este proyecto implementó una solución de **BI end-to-end** utilizando **Microsoft Fabric**. El objetivo principal fue transformar datos transaccionales dispersos —alojados en **Supabase (PostgreSQL)**— en una arquitectura de **Lakehouse** optimizada para el cálculo de rentabilidad real mediante un enfoque de **Arquitectura Medallion**.

---

## 🔧 Sistema Completo Implementado

Este proyecto no es solo el pipeline Fabric. Es una solución end-to-end que incluye:

**1. Frontend Transaccional**
- Checkout funcional que captura: producto, peso, dimensiones, dirección completa

**2. Automatización de Decisiones (n8n)**
- Orquestador que evalúa múltiples proveedores logísticos en tiempo real
- Calcula costo real considerando zona, peso y tiempo de entrega
- Aplica reglas de negocio para selección óptima
- Guarda decisión en base transaccional

**3. Base Transaccional (Supabase PostgreSQL)**
- Almacena: pedidos, catálogo de proveedores, decisiones logísticas, devoluciones

**4. Pipeline Analytics (Microsoft Fabric)** ← Enfoque principal de este README
- Arquitectura medallion (Bronze/Silver/Gold)
- Transformaciones con Spark SQL
- Modelo estrella optimizado

**5. Dashboards Ejecutivos (Power BI)**
- Visualización de rentabilidad real
- Análisis comparativo de proveedores
- KPIs operativos

---

## 📊 Resultados Obtenidos

### Impacto Operativo
- **Automatización completa:** Decisiones de paquetería en <2 seg vs 30 min manuales
- **Visibilidad total:** Primera vez calculando margen neto real por producto/zona
- **Pipeline confiable:** Orquestación automatizada con Data Factory

### Impacto Financiero
- **23% reducción** en costos de envío
- **Identificación de productos no rentables** después de considerar costos logísticos completos
- **Visibilidad de costos ocultos:** Peso volumétrico, zonas extendidas, devoluciones

### Insights Estratégicos
- Zona sureste 40% más cara → Cliente ajustó estrategia de pricing
- Peso volumétrico impactaba 35% de envíos (no se calculaba antes)
- Proveedor "habitual" no era el más económico en 60% de casos

---

## El Problema de Negocio

Las E-commerce (Pymes) suelen operar con una visión parcial de su salud financiera debido a:

* **Datos Fragmentados:** Información dispersa entre diversas plataformas de venta, ERPs y operadores logísticos.
* **Inconsistencia de Tipos:** Datos numéricos que ingresaban como texto (`String`), bloqueando cualquier análisis de agregación.
* **Costos Ocultos:** Incapacidad de integrar devoluciones, comisiones de pasarelas y gastos de última milla en el cálculo del margen bruto y neto.

---

## Arquitectura de Datos (Modern ELT)

A diferencia del ETL tradicional, se desarrolló un flujo **ELT** (Extract, Load, Transform) aprovechando el poder de procesamiento de **Microsoft Fabric** y el almacenamiento unificado en **OneLake**.

### Diagrama de Arquitectura

<img width="1917" height="746" alt="Arquitectura completa" src="https://github.com/user-attachments/assets/a274f752-a44e-4f56-b562-bbe40d112c74" />
<img width="1904" height="899" alt="Pipeline Fabric" src="https://github.com/user-attachments/assets/22b43c7a-f93e-49fd-b31a-cd59e7f535be" />

### Capas del Lakehouse:

**1. Capa Bronze (Raw data)**
- Conexión SQL desde **Supabase** mediante **Notebooks**
- Datos se mantienen en formato original sin transformaciones
- Ingesta automatizada vía Data Factory

**2. Capa Silver (Cleaned & Validated)**
- Normalización de esquemas
- Eliminación de registros duplicados
- **Solución de tipado:** Conversión de campos numéricos tipo `String` a `Decimal` mediante funciones de limpieza
- Validación de integridad referencial

**3. Capa Gold (Business Layer)**
- **Modelo en Estrella (Star Schema)** creado con Notebooks **Spark SQL** (`%%sql`)
- Tablas de hechos y dimensiones optimizadas
- Servido mediante **Direct Lake**: Power BI consume archivos Parquet en OneLake sin importar datos
- Garantiza latencia mínima y sincronización automática

---

## Orquestación y Automatización (Data Factory)

Para garantizar la actualización constante de los datos, se configuró un **Data Factory Pipeline** que:

- Actúa como orquestador central
- Automatiza la ingesta desde Supabase
- Ejecuta secuencialmente los Notebooks de transformación (Bronze → Silver → Gold)
- Programa refreshes según necesidades del negocio

<img width="1904" height="731" alt="Data Factory Pipeline" src="https://github.com/user-attachments/assets/fa0dc0e8-53f6-40cd-8150-b2a32282e7ef" />

---

## Modelo de Datos Optimizado

El diseño del modelo se estructuró utilizando:
- **Tabla de Hechos:** FactVentas (grano: pedido individual)
- **Dimensiones:** Producto, Cliente, Zona, Proveedor, Tiempo

<img width="1893" height="862" alt="Modelo Estrella" src="https://github.com/user-attachments/assets/427ef38f-2693-4fab-839a-964ebd3fb882" />

### Solución de Ingeniería: Tipado de Datos

**Problema:** Campos numéricos (precios, costos, pesos) ingresaban como `String` desde Supabase, bloqueando agregaciones en Power BI.

**Solución:** Durante la transformación en la capa **Silver**, se integró script que:
- Utilizó funciones de reemplazo (`regexp_replace`) para eliminar caracteres no numéricos
- Aplicó re-tipado forzado al esquema de datos correcto
- Aseguró compatibilidad con medidas DAX de time intelligence y cálculos de margen

Esto garantizó que el motor de Power BI pudiera ejecutar cálculos complejos sin errores de compatibilidad.

---

## Estrategia de Consumo y Optimización de Costos

Para maximizar la eficiencia operativa y reducir costos de licenciamiento:

* **Modelo Semántico Centralizado:** Publicado en servicio de Fabric con configuración Direct Lake
* **Consumo Local (Power BI Desktop):** Para fines de demostración, se utilizó Power BI Desktop conectado al **Modelo Semántico** del medallion
* **Beneficio:** Permite diseñar reportes sin consumir capacidad de Fabric en cada cambio visual, optimizando costos de licencia

---

## Tecnologías Utilizadas

| Componente | Tecnología | Propósito |
|-----------|-----------|-----------|
| **Frontend** | HTML/CSS/JS | Checkout transaccional |
| **Automatización** | n8n + Python | Orquestación decisiones logísticas |
| **Transaccional** | Supabase (PostgreSQL) | Base de datos OLTP |
| **Ingeniería Datos** | Microsoft Fabric | Pipeline ELT y orquestación |
| **Procesamiento** | Spark SQL (Notebooks) | Transformaciones Silver/Gold |
| **Storage** | OneLake (Delta/Parquet) | Lakehouse unificado |
| **Modelado** | Star Schema | Optimización queries |
| **Visualización** | Power BI + DAX | Dashboards ejecutivos |
| **Orquestación** | Data Factory Pipelines | Automatización ETL |

---

## Dashboards Interactivos

Ver reporte completo 👉 [Dashboard de Rentabilidad](https://app.powerbi.com/view?r=eyJrIjoiMzJiODdjNTAtYmZiNS00NTM0LWEwZTQtODg1ZGU3NzYwMWI1IiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9)

<img width="1491" height="827" alt="Dashboard 1" src="https://github.com/user-attachments/assets/2297200e-4cac-4d55-bca6-d046c2e2beab" /> 
<img width="1508" height="822" alt="Dashboard 2" src="https://github.com/user-attachments/assets/f4dc5cc5-4e79-4b4d-bf64-17aeef9bc9aa" />
<img width="1493" height="841" alt="Dashboard 3" src="https://github.com/user-attachments/assets/0474aa6d-a7fe-4c5d-a14f-b5e88bf9be4a" />

---

## ¿En qué situaciones se podría implementar?

Este tipo de arquitectura de datos (Modern ELT) es ideal para organizaciones que:

* **Manejan volúmenes significativos de datos de ventas** que crecen continuamente y requieren procesamiento escalable
* **Necesitan realizar análisis de rentabilidad complejos** sobre datos históricos y actuales
* **Buscan una "fuente única de verdad"** para eliminar discrepancias entre reportes de finanzas, ventas y logística
* **Quieren desacoplar las cargas de trabajo analíticas** de sus sistemas transaccionales para no afectar el rendimiento operativo
* **Desean optimizar costos de licencia**, centralizando el modelo en Fabric y consumiéndolo localmente para diseño de reportes
* **Negocios que requieren migrar de Power BI Service** hacia la capacidad y potencia analítica de **Microsoft Fabric**
* **E-commerce o retail** con múltiples canales de venta y proveedores logísticos

---

## 📧 Contacto

**Ernesto Roldán**  
Fabric Analytics Engineer | DP-600, PL-300 & PL-200 Certified

📧 [Contacto](mailto:tu@email.com)  
💼 [LinkedIn](https://linkedin.com/in/tu-perfil)  
🌐 [ieoanalytics.com](https://ieoanalytics.com)

---

**¿Necesitas implementar arquitectura similar para tu negocio?**  
Contáctame para consultoría personalizada.
```

---

---

---

## **POST LINKEDIN (Híbrido: Negocio + Técnico)**
```
Tres sistemas separados.
Cero visibilidad del costo real.

Un e-commerce de electrónica operaba así:
→ Ventas en un sistema
→ Logística en Excel
→ Costos... nadie los calculaba completos

Resultado:
Pensaban tener 35% de margen.
Realidad: 12% después de todos los costos.

La diferencia: $18,000-25,000 MXN mensuales 
que se perdían sin que nadie lo supiera.

El problema no era vender poco.
Era no saber cuánto costaba realmente cada venta.

Construí sistema end-to-end que resuelve esto:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PARTE 1: AUTOMATIZACIÓN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Cliente compra laptop 3.5kg con envío a Oaxaca.

Sistema evalúa 8 proveedores en 2 segundos:
- DHL: $225 (3 días)
- Redpack: $185 (5 días) ← Elige este
- Estafeta: $210 (4 días)
- 99minutos: No disponible
- [+ 4 más]

Decisión automática.
Sin intervención humana.
Guarda todo en PostgreSQL.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PARTE 2: INTEGRACIÓN DE DATOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Aquí es donde se pone técnico.

Construí pipeline en Microsoft Fabric con arquitectura medallion:

🔶 BRONZE: Ingesta raw desde Supabase
   - Datos en formato original
   - Sin transformaciones

🔷 SILVER: Limpieza y normalización
   - Problema: Campos numéricos venían como texto
   - Solución: Script con Spark SQL que limpia y retipa
   - Elimina duplicados
   - Valida integridad

🔸 GOLD: Modelo estrella para analytics
   - Fact: Ventas (grano: pedido individual)
   - Dims: Producto, Cliente, Zona, Proveedor, Tiempo
   - Storage: Delta/Parquet en OneLake
   - Consumo: Direct Lake (Power BI lee Parquet directo)

Orquestación: Data Factory automatiza todo el flujo.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PARTE 3: LOS INSIGHTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Al cruzar ventas + logística + devoluciones + comisiones,
aparecieron cosas que nadie sabía:

🔍 12 productos perdían dinero después de envío
   (los más vendidos, irónicamente)

🔍 Zona sureste costaba 40% más
   (cobraban precio estándar a todos)

🔍 Peso volumétrico impactaba 35% de envíos
   (no lo calculaban antes de elegir proveedor)

🔍 Proveedor "habitual" era 28% más caro
   en 6 de cada 10 envíos

Dashboard muestra todo esto en tiempo real:
→ Margen neto REAL por producto
→ Rentabilidad por zona
→ Comparativa proveedores
→ Simulador: "Cambiar a proveedor X = ahorro Y"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RESULTADOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Después de 3 meses:

✅ 23% reducción en costos de envío
✅ Decisiones en 2 seg vs 30 min manuales
✅ Primera vez viendo margen real, no solo revenue
✅ 3 productos descontinuados (margen negativo)
✅ Pricing ajustado en zonas no rentables

El cliente ahora sabe exactamente:
→ Cuánto gana en cada pedido
→ Qué zonas son rentables
→ Qué proveedor conviene según peso/destino

Ya no adivina.
Lo ve.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STACK TÉCNICO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Para quien le interese la implementación:

Sistema completo:
→ Frontend: Checkout transaccional
→ Automatización: n8n (orquestador decisiones)
→ Transaccional: Supabase (PostgreSQL)
→ Pipeline: Microsoft Fabric (medallion architecture)
→ Procesamiento: Spark SQL (transformaciones)
→ Storage: OneLake (Delta/Parquet)
→ Analytics: Power BI (Direct Lake)

Todo integrado.
Desde el checkout hasta el análisis.

Dashboard público: 
https://app.powerbi.com/view?r=...

Arquitectura técnica: 
github.com/ErnestRr/msfabric-eccomerce

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LECCIONES DE ESTE PROYECTO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. El problema NO era tecnológico
   Era de visibilidad

2. Los costos ocultos importan más que las tarifas base
   (Volumétrico, zonas, devoluciones, comisiones)

3. Ver tu margen REAL puede doler
   Pero es la única forma de optimizarlo

4. La automatización sin analytics es medio proyecto
   Necesitas ambos

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

¿Tu negocio calcula margen real 
o solo sabe cuánto factura?

Para dueños de negocio:
Si tienes e-commerce/retail y quieres saber
dónde están tus fugas de dinero, escríbeme.
Primera auditoría sin costo.

Para equipos técnicos:
Si necesitas migrar a Fabric o implementar
arquitectura medallion, también escríbeme.
Puedo ayudar con la implementación.

---

PD: La diferencia entre "cuánto vendí" 
y "cuánto realmente gané" puede ser 20-30%.

Vale la pena saberlo.

#MicrosoftFabric #PowerBI #Ecommerce #DataEngineering #Rentabilidad #Fabric #Analytics #CostosReales

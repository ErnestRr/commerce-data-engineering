# 📈 E-commerce Profitability & Data Engineering: Microsoft Fabric Architecture

Este proyecto **implementó** una solución de **Ingeniería de Datos end to end** utilizando **Microsoft Fabric**. El objetivo principal fue transformar datos transaccionales ruidosos —alojados inicialmente en **Supabase (PostgreSQL)**— en una arquitectura de **Lakehouse** optimizada para el cálculo de la rentabilidad real (Net Profit) y la eficiencia operativa mediante un enfoque de **Arquitectura Medallion**.

## 🎯 El Problema de Negocio (Business Case)
Las organizaciones de E-commerce (Pymes) suelen operar con una visión parcial de su salud financiera debido a:
* **Datos Fragmentados:** Información dispersa entre diversas plataformas de venta, ERPs y operadores logísticos.
* **Inconsistencia de Tipos:** Datos numéricos que ingresaban como texto (`String`), bloqueando cualquier análisis de agregación.
* **Costos Ocultos:** Incapacidad de integrar devoluciones, comisiones de pasarelas y gastos de última milla en el cálculo del margen bruto y neto.

## 🏗️ Arquitectura de Datos (Modern ELT)
A diferencia del ETL tradicional, se **desarrolló** un flujo **ELT** (Extract, Load, Transform) aprovechando el poder de procesamiento de **Microsoft Fabric** y el almacenamiento unificado en **OneLake**.

<img width="1904" height="899" alt="image" src="https://github.com/user-attachments/assets/22b43c7a-f93e-49fd-b31a-cd59e7f535be" />

<img width="1917" height="746" alt="image" src="https://github.com/user-attachments/assets/a274f752-a44e-4f56-b562-bbe40d112c74" />

### Capas del Lakehouse:
1.  **Capa Bronze (Raw data):** Se **realizó** la ingesta de archivos CSV, Excel y conexiones SQL desde **Supabase** mediante **Data Factory Pipelines**. Los datos se mantuvieron en su formato original para fines de auditoría.
2.  **Capa Silver (Cleansed):** Los datos fueron procesados con **Power Query Online**. 
        * Se normalizaron esquemas y se eliminaron registros duplicados.
3.  **Capa Gold (Notebook %%sql):** Se **creó** un **Modelo en Estrella (Star Schema)** utilizando Notebooks con **Spark SQL**. Los datos se sirvieron mediante **Direct Lake**, permitiendo que Power BI consumiera los archivos Parquet en OneLake sin necesidad de importar datos, garantizando latencia mínima.

## 📈 Modelo de Datos Optimizado
El diseño del modelo se **estructuró** utilizando una tabla de hechos  y dimensiones.

<img width="1893" height="862" alt="image" src="https://github.com/user-attachments/assets/427ef38f-2693-4fab-839a-964ebd3fb882" />

> **Solución de Ingeniería:** Durante la transformación en la capa **Silver**, se **integró** un script que utilizó funciones de reemplazo para caracteres no numéricos y un re-tipado forzado al esquema de datos. Esto **aseguró** que el motor de Power BI pudiera ejecutar medidas DAX de inteligencia de tiempo y cálculos de margen sin errores de compatibilidad.

## 💡 Estrategia de Consumo y Optimización de Costos
Para maximizar la eficiencia operativa y reducir costos de licenciamiento, el flujo de trabajo se **diseñó** de la siguiente manera:
* **Modelo Semántico Centralizado:** Se publicó el modelo optimizado en el servicio de Fabric.
* **Consumo Local (Power BI Desktop):** Se **utilizó** Power BI Desktop para conectar al **Modelo Semántico del Medallion**. Esto permitió diseñar el reporte sin requerir el procesamiento de la nube para cada cambio visual, optimizando los costos de licencia de capacidad.

## ⚙️ Tecnologías Utilizadas
* **Supabase:** Tablas transaccionales (PostgreSQL).
* **Microsoft Fabric:** Orquestación, Lakehouse y Gobernanza.
* **Spark SQL (%%sql):** Procesamiento y modelado de la capa Gold.
* **OneLake:** Almacenamiento en formato **Delta / Parquet**.
* **Power BI & DAX:** Modelado semántico y visualización de KPIs.
* **Power Query (M):** Motores de transformación para la limpieza de datos.
* **n8n / Python:** Lógica de asignación de proveedor de delivery.

Ver reporte interactivo 👉 [Dashboard de Rentabilidad](https://app.powerbi.com/view?r=eyJrIjoiMzJiODdjNTAtYmZiNS00NTM0LWEwZTQtODg1ZGU3NzYwMWI1IiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9)

<img width="1491" height="827" alt="image" src="https://github.com/user-attachments/assets/2297200e-4cac-4d55-bca6-d046c2e2beab" /> 
<img width="1508" height="822" alt="image" src="https://github.com/user-attachments/assets/f4dc5cc5-4e79-4b4d-bf64-17aeef9bc9aa" />
<img width="1493" height="841" alt="image" src="https://github.com/user-attachments/assets/0474aa6d-a7fe-4c5d-a14f-b5e88bf9be4a" />

---

## 🗺️ ¿En qué situaciones se podría implementar?

Este tipo de canalización de datos (Modern ELT) es ideal para organizaciones que:

* **Manejan volúmenes significativos de datos de ventas** que crecen continuamente y requieren procesamiento escalable.
* **Necesitan realizar análisis de rentabilidad complejos** sobre datos históricos y actuales en tiempo real.
* **Buscan una "fuente única de verdad"** para eliminar las discrepancias entre los reportes de finanzas, ventas y logística.
* **Desean optimizar costos de licencia**, centralizando el modelo en Fabric y consumiéndolo localmente para el diseño de reportes.
* **Quieren desacoplar las cargas de trabajo analíticas** de sus sistemas transaccionales para no afectar el rendimiento de la operación.
* **Requieren un proceso de ingesta automatizado** que detecte y limpie errores de tipado de forma automática.
* **Negocios que requieren migrar de la versión anterior (Power BI Service)** hacia la capacidad y potencia analítica de **Microsoft Fabric**.

---
**Desarrollado por Ernesto Roldán**

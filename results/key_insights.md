# Key Insights

Este documento resume los principales resultados obtenidos durante el análisis exploratorio, la segmentación RFM, la segmentación mediante K-Means y el perfilado final de clientes.

## 1. Dataset y calidad de los datos

El dataset contiene **2.240 clientes y 29 variables originales**, incluyendo información sociodemográfica, comportamiento de compra, gasto por categoría de producto, interacción con campañas de marketing y actividad en distintos canales.

Durante la fase de análisis y preparación se identificaron varios aspectos relevantes:

- `Income` contenía **24 valores ausentes**, imputados mediante la mediana de ingresos dentro de cada nivel educativo.
- Se detectaron tres años de nacimiento especialmente antiguos: **1893, 1899 y 1900**. Al no existir evidencia suficiente para determinar su valor correcto, se conservaron en el dataset preparado.
- Se identificó un ingreso de **666.666**, muy alejado del resto de observaciones. También se conservó en el dataset preparado, evitando sustituirlo por un valor arbitrario.
- `Z_CostContact` y `Z_Revenue` eran variables constantes y fueron eliminadas.
- Se crearon seis variables derivadas: `Age`, `CustomerTenure`, `Children`, `TotalSpend`, `TotalPurchases` y `CampaignAcceptances`.

El dataset preparado contiene **2.240 registros y 33 variables**, sin valores ausentes ni registros duplicados.

## 2. Exploratory Data Analysis

### Comportamiento de compra

El gasto presenta una distribución asimétrica: una parte de los clientes concentra niveles de consumo considerablemente superiores al resto.

El vino constituye una categoría especialmente relevante dentro del gasto de los clientes, mientras que carne representa también una parte importante del consumo.

En cuanto a los canales de compra:

- Las tiendas físicas presentan el mayor número medio de compras.
- El canal web ocupa la segunda posición.
- Las compras por catálogo tienen una utilización inferior.
- Las compras con descuento se analizaron como comportamiento promocional y no como un canal adicional.

### Actividad digital

El número de visitas mensuales a la web presenta una asociación prácticamente nula con el número de compras online:

- Pearson: **-0,056**
- Spearman: **-0,097**

Por tanto, una mayor frecuencia de visitas no implica necesariamente una mayor actividad de compra online.

### Campañas de marketing

La tasa de respuesta a la última campaña es aproximadamente del **14,9%**, superior a las tasas individuales observadas en las cinco campañas anteriores.

Los clientes que respondieron a la última campaña presentan, en términos descriptivos, mayores niveles de ingresos y gasto que quienes no respondieron.

Estas relaciones son descriptivas y no permiten establecer causalidad.

## 3. RFM Customer Segmentation

La segmentación RFM utiliza:

- **Recency:** días desde la última compra.
- **Frequency:** número total de compras.
- **Monetary:** gasto total acumulado.

Se asignaron puntuaciones de 1 a 4 para cada dimensión y posteriormente se agruparon las combinaciones RFM en seis segmentos de negocio.

| Segmento | Clientes | % |
|---|---:|---:|
| Champions | 535 | 23,88% |
| Loyal Customers | 209 | 9,33% |
| Potential Loyalists | 378 | 16,88% |
| At Risk | 558 | 24,91% |
| Need Attention | 200 | 8,93% |
| Hibernating | 360 | 16,07% |

`TotalPurchases` y `TotalSpend` presentan una correlación de Spearman de aproximadamente **0,91**, mostrando una relación muy fuerte entre frecuencia de compra y valor monetario.

La dimensión de recencia permite diferenciar clientes con un valor histórico similar pero con distinto nivel de actividad reciente. Por ejemplo, `Champions` y `At Risk` presentan niveles elevados de gasto y frecuencia, pero se diferencian claramente por su recencia.

![Distribución de segmentos RFM](../reports/figures/rfm_segment_distribution.png)

![Recencia y gasto por segmento RFM](../reports/figures/rfm_recency_spend.png)

## 4. K-Means Customer Segmentation

Para la segmentación mediante K-Means se utilizaron seis variables:

- `Age`
- `Income`
- `Children`
- `CustomerTenure`
- `Recency`
- `TotalSpend`

`TotalPurchases` se excluyó del modelo debido a su elevada relación con `TotalSpend`, evitando introducir información altamente redundante.

Antes del entrenamiento se excluyeron únicamente de la muestra de clustering cuatro observaciones extremas:

- tres clientes con edad superior a 100 años;
- un cliente con ingresos de 666.666.

Estas observaciones permanecen en el dataset preparado original.

### Selección del número de clusters

Se evaluaron soluciones entre **k = 2 y k = 10** mediante inercia y Silhouette Score.

La solución con `k = 2` obtuvo el mayor Silhouette Score (**0,252**), mientras que `k = 3` obtuvo aproximadamente **0,190**.

Se seleccionó **k = 3** porque aportaba una granularidad adicional útil para interpretar diferentes perfiles de cliente, mantenía tamaños de cluster equilibrados y mostró estabilidad completa frente a diferentes inicializaciones.

La estabilidad de `k = 3` se comprobó utilizando diferentes semillas aleatorias, obteniendo un **Adjusted Rand Index de 1,0** en todas las comparaciones realizadas.

Esta estabilidad indica consistencia frente a la inicialización del algoritmo, pero no demuestra por sí sola la existencia de tres grupos naturales perfectamente separados.

![Silhouette Score según número de clusters](../reports/figures/kmeans_silhouette.png)

### Evaluación mediante PCA

PCA se utilizó únicamente como herramienta de visualización, no para entrenar el modelo K-Means.

Las dos primeras componentes explican aproximadamente:

- PC1: **35,61%**
- PC2: **18,28%**
- Varianza acumulada: **53,89%**

PC1 está asociada principalmente con gasto, ingresos y composición familiar, mientras que PC2 está dominada principalmente por edad y, en menor medida, número de hijos.

La proyección muestra una separación económica relativamente clara del cluster de mayor valor, mientras que existe mayor solapamiento entre los otros dos grupos.

![Clusters proyectados mediante PCA](../reports/figures/kmeans_pca_clusters.png)

## 5. Customer Profiles

La combinación del modelo K-Means con variables de comportamiento, marketing y la segmentación RFM permitió definir tres perfiles interpretables.

### Cluster 0 — Young Low-Value Customers

Características principales:

- Edad media: **36,9 años**
- Ingresos medios: **31.945**
- Gasto medio: **151**
- Compras medias: **7,0**
- Hijos medios: **0,84**
- Respuesta a la última campaña: **11,0%**

Es el grupo más joven y presenta los menores niveles de ingresos y gasto.

Dentro del cluster existe una diferencia importante entre clientes `Potential Loyalists` y `Hibernating`, por lo que la recencia puede utilizarse para distinguir clientes con mayor potencial de activación.

### Cluster 1 — High-Value Customers

Características principales:

- Edad media: **46,5 años**
- Ingresos medios: **73.388**
- Gasto medio: **1.286**
- Compras medias: **19,6**
- Hijos medios: **0,40**
- Respuesta a la última campaña: **24,0%**

Es el segmento de mayor valor económico, con niveles claramente superiores de gasto y actividad de compra.

Aproximadamente la mitad del cluster pertenece al segmento RFM `At Risk` y gran parte del resto a `Champions`, mostrando que clientes con valor económico similar pueden encontrarse en situaciones muy diferentes respecto a su actividad reciente.

### Cluster 2 — Family-Oriented Moderate-Value Customers

Características principales:

- Edad media: **51,7 años**
- Ingresos medios: **47.918**
- Gasto medio: **299**
- Compras medias: **10,2**
- Hijos medios: **1,67**
- Respuesta a la última campaña: **8,6%**

Es el grupo de mayor edad y con mayor presencia de hijos en el hogar.

Su composición RFM es más heterogénea que la de los otros clusters, por lo que el cluster por sí solo no permite determinar el estado de la relación reciente con el cliente.

## 6. K-Means y RFM como enfoques complementarios

Las dos segmentaciones responden a preguntas diferentes:

- **K-Means** identifica perfiles multidimensionales basados en características económicas, demográficas y de comportamiento.
- **RFM** describe el estado de la relación comercial a partir de recencia, frecuencia y gasto.

El cruce entre ambas segmentaciones aporta más información que cualquiera de ellas por separado.

Por ejemplo, el cluster `High-Value Customers` contiene principalmente `Champions` y `At Risk`: clientes con características económicas similares, pero con niveles de actividad reciente muy diferentes.

![Comparación entre clusters K-Means y segmentos RFM](../reports/figures/rfm_kmeans_heatmap.png)

## 7. Marketing Insights

A partir de los perfiles observados se plantean las siguientes hipótesis de actuación:

- **Young Low-Value Customers:** estrategias orientadas a incrementar frecuencia y activación, diferenciando especialmente entre clientes recientes con potencial y clientes inactivos.
- **High-Value Customers:** priorizar retención, fidelización y cross-selling, utilizando RFM para diferenciar `Champions` de clientes `At Risk`.
- **Family-Oriented Moderate-Value Customers:** explorar estrategias de incremento de cesta, diversificación de categorías y promociones segmentadas.

Estas recomendaciones representan **hipótesis de negocio derivadas de asociaciones descriptivas**. Su impacto debería validarse mediante experimentación, grupos de control o tests A/B antes de atribuir efectos causales.

## 8. Conclusión

El análisis muestra que una única segmentación no captura todas las dimensiones relevantes del comportamiento del cliente.

K-Means permite identificar perfiles estructurales diferenciados, mientras que RFM incorpora la situación reciente de la relación comercial. Su utilización conjunta proporciona una visión más completa para diseñar estrategias de marketing segmentadas y generar hipótesis de actuación medibles.
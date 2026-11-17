# Bitácora de desarrollo del proyecto

**Proyecto:** Comparación experimental de modelos preentrenados para reconocer enfermedades de la yuca mediante fotografías de hojas.

**Framework:** TensorFlow/Keras

**Entorno:** Google Colab

**Repositorio:** https://github.com/ThePretelR/cassava-disease-model-comparison

---

## Registro 01 — 13 de noviembre de 2026

### Actividad: Configuración del entorno de trabajo

**Objetivo:** Preparar el entorno de desarrollo para realizar los experimentos de clasificación de imágenes.

**Actividades realizadas:**
- Configuración de Google Colab.
- Verificación de Python 3.13.16.
- Verificación de TensorFlow 2.21.0.
- Verificación de TensorFlow Datasets 4.9.10.
- Comprobación de disponibilidad de GPU.

**Resultado:** El entorno reconoció correctamente una GPU compatible con TensorFlow.

### Problema encontrado

Durante la carga inicial del dataset Cassava se presentó el siguiente error:

`ModuleNotFoundError: No module named 'importlib_resources'`

**Solución aplicada:** Instalación de la dependencia faltante mediante `%pip install importlib_resources`.

**Resultado:** La descarga y preparación del dataset se completaron correctamente.

---

## Registro 02 — 14 de noviembre de 2026

### Actividad: Descarga y verificación del dataset

**Fuente:** TensorFlow Datasets.

**Dataset:** Cassava 0.1.0.

**Resultados:**

| Partición | Imágenes |
|---|---:|
| Entrenamiento | 5.656 |
| Validación | 1.889 |
| Prueba | 1.885 |
| **Total** | **9.430** |

**Resultado:** Se confirmó la carga de las tres particiones y la disponibilidad de las cinco categorías.

---

## Registro 03 — 16 de noviembre de 2026

### Actividad: Exploración inicial de datos

Se realizaron los siguientes análisis:

1. Identificación de las cinco categorías.
2. Distribución de imágenes por clase.
3. Visualización de fotografías.
4. Análisis de dimensiones originales.
5. Estadísticas de brillo y canales RGB.

**Resultados principales:**

- Imágenes de entrenamiento analizadas: 5.656.
- Resoluciones diferentes: 574.
- Altura promedio: 572,88 píxeles.
- Anchura promedio: 610,93 píxeles.
- Clase con mayor cantidad de imágenes: CMD (46,99 %).
- Clase con menor cantidad de imágenes: Healthy (5,59 %).

**Observación:** Se identificó un desbalance importante entre las categorías.

---

## Registro 04 — 16 de noviembre de 2026

### Actividad: Detección de duplicados y revisión de integridad

Se aplicaron dos técnicas:

- SHA-256 sobre los píxeles originales.
- Hash perceptual (pHash).

**Resultados de SHA-256:**

- Imágenes analizadas: 9.430.
- Hashes únicos: 9.388.
- Hashes compartidos entre entrenamiento y validación: 11.
- Hashes compartidos entre entrenamiento y prueba: 10.
- Hashes compartidos entre validación y prueba: 4.

**Resultados de pHash:**

- Hashes perceptuales únicos: 9.381.
- Registros asociados a hashes perceptuales repetidos: 82.

### Problema encontrado

Se detectaron imágenes con contenido idéntico en distintas particiones y algunos registros con etiquetas diferentes para el mismo contenido.

**Estado:** Pendiente de verificación visual y revisión de consistencia.

**Acción prevista:** Inspeccionar las coincidencias y definir un procedimiento reproducible para evitar fugas de información durante el entrenamiento y la evaluación.

No se han eliminado imágenes ni modificado las particiones originales.

---

## Registro 05 — 16 de noviembre de 2026

### Actividad: Preparación de documentación experimental

Se prepararon los siguientes documentos:

- `01_fuente_datos.md`
- `02_inventario_clases.md`
- `03_protocolo_experimental.md`
- `04_hipotesis.md`

**Decisiones propuestas:**

- Utilizar TensorFlow/Keras.
- Comparar una CNN base, MobileNetV2 y ResNet50.
- Evaluar extracción de características y fine-tuning parcial.
- Utilizar macro-F1 como métrica principal.
- Registrar accuracy, métricas por clase y costo computacional.

**Estado:** Documentación inicial preparada. Protocolo sujeto a la verificación de integridad del dataset.

---

## Registro 07 — 16 de noviembre de 2026

### Actividad: Depuración del dataset y creación de particiones limpias

**Objetivo:** Identificar y tratar imágenes duplicadas y registros con etiquetas contradictorias para reducir el riesgo de fuga de datos durante el entrenamiento y la evaluación de los modelos.

**Problema encontrado:**

Durante el análisis SHA-256 del dataset Cassava se identificaron 26 grupos de imágenes duplicadas, correspondientes a 68 registros. De estos, 18 grupos presentaban etiquetas contradictorias.

También se encontraron imágenes idénticas compartidas entre entrenamiento, validación y prueba, lo que podría comprometer la confiabilidad de las métricas.

**Alternativas consideradas:**

- Conservar las particiones originales sin modificaciones.
- Eliminar únicamente los duplicados entre particiones.
- Excluir los registros con etiquetas contradictorias y conservar una única aparición de cada grupo duplicado consistente.

**Decisión tomada:**

Se adoptó la tercera alternativa para el tratamiento de duplicados exactos.

Se excluyeron todos los registros pertenecientes a grupos con etiquetas contradictorias. Para los duplicados con etiquetas consistentes se conservó una sola aparición, utilizando el orden de prioridad: prueba, validación y entrenamiento.

**Prueba realizada:**

Se generaron manifiestos de registros conservados y excluidos mediante Pandas. Posteriormente, se utilizaron los índices originales para construir datasets filtrados con TensorFlow.

Se verificó la cantidad de elementos mediante iteración completa sobre cada partición.

**Resultados obtenidos:**

| Concepto | Cantidad |
|---|---:|
| Imágenes originales | 9.430 |
| Registros excluidos por etiquetas contradictorias | 52 |
| Registros excluidos por duplicados consistentes | 8 |
| Total de registros excluidos | 60 |
| Imágenes conservadas | 9.370 |

**Distribución final:**

| Partición | Antes | Después |
|---|---:|---:|
| Entrenamiento | 5.656 | 5.616 |
| Validación | 1.889 | 1.879 |
| Prueba | 1.885 | 1.875 |
| **Total** | **9.430** | **9.370** |

Durante la verificación, TensorFlow devolvió una cardinalidad desconocida (`-2`) debido al uso de `.filter()`. El conteo manual confirmó las cantidades esperadas.

**Resultado:** Se construyeron las tres particiones filtradas y se verificó su cantidad de imágenes. Los datasets originales permanecieron disponibles sin modificaciones.

**Estado:** Limpieza de duplicados exactos implementada. Pendiente de verificar los hashes de las particiones filtradas y revisar posibles duplicados visuales detectados mediante pHash.

**Próximo paso:** Finalizar las comprobaciones de integridad, exportar evidencias y preparar el pipeline de entrenamiento de E0.

## Registro 08 — 17 de noviembre de 2026

### Actividad: Limpieza perceptual y cierre del análisis exploratorio de datos

**Objetivo:** Completar la depuración del dataset Cassava mediante hashes perceptuales (pHash), verificar la distribución final de clases y establecer las particiones definitivas para los experimentos.

**Problema identificado:**

Después de la limpieza por SHA-256 permanecían siete grupos de imágenes con pHash repetido, correspondientes a 14 registros. Cuatro grupos incluían imágenes de distintas particiones y dos presentaban etiquetas diferentes.

**Pruebas realizadas:**

1. Se analizaron los pHash de las 9.370 imágenes conservadas después de la primera limpieza.
2. Se visualizaron los siete pares sospechosos y se confirmó que eran visualmente iguales.
3. Se excluyeron cuatro registros pertenecientes a grupos con etiquetas contradictorias.
4. Se excluyeron cinco registros duplicados con etiquetas consistentes, conservando una aparición según la prioridad prueba, validación y entrenamiento.
5. Se generaron los manifiestos finales y los datasets definitivos en TensorFlow.
6. Se verificaron las cantidades de las tres particiones y la distribución de las cinco clases.

**Resultados de la depuración:**

| Concepto | Cantidad |
|---|---:|
| Imágenes originales | 9.430 |
| Exclusiones por SHA-256 | 60 |
| Exclusiones adicionales por pHash | 9 |
| Total de exclusiones | 69 |
| Imágenes finales | 9.361 |

**Particiones definitivas:**

| Partición | Imágenes |
|---|---:|
| Entrenamiento | 5.610 |
| Validación | 1.877 |
| Prueba | 1.874 |
| Total | 9.361 |

**Distribución final por clase:**

| Clase | Total | Porcentaje |
|---|---:|---:|
| cbb | 760 | 8,12 % |
| cbsd | 2.382 | 25,45 % |
| cgm | 1.272 | 13,59 % |
| cmd | 4.422 | 47,24 % |
| healthy | 525 | 5,61 % |

**Interpretación:**

El dataset presenta un desequilibrio significativo. La clase `cmd` representa aproximadamente el 47 % de las imágenes, mientras que `healthy` representa cerca del 6 %.

Por esta razón, se utilizará macro-F1 como métrica principal para comparar los modelos, complementada con accuracy, recall y F1 por clase y matrices de confusión.

**Decisión metodológica:**

Todos los experimentos utilizarán las mismas particiones depuradas. La limpieza se realizó antes del entrenamiento para reducir el riesgo de fuga de información y mejorar la consistencia de la evaluación.

**Resultado:** Se completó el análisis exploratorio, se construyeron los datasets definitivos y se verificó su distribución.

**Próximo paso:** Preparar el preprocesamiento común e implementar el experimento E0, una CNN entrenada desde cero.

## Próximas actividades

- [ ] Implementar y entrenar E0.
- [ ] Iniciar E1 con MobileNetV2.
- [ ] Registrar métricas y tiempos de ejecución.
- [ ] Preparar evidencias para la revisión intermedia.

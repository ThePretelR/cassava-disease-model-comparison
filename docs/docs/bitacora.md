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

## Bitácora de desarrollo — Experimento E0

**Fecha:** 17 de noviembre de 2026  
**Proyecto:** Comparación experimental de modelos preentrenados para reconocer enfermedades de la yuca mediante fotografías de hojas.  
**Experimento:** E0 — CNN entrenada desde cero.

### 1. Objetivo de la actividad

Implementar, entrenar y evaluar una red neuronal convolucional (CNN) desde cero, estableciendo un modelo base para comparar posteriormente el rendimiento de MobileNetV2 y ResNet50.

### 2. Actividades realizadas

1. Se reconstruyeron las particiones definitivas del dataset Cassava, utilizando el manifiesto de imágenes conservadas después del proceso de limpieza.
2. Se verificó la integridad de las 9.361 imágenes mediante hashes SHA-256.
3. Se configuró el preprocesamiento con imágenes de 224 × 224 píxeles y normalización al intervalo [0, 1].
4. Se implementó una CNN con tres capas convolucionales, operaciones de MaxPooling, GlobalAveragePooling, una capa densa, Dropout y una salida Softmax de cinco clases.
5. Se configuró el entrenamiento con el optimizador Adam, función de pérdida Sparse Categorical Crossentropy, EarlyStopping y ModelCheckpoint.
6. Se presentaron problemas de ejecución relacionados con el consumo de memoria y el agotamiento de datos durante el entrenamiento.
7. Se optimizó la preparación del dataset utilizando un búfer de mezcla de 512 imágenes, procesamiento paralelo limitado, prefetch de un lote y batch size de 8.
8. Se completó el entrenamiento de 15 épocas y se evaluó el modelo utilizando el conjunto de validación.
9. Se generaron las métricas de clasificación, la matriz de confusión y las curvas de aprendizaje.
10. Se organizaron y subieron a GitHub el notebook, los archivos CSV y las gráficas del experimento.

### 3. Configuración experimental

| Parámetro | Valor |
|---|---|
| Modelo | CNN E0 |
| Parámetros entrenables | 110.405 |
| Imágenes de entrenamiento | 5.610 |
| Imágenes de validación | 1.877 |
| Imágenes de prueba reservadas | 1.874 |
| Resolución | 224 × 224 |
| Batch size | 8 |
| Épocas | 15 |
| Optimizador | Adam |
| Learning rate | 0,001 |
| Semilla | 42 |

### 4. Resultados obtenidos

| Métrica | Resultado |
|---|---|
| Accuracy de validación | 67,87 % |
| Macro-F1 | 0,4699 |
| Loss de validación | 0,8850 |
| Tiempo de entrenamiento | 495,67 segundos |
| Épocas completadas | 15 |

El modelo presentó un mejor reconocimiento de las clases CMD y CBSD, con F1-score de aproximadamente 0,82 y 0,70, respectivamente. Las clases CBB y CGM obtuvieron los resultados más bajos, con F1-score cercanos a 0,18 y 0,19.

La matriz de confusión evidenció 88 imágenes CBB clasificadas como CBSD y 155 imágenes CGM clasificadas como CMD.

### 5. Dificultades y soluciones

**Dificultad 1:** El entrenamiento se interrumpía antes de completar las épocas previstas.

**Solución:** Se modificó la canalización de datos para reducir el consumo de memoria y se configuró la repetición controlada del dataset durante el entrenamiento.

**Dificultad 2:** Se identificó un aumento progresivo en el uso de RAM.

**Solución:** Se redujo el tamaño del lote a 8, se limitó el búfer de mezcla a 512 y se agregó un callback para monitorear el consumo de memoria.

**Dificultad 3:** Se detectó un rendimiento desigual entre las clases.

**Análisis:** El modelo reconoce mejor las clases con mayor representación, aunque también pueden influir las similitudes visuales y la capacidad de la arquitectura.

### 6. Conclusiones

El experimento E0 se completó y permitió establecer una línea base reproducible para la comparación de modelos. Las curvas de aprendizaje mostraron una mejora progresiva durante las 15 épocas, sin señales claras de sobreajuste.

La diferencia entre accuracy y Macro-F1 evidencia la importancia de evaluar el rendimiento equilibrado entre clases, especialmente ante el desbalance del conjunto de datos.

### 7. Próximas actividades

Implementar el experimento E1 utilizando MobileNetV2 preentrenada con

## 18 de noviembre de 2026 — Ejecución parcial del experimento E1

**Experimento:** E1 — MobileNetV2 preentrenada con ImageNet y base convolucional congelada.

### Objetivo

Implementar y ejecutar parcialmente un modelo de transferencia de aprendizaje para evaluar su rendimiento inicial frente a la CNN entrenada desde cero (E0).

### Actividades realizadas

1. Se reconstruyeron las particiones definitivas del dataset Cassava, conservando 5.610 imágenes de entrenamiento, 1.877 de validación y 1.874 de prueba.
2. Se configuró MobileNetV2 con pesos preentrenados de ImageNet y se congelaron todas las capas de su base convolucional.
3. Se implementó una cabeza de clasificación con GlobalAveragePooling2D, una capa densa de 128 neuronas, Dropout y una salida Softmax de cinco clases.
4. Se aplicó el preprocesamiento específico de MobileNetV2, transformando los valores de los píxeles al intervalo [-1, 1].
5. Se optimizó el procesamiento de datos para limitar el consumo de RAM utilizando lotes de ocho imágenes, un búfer de mezcla de 512 y prefetch de un lote.
6. Se ejecutaron tres épocas de entrenamiento en Google Colab con GPU.
7. Se calcularon las métricas preliminares sobre el conjunto de validación y se generaron los archivos de resultados y visualizaciones.

### Configuración

| Parámetro | Valor |
|---|---|
| Arquitectura | MobileNetV2 |
| Parámetros totales | 2.422.597 |
| Parámetros entrenables | 164.613 |
| Parámetros congelados | 2.257.984 |
| Resolución | 224 × 224 |
| Batch size | 8 |
| Épocas ejecutadas | 3 |
| TensorFlow | 2.21.0 |

### Resultados preliminares

| Métrica | Resultado |
|---|---|
| Accuracy de validación | 76,45 % |
| Macro-F1 | 0,6777 |
| Pérdida de validación | 0,6520 |
| Tiempo de entrenamiento | 174,13 segundos |
| Tiempo de inferencia | 5,35 ms/imagen |

El Macro-F1 de E1 fue superior al de E0 (0,4699), con una diferencia de 0,2078. La mejora fue especialmente notable en la clasificación de CGM.

### Consumo de memoria

Durante el entrenamiento, la RAM utilizada aumentó del 21,7 % al finalizar la primera época al 28,3 % al finalizar la tercera. El proceso terminó con aproximadamente 9,09 GB disponibles, sin interrupciones por falta de memoria.

### Conclusión

La ejecución parcial permitió verificar que MobileNetV2 funciona correctamente con la base congelada y presenta resultados preliminares prometedores. La comparación todavía no es definitiva, ya que E0 se entrenó durante 15 épocas y E1 solamente durante tres.

### Próximos pasos

Completar el entrenamiento de E1, analizar su rendimiento final y continuar con E2, correspondiente al ajuste fino parcial de MobileNetV2.

## Bitácora de desarrollo — Experimento E1 (finalización)

**Experimento:** E1 — MobileNetV2 preentrenada con ImageNet y base convolucional congelada.

### Objetivo

Completar el entrenamiento de E1 con el máximo de 15 épocas definido en el protocolo, para que la comparación con E0 y con los demás experimentos sea equitativa, y obtener el modelo final que sirve de punto de partida para E2.

### Problema identificado

La ejecución parcial anterior se realizó con `EPOCHS = 3`, aunque el protocolo establece un máximo de 15 épocas. La pérdida de validación seguía disminuyendo en la tercera época (0,750 → 0,705 → 0,652), por lo que el entrenamiento no podía considerarse concluido.

### Actividades realizadas

1. Se modificó el parámetro `EPOCHS` de 3 a 15 y se volvió a ejecutar el notebook completo desde una sesión nueva en Google Colab con GPU T4.
2. Se mantuvieron sin cambios el resto de la configuración: batch size 8, learning rate 0,001, semilla 42, `EarlyStopping` con paciencia 4 sobre `val_loss` y sin aumento de datos.
3. Se evaluó el mejor modelo sobre el conjunto de validación y se regeneraron los archivos de `results/e1/` (historial, reporte de clasificación, matriz de confusión, curvas, comparación con E0 y errores inspeccionados).

### Resultados

| Concepto | Resultado |
|---|---|
| Épocas ejecutadas | 7 de 15 |
| Mejor época (menor `val_loss`) | 3 |
| Motivo de la detención | `EarlyStopping`: la pérdida de validación no mejoró durante las cuatro épocas siguientes a la época 3 |
| Accuracy de validación (mejor modelo) | 76,45 % |
| Macro-F1 (mejor modelo) | 0,6777 |
| Tiempo de entrenamiento (7 épocas) | [completar desde `resumen_metricas.csv`] |
| Tiempo de inferencia | [completar desde `resumen_metricas.csv`] ms/imagen |

El mejor modelo corresponde a la época 3, porque `EarlyStopping` restauró los pesos con menor `val_loss`. Por eso las métricas finales coinciden con las de la ejecución parcial anterior (Macro-F1 0,6777 y accuracy 0,7645).

### Interpretación

La ejecución de 15 épocas confirmó que más entrenamiento no mejoraba la pérdida de validación con esta configuración: el valor mínimo se alcanzó en la época 3. La ejecución parcial anterior ya estaba cerca del óptimo, y los resultados de E1 son ahora definitivos para el protocolo.

La comparación con E0 se mantiene: Macro-F1 de 0,6777 frente a 0,4699, es decir, una mejora de 0,2078 con el modelo preentrenado. Al igual que en la ejecución parcial, la mejora más notable se observa en las clases pequeñas (CGM y CBB).

### Resultado

E1 quedó completo y su modelo final (`mejor_modelo_e1.keras`) se utilizó como punto de partida del experimento E2.

### Próximo paso

Ejecutar E2 (ajuste fino parcial de MobileNetV2) a partir del mejor modelo de E1.

## Bitácora de desarrollo — Experimento E2

**Experimento:** E2 — MobileNetV2 con ajuste fino (fine-tuning) parcial.

### Objetivo

Evaluar si descongelar las últimas capas del backbone de MobileNetV2 mejora el desempeño de E1 (comparación secundaria E1 frente a E2, hipótesis H3).

### Actividades realizadas

1. Se reconstruyeron las particiones definitivas con el manifiesto de imágenes conservadas: 5.610 de entrenamiento, 1.877 de validación y 1.874 de prueba (reservadas, no utilizadas).
2. Se cargó el mejor modelo de E1 y se volvió a evaluar en validación dentro del mismo notebook, para comparar ambos experimentos con el mismo código de métricas. El Macro-F1 recalculado de E1 fue 0,6777, idéntico al registrado.
3. Se descongelaron las últimas 20 capas del backbone, manteniendo congeladas las capas BatchNormalization (7 de las 20). El resto del backbone y todas las capas anteriores permanecieron congelados.
4. Se recompiló el modelo con Adam y learning rate 1e-5, y se entrenó un máximo de 10 épocas con `EarlyStopping` (paciencia 4) sobre `val_loss` y `ModelCheckpoint`.
5. Se evaluó el mejor checkpoint en validación y se generaron las métricas, la matriz de confusión, las curvas de aprendizaje, la comparación E1 frente a E2 (global y por clase) y la inspección de ocho errores.

### Configuración

| Parámetro | Valor |
|---|---|
| Punto de partida | `mejor_modelo_e1.keras` (mejor época de E1) |
| Capas descongeladas | Últimas 20 del backbone (13 entrenables y 7 BatchNorm congeladas) |
| Parámetros totales | 2.422.597 |
| Parámetros entrenables | 1.359.493 (en E1: 164.613) |
| Learning rate | 1e-5 |
| Optimizador | Adam |
| Batch size | 8 |
| Épocas máximas | 10 |
| Semilla | 42 |
| Preprocesamiento | Igual que E1 (normalización a [-1, 1] dentro del modelo) |

### Resultados (validación)

| Métrica | E1 | E2 | Diferencia |
|---|---:|---:|---:|
| Accuracy | 0,7645 | 0,7954 | +0,0309 |
| Macro-F1 | 0,6777 | 0,7182 | +0,0405 |

| Clase | F1 en E1 | F1 en E2 | Diferencia |
|---|---:|---:|---:|
| cbb | 0,4833 | 0,5296 | +0,0463 |
| cbsd | 0,7273 | 0,7654 | +0,0381 |
| cgm | 0,6808 | 0,7113 | +0,0305 |
| cmd | 0,8605 | 0,8800 | +0,0195 |
| healthy | 0,6369 | 0,7048 | +0,0679 |

| Concepto | Resultado |
|---|---|
| Épocas ejecutadas | 7 de 10 |
| Mejor época (menor `val_loss`) | 3 (`val_loss` = 0,5754) |
| Motivo de la detención | `EarlyStopping` |
| Tiempo de entrenamiento | 228,55 segundos |
| Tiempo de inferencia | 5,04 ms/imagen |

### Interpretación

El ajuste fino mejoró el Macro-F1 en 0,0405 y el F1 de las cinco clases. Los mayores aumentos se dieron en `healthy` (+0,068) y `cbb` (+0,046), las clases con menos imágenes. CBB sigue siendo la clase más débil (recall de 0,44).

En las curvas se observa que la accuracy de entrenamiento llega a 0,90 en la época 7, mientras que la de validación se mantiene cerca de 0,79. La pérdida de validación alcanza su mínimo en la época 3 y luego sube, lo que indica el inicio de sobreajuste, y es lo que detuvo el entrenamiento.

Con estos resultados, la hipótesis H3 (E2 supera a E1) queda respaldada en esta ejecución.

### Observaciones y limitaciones

- Se trata de una sola ejecución con semilla 42 y evaluada solo en validación. Las diferencias por clase, en especial en las clases con pocas imágenes (CBB: 151 y healthy: 105), pueden deberse al azar.
- **Tiempo de inferencia:** E1 midió 10,46 ms/imagen y E2 5,04 ms/imagen. Ambos modelos tienen la misma arquitectura y el mismo número total de parámetros, por lo que la diferencia no corresponde a una mejora real. La medición de E1 se realizó primero en la sesión, con el modelo recién cargado. El costo de inferencia de ambos es de aproximadamente 5 ms/imagen.
- **Tamaño del modelo:** el archivo de E1 pesa 11,07 MB y el de E2, 20,18 MB. La diferencia se debe al estado del optimizador Adam que Keras guarda en el archivo `.keras` (E2 tiene más parámetros entrenables), no a un cambio de arquitectura. Para comparar con E3 conviene medir el tamaño de los modelos guardados sin optimizador.
- Se mantuvieron las diferencias de E1 frente al protocolo: batch size 8 (el protocolo indica 32), capa densa de 128 neuronas (el protocolo indica 64) y Dropout de 0,3.
- No se utilizó el conjunto de prueba.

### Archivos generados

`results/e2/`: `historial_entrenamiento.csv`, `reporte_clasificacion.csv`, `matriz_confusion.csv`, `matriz_confusion.png`, `curvas_entrenamiento.png`, `resumen_metricas.csv`, `comparacion_e1_e2.csv`, `comparacion_f1_por_clase_e1_e2.csv`, `errores_inspeccionados.csv`, `errores_inspeccionados.png` y `mejor_modelo_e2.keras`.

Notebook: `notebooks/04_entrenamiento_E2.ipynb`.

## 23/11/2026 – Preparación del experimento E3: ResNet50 congelada

### Objetivo

Comparar el desempeño y el costo computacional de ResNet50 frente a MobileNetV2 (E1), manteniendo condiciones experimentales comparables.

### Problema identificado

Aunque se completaron los experimentos E0, E1 y E2, todavía falta comparar dos arquitecturas preentrenadas diferentes bajo el mismo régimen de extracción de características.

### Alternativas consideradas

- ResNet18: arquitectura residual con menor complejidad computacional.
- ResNet50: arquitectura residual de mayor profundidad, disponible directamente en TensorFlow/Keras.

### Decisión

Se seleccionó ResNet50 con pesos preentrenados en ImageNet, debido a su disponibilidad en Keras y su capacidad de extracción de características. Se mantendrá congelada la base convolucional y se entrenará únicamente la cabeza clasificadora, conservando las condiciones experimentales de E1 y utilizando un máximo de 7 épocas.

### Hipótesis previas

- **H1 – Desempeño:** se espera que ResNet50 obtenga un Macro-F1 superior al 0.6777 alcanzado por MobileNetV2 en E1, debido a su mayor profundidad y capacidad de representación de características visuales.
- **H2 – Clasificación por clase:** se espera una mejora en la clasificación de las categorías minoritarias, especialmente CBB y Healthy, mediante la extracción de características visuales más complejas. Sin embargo, el desbalance del dataset podría limitar esta mejora.
- **H3 – Costo computacional:** se espera que ResNet50 presente mayores tiempos de entrenamiento e inferencia, mayor consumo de memoria y mayor tamaño del modelo que MobileNetV2, debido a su mayor cantidad de parámetros.

### Configuración experimental

| Parámetro | Valor |
|---|---|
| Arquitectura | ResNet50 |
| Pesos iniciales | ImageNet |
| Backbone | Congelado |
| Semilla | 42 |
| Resolución | 224 × 224 |
| Batch size | 8 |
| Épocas máximas | 7 |
| Learning rate | 0.001 |
| Optimizador | Adam |
| Capa densa | 128 neuronas |
| Dropout | 0.3 |
| EarlyStopping | Paciencia de 4 épocas |
| Augmentation | No |
| Imágenes de entrenamiento | 5610 |
| Imágenes de validación | 1877 |
| Imágenes de prueba reservadas | 1874 |
| Métrica principal | Macro-F1 |

### Prueba planificada

- Entrenar ResNet50 utilizando las mismas particiones depuradas de los experimentos anteriores.
- Aplicar el preprocesamiento específico de ResNet50.
- Monitorear el consumo de memoria RAM.
- Calcular accuracy, Macro-F1, recall y F1 por clase.
- Generar la matriz de confusión y las curvas de aprendizaje.
- Registrar los tiempos de entrenamiento e inferencia y el tamaño del modelo.
- Inspeccionar ocho errores de clasificación.
- Comparar los resultados de E3 frente a E1.

### Resultado

El experimento E3 se ejecutó correctamente durante 7 épocas, utilizando ResNet50 con la base convolucional congelada y las mismas particiones depuradas de los experimentos anteriores.

Resultados obtenidos:

- Accuracy: 0.7896.
- Macro-F1: 0.7152.
- Tiempo de entrenamiento: 294.63 segundos.
- Tiempo de inferencia: 6.26 ms por imagen.
- Parámetros totales: 23,850,641.
- Parámetros entrenables: 262,917.
- Mejor pérdida de validación: 0.6153, en la cuarta época.
- Consumo máximo de RAM registrado al finalizar una época: 41 %.

### Contrastación de hipótesis

**H1 – Desempeño:** respaldada. ResNet50 obtuvo un Macro-F1 de 0.7152, superior al 0.6777 de MobileNetV2 en E1.

**H2 – Clasificación por clase:** respaldada. El F1 de CBB aumentó de 0.48 a 0.55 y el de Healthy aumentó de 0.64 a 0.71.

**H3 – Costo computacional:** parcialmente respaldada. ResNet50 presentó mayores tiempos de entrenamiento e inferencia y una cantidad considerablemente mayor de parámetros. Queda pendiente verificar el tamaño definitivo del archivo y comparar el consumo de memoria bajo condiciones equivalentes.

### Revisión de la decisión

ResNet50 mejoró el desempeño frente a MobileNetV2 congelada, aunque presenta un mayor costo computacional. Las curvas de aprendizaje muestran indicios de sobreajuste después de la cuarta época.

Se considera válida la ejecución de E3 y no se realizarán modificaciones adicionales de hiperparámetros. La comparación definitiva se realizará utilizando el conjunto de prueba reservado.

### Revisión de la decisión

Después del entrenamiento se contrastarán los resultados obtenidos con las hipótesis planteadas, determinando si la mayor complejidad de ResNet50 se traduce en una mejora del desempeño que justifique su costo computacional.

## Evaluación final de los modelos E0, E1, E2 y E3

**Fecha:** 24/11/2026

### Problema

Los cuatro experimentos principales habían finalizado, pero las métricas obtenidas correspondían al conjunto de validación. Era necesario evaluar la capacidad de generalización de los modelos sobre imágenes que no participaron en el entrenamiento ni en la selección de hiperparámetros.

### Alternativas consideradas

- Utilizar las métricas de validación como resultados definitivos.
- Evaluar cada modelo de manera independiente en diferentes sesiones.
- Evaluar los cuatro modelos en un mismo notebook, utilizando el mismo conjunto de prueba y condiciones de ejecución.

### Decisión

Se seleccionó la tercera alternativa para garantizar una comparación experimental bajo condiciones equivalentes.

Se creó el notebook `06_evaluacion_final.ipynb`, utilizando los mejores checkpoints obtenidos durante los experimentos E0, E1, E2 y E3, sin realizar entrenamientos adicionales ni modificar los hiperparámetros.

Se mantuvo Macro-F1 como métrica principal debido al desbalance existente entre las cinco clases del dataset.

### Prueba realizada

Se evaluaron los cuatro modelos sobre las 1.874 imágenes del conjunto de prueba definitivo, manteniendo una resolución de 224 × 224 píxeles, un tamaño de lote de 8 y una semilla de 42.

Se respetó el preprocesamiento correspondiente a cada arquitectura. Para E0 se aplicó la normalización entre 0 y 1, mientras que E1, E2 y E3 utilizaron el preprocesamiento incorporado en sus respectivas arquitecturas.

Durante la ejecución se presentó un inconveniente en la carga de los checkpoints, debido a que Google Colab renombró un archivo duplicado. Se solucionó modificando el notebook para utilizar directamente los archivos almacenados en `/content/`, evitando cargas innecesarias.

### Resultados obtenidos

| Modelo | Accuracy | Macro-F1 | Entrenamiento (s) | Inferencia (ms/imagen) | Pesos (MiB) |
|---|---:|---:|---:|---:|---:|
| E0 | 0.6772 | 0.4566 | 495.67 | 4.82 | 0.42 |
| E1 | 0.7620 | 0.6600 | 286.50 | 10.09 | 9.24 |
| E2 | 0.8122 | 0.7403 | 228.55 | 5.48 | 9.24 |
| E3 | 0.8068 | 0.7273 | 294.63 | 11.11 | 90.98 |

El tiempo de entrenamiento de E2 corresponde únicamente a la etapa de fine-tuning. Su tiempo acumulado, incluyendo el entrenamiento previo de E1, fue de 515.06 segundos.

### Análisis de resultados

**Comparación E1 frente a E0: efecto del aprendizaje por transferencia**

MobileNetV2 congelada obtuvo una mejora de 20.34 puntos porcentuales en Macro-F1 y 8.48 puntos porcentuales en accuracy respecto a la CNN baseline.

Estos resultados respaldan la hipótesis de que utilizar características previamente aprendidas mediante ImageNet puede mejorar el reconocimiento de enfermedades de la yuca frente a una arquitectura entrenada desde cero.

**Comparación E2 frente a E1: efecto del fine-tuning**

El ajuste fino de MobileNetV2 produjo una mejora de 8.03 puntos porcentuales en Macro-F1 y 5.02 puntos porcentuales en accuracy.

Esto respalda la hipótesis de que adaptar parcialmente las características preentrenadas al dominio específico de las enfermedades de la yuca mejora el rendimiento de clasificación.

**Comparación E3 frente a E1: efecto de la arquitectura preentrenada**

ResNet50 superó a MobileNetV2 congelada en 6.73 puntos porcentuales de Macro-F1 y 4.48 puntos porcentuales de accuracy.

Sin embargo, ResNet50 posee aproximadamente 9.85 veces más parámetros y requiere un mayor espacio de almacenamiento para sus pesos. Por tanto, la mejora predictiva implica un mayor costo en recursos.

**Comparación E2 frente a E3**

E2 obtuvo el mejor rendimiento general, alcanzando un accuracy del 81.22 % y un Macro-F1 del 74.03 %.

Aunque E3 obtuvo resultados cercanos, E2 presentó un Macro-F1 superior en aproximadamente 1.30 puntos porcentuales, utilizando considerablemente menos parámetros y almacenamiento.

No obstante, esta diferencia no permite afirmar una superioridad estadísticamente significativa, debido a que los experimentos se realizaron con una única semilla y no se efectuaron pruebas de significancia estadística.

### Revisión del costo computacional

E0 presentó el menor tamaño de pesos y el menor tiempo de inferencia observado, aunque obtuvo el rendimiento predictivo más bajo.

E2 presentó el mejor equilibrio observado entre capacidad predictiva, tamaño del modelo y velocidad de inferencia.

Los modelos E1 y E2 poseen la misma cantidad de parámetros totales (2.422.597) y el mismo tamaño de pesos (9.24 MiB). El mayor tamaño del checkpoint de E2 no representa un aumento de la arquitectura y puede estar relacionado con el almacenamiento del estado del optimizador durante el ajuste fino.

Las diferencias en los tiempos de inferencia entre E1 y E2 deben interpretarse con precaución, considerando que ambos utilizan la misma arquitectura y que las mediciones pueden verse afectadas por las condiciones de ejecución de Google Colab.

### Revisión y decisión posterior

Los resultados obtenidos permiten identificar a E2 como el modelo con mejor rendimiento predictivo observado y mejor equilibrio general entre precisión y recursos computacionales.

Se decidió conservar los cuatro modelos sin realizar modificaciones adicionales, evitando utilizar los resultados del conjunto de prueba para ajustar hiperparámetros.

Como siguiente actividad, se realizará el experimento controlado de robustez mediante una reducción del brillo del 20 % sobre una copia del conjunto de prueba, manteniendo constantes los pesos, las etiquetas y las demás condiciones experimentales.

### Evidencias generadas

- Notebook `06_evaluacion_final.ipynb`.
- Tabla consolidada de resultados.
- Métricas individuales por clase.
- Matrices de confusión.
- Predicciones de los cuatro modelos.
- Gráficas comparativas de rendimiento.
- Mediciones de tiempo de inferencia y tamaño de los modelos.
- Registro del entorno de ejecución.
## Experimento adicional: robustez ante variaciones de iluminación

**Fecha:** 24/11/2026

### Problema

Las fotografías de hojas de yuca capturadas en condiciones reales pueden presentar variaciones de iluminación que afecten la capacidad de los modelos para reconocer los síntomas de las enfermedades.

Por esta razón, fue necesario evaluar la estabilidad de los cuatro modelos ante una modificación controlada del brillo de las imágenes.

### Alternativas consideradas

- Aplicar técnicas de aumento de datos durante el entrenamiento.
- Evaluar los modelos ante diferentes niveles de ruido.
- Reducir el brillo de las imágenes del conjunto de prueba sin modificar los modelos entrenados.

### Decisión

Se seleccionó la tercera alternativa, conforme al protocolo experimental establecido previamente.

Se aplicó un factor multiplicativo de 0.8 sobre los valores de los píxeles, equivalente a una reducción del 20 % del brillo. Esta transformación permitió estudiar el efecto de la iluminación sin modificar las etiquetas ni los pesos de los modelos.

### Hipótesis

- **H1:** La reducción del brillo disminuirá el Macro-F1 de los modelos debido a la posible pérdida de visibilidad de los síntomas.
- **H2:** El efecto de la reducción del brillo será diferente entre las arquitecturas evaluadas.

### Prueba realizada

Se implementó el notebook `07_experimento_robustez.ipynb`, utilizando los cuatro checkpoints obtenidos durante los experimentos principales.

Se evaluaron las 1.874 imágenes originales del conjunto de prueba y una versión oscurecida de estas mismas imágenes.

Se mantuvieron constantes las siguientes condiciones:

- Semilla: 42.
- Resolución: 224 × 224 píxeles.
- Tamaño de lote: 8.
- Factor de brillo: 0.8.
- Modelos y pesos previamente entrenados.
- Etiquetas y partición de prueba.
- Preprocesamiento correspondiente a cada arquitectura.
- Macro-F1 como métrica principal.

No se realizaron entrenamientos adicionales ni ajustes de hiperparámetros.

### Resultados obtenidos

| Modelo | Accuracy original | Accuracy oscurecida | Macro-F1 original | Macro-F1 oscurecido | Variación Macro-F1 |
|---|---:|---:|---:|---:|---:|
| E0 | 0.6772 | 0.6692 | 0.4566 | 0.4468 | -0.0097 |
| E1 | 0.7620 | 0.7572 | 0.6600 | 0.6509 | -0.0091 |
| E2 | 0.8122 | 0.8026 | 0.7403 | 0.7245 | -0.0158 |
| E3 | 0.8068 | 0.8106 | 0.7273 | 0.7276 | +0.0003 |

### Análisis de resultados

E0 presentó una disminución de 0.97 puntos porcentuales en Macro-F1, manteniéndose como el modelo con menor rendimiento predictivo.

E1 presentó una disminución de 0.91 puntos porcentuales, mostrando una sensibilidad moderada ante la perturbación aplicada.

E2 obtuvo la mayor reducción de Macro-F1, equivalente a 1.58 puntos porcentuales. Aunque fue el modelo con mejor rendimiento sobre las imágenes originales, mostró una mayor sensibilidad a la reducción del brillo.

E3 presentó un incremento de 0.03 puntos porcentuales en Macro-F1 y de aproximadamente 0.37 puntos porcentuales en accuracy. Este comportamiento indica que ResNet50 mantuvo una mayor estabilidad ante la perturbación evaluada.

Sin embargo, el incremento observado en E3 es demasiado pequeño para afirmar que reducir el brillo mejora sistemáticamente su rendimiento.

### Análisis por clase

La respuesta de los modelos ante la reducción del brillo no fue uniforme entre las cinco clases.

| Clase | Variación F1 E0 | Variación F1 E1 | Variación F1 E2 | Variación F1 E3 |
|---|---:|---:|---:|---:|
| CBB | -0.0332 | -0.0237 | -0.0451 | +0.0041 |
| CBSD | -0.0128 | +0.0022 | +0.0058 | +0.0185 |
| CGM | -0.0229 | -0.0182 | -0.0411 | -0.0096 |
| CMD | -0.0039 | -0.0033 | -0.0092 | -0.0021 |
| Healthy | +0.0240 | -0.0025 | +0.0108 | -0.0096 |

La clase CGM presentó una disminución del F1 en los cuatro modelos, lo que evidencia una sensibilidad compartida ante la reducción del brillo.

La clase CBB presentó una disminución considerable en E2, pasando de un F1 de 0.5391 a 0.4939. En contraste, E3 mantuvo resultados relativamente estables para esta clase.

Una posible explicación es que la reducción de iluminación dificulta la identificación de determinados patrones visuales, manchas y cambios de color asociados con las enfermedades. No obstante, esta interpretación requiere una inspección visual adicional para ser confirmada.

La clase CBSD presentó un aumento del recall en los cuatro modelos, aunque esto no implicó necesariamente una mejora del F1 en todos los casos.

CMD presentó variaciones relativamente pequeñas en comparación con otras clases, manteniendo un rendimiento elevado.

### Contrastación de hipótesis

**H1: Parcialmente respaldada.**

Los modelos E0, E1 y E2 presentaron una disminución del Macro-F1 después de reducir el brillo. Sin embargo, E3 mantuvo su rendimiento e incluso presentó un incremento mínimo.

Por tanto, la reducción del brillo no produjo una disminución generalizada en todos los modelos.

**H2: Respaldada por los resultados observados.**

Las arquitecturas presentaron diferentes niveles de sensibilidad ante la misma perturbación. E2 experimentó la mayor disminución del Macro-F1, mientras que E3 presentó la mayor estabilidad.

### Verificación de reproducibilidad

Se compararon los resultados obtenidos sobre las imágenes originales con los registrados durante la evaluación final.

| Modelo | Macro-F1 previo | Macro-F1 actual |
|---|---:|---:|
| E0 | 0.4566 | 0.456560 |
| E1 | 0.6600 | 0.659984 |
| E2 | 0.7403 | 0.740293 |
| E3 | 0.7273 | 0.727316 |

Los resultados coincidieron al redondearlos a cuatro cifras decimales, lo que respalda la reproducibilidad de las métricas y la consistencia del procedimiento de evaluación.

### Limitaciones

El experimento consideró únicamente un nivel de reducción del brillo. Por tanto, los resultados no permiten generalizar el comportamiento de los modelos ante condiciones extremas de iluminación, sombras, desenfoque, ruido u otras perturbaciones.

Tampoco se realizaron pruebas de significancia estadística que permitan establecer si las diferencias observadas son estadísticamente significativas.

### Revisión y decisión posterior

Se concluyó que E3 presentó la mayor robustez observada frente a la reducción del brillo, mientras que E2 mantuvo el mejor rendimiento sobre las imágenes originales y una mayor eficiencia en términos de tamaño del modelo.

Se decidió conservar los resultados de los cuatro modelos sin realizar modificaciones adicionales, evitando utilizar el conjunto de prueba para ajustar hiperparámetros.

Con este experimento se finalizó la fase experimental del proyecto. Las siguientes actividades estarán orientadas a consolidar las evidencias, completar la documentación y elaborar el informe final.

### Evidencias generadas

- Notebook `07_experimento_robustez.ipynb`.
- Tabla comparativa de robustez.
- Métricas por clase.
- Matrices de confusión.
- Gráficas comparativas.
- Predicciones individuales.
- Ejemplos visuales de la perturbación.
- Verificación de las métricas originales.

# Protocolo experimental

**Proyecto:** Comparación experimental de modelos preentrenados para reconocer enfermedades de la yuca mediante fotografías de hojas.

**Fecha de elaboración:** 14 de noviembre de 2026.

**Entorno:** Google Colab.

**Framework:** TensorFlow/Keras.

**Estado:** Protocolo propuesto, pendiente de verificar la integridad de las particiones antes del entrenamiento.

## 1. Objetivo experimental

Comparar el desempeño de dos arquitecturas convolucionales preentrenadas para la clasificación de enfermedades de hojas de yuca, considerando su capacidad de generalización, rendimiento por clase y costo computacional.

Se evaluará también la diferencia entre utilizar una arquitectura preentrenada como extractor de características y realizar un ajuste fino parcial de sus parámetros.

## 2. Conjunto de datos

Se utilizará Cassava, versión `0.1.0`, disponible en TensorFlow Datasets.

| Partición | Imágenes | Uso |
|---|---:|---|
| Entrenamiento | 5.656 | Ajuste de parámetros |
| Validación | 1.889 | Seguimiento y selección |
| Prueba | 1.885 | Evaluación final |
| **Total** | **9.430** | |

Las categorías son CBB, CBSD, CGM, CMD y Healthy.

Se conservará la trazabilidad de las imágenes mediante identificadores, etiquetas y registros de integridad.

### 2.1. Control de duplicados y fugas

El análisis exploratorio inicial detectó coincidencias exactas de contenido entre las particiones originales y posibles inconsistencias de etiquetas.

Antes de entrenar se verificará la correspondencia entre imágenes, etiquetas e índices. Posteriormente se definirá un tratamiento reproducible para evitar que imágenes idénticas aparezcan simultáneamente en entrenamiento y evaluación.

Las particiones definitivas, sus tamaños y cualquier exclusión se registrarán en un archivo de manifiesto.

No se utilizará el conjunto de prueba para seleccionar hiperparámetros ni elegir el modelo ganador.

## 3. Configuración general

| Parámetro | Configuración propuesta |
|---|---|
| Semilla aleatoria | 42 |
| Tamaño de entrada | 224 × 224 píxeles |
| Canales | RGB |
| Batch size | 32 |
| Máximo de épocas | 15 |
| Optimizador | Adam |
| Función de pérdida | Sparse Categorical Crossentropy |
| Salida | Softmax de cinco clases |
| Early stopping | Paciencia de 4 épocas |
| Criterio de restauración | Mejor pérdida de validación |
| Hardware | GPU disponible en Google Colab |

El número máximo de épocas será común para E0, E1 y E3. E2 tendrá un presupuesto adicional de ajuste fino, registrado por separado.

Se registrarán la versión de TensorFlow, el tipo de GPU y los tiempos de ejecución.

## 4. Preprocesamiento

Las imágenes se convertirán a RGB y se redimensionarán a 224 × 224 píxeles.

Para E0 se utilizará normalización al intervalo [0, 1].

Para los modelos preentrenados se aplicará el preprocesamiento correspondiente a cada arquitectura y a sus pesos ImageNet.

No se aplicará aumento artificial de datos durante validación o prueba.

El redimensionamiento y su posible efecto sobre la relación de aspecto se documentarán como limitación metodológica.

## 5. Experimentos

### E0. Baseline

Se implementará una red neuronal convolucional pequeña entrenada desde cero.

Configuración propuesta:

- Tres bloques convolucionales con 32, 64 y 128 filtros.
- MaxPooling después de cada bloque.
- Global Average Pooling.
- Capa densa de 64 unidades.
- Capa de salida Softmax de cinco clases.
- Tasa de aprendizaje inicial: 0,001.

**Propósito:** establecer un rendimiento de referencia sin transferencia de aprendizaje.

### E1. Extracción de características con MobileNetV2

Se utilizará MobileNetV2 con pesos preentrenados en ImageNet.

Configuración:

- Backbone congelado.
- Exclusión de la capa clasificadora original.
- Global Average Pooling.
- Nueva cabeza clasificadora de cinco clases.
- Tasa de aprendizaje inicial: 0,001.

**Propósito:** evaluar la contribución del aprendizaje por transferencia sin modificar el extractor de características.

### E2. Fine-tuning parcial de MobileNetV2

Se partirá del modelo entrenado en E1.

Configuración propuesta:

- Conservar la cabeza clasificadora entrenada.
- Descongelar las últimas 20 capas del backbone, manteniendo congeladas las capas BatchNormalization.
- Utilizar una tasa de aprendizaje de 0,00001.
- Ejecutar hasta 10 épocas adicionales.
- Aplicar early stopping con paciencia de 3 épocas.

**Propósito:** estudiar el efecto del ajuste fino parcial y su costo computacional adicional.

### E3. Extracción de características con ResNet50

Se utilizará ResNet50 con pesos preentrenados en ImageNet.

Configuración:

- Backbone congelado.
- Exclusión de la capa clasificadora original.
- Global Average Pooling.
- Nueva cabeza clasificadora de cinco clases.
- Tasa de aprendizaje inicial: 0,001.

**Propósito:** comparar una segunda arquitectura preentrenada bajo un régimen equivalente al de E1.

## 6. Comparaciones experimentales

**Comparación principal:** E1 frente a E3.

Se compararán dos arquitecturas diferentes, ambas utilizadas como extractores de características congelados.

**Comparación secundaria:** E1 frente a E2.

Se evaluará el efecto del fine-tuning parcial sobre una misma arquitectura.

**Comparación de referencia:** E0 frente a E1, E2 y E3.

Permitirá establecer las diferencias respecto a un modelo entrenado desde cero.

## 7. Métricas de evaluación

La métrica principal será macro-F1, debido a la distribución desigual de las clases.

También se calcularán:

- Accuracy.
- Precision, recall y F1 por clase.
- Matriz de confusión.
- Curvas de pérdida y accuracy.
- Tiempo total de entrenamiento.
- Tiempo de inferencia por imagen.
- Cantidad de parámetros y tamaño del modelo.

Se utilizará el conjunto de validación durante el desarrollo y el conjunto de prueba únicamente para la evaluación final.

## 8. Experimento adicional de robustez

Se propone evaluar el efecto de una perturbación moderada de brillo sobre copias de las imágenes del conjunto de prueba, después de congelar la configuración de los modelos.

Se comparará el desempeño con las imágenes originales y con las imágenes modificadas, sin volver a entrenar los modelos.

La transformación propuesta será multiplicar la intensidad de la imagen por 0,8, limitando los valores al intervalo válido.

Se mantendrán las etiquetas originales y se documentará la transformación aplicada.

## 9. Análisis de errores

Se inspeccionarán al menos ocho predicciones incorrectas reales.

Para cada caso se registrarán:

- Imagen original.
- Clase verdadera.
- Clase predicha.
- Confianza de la predicción.
- Observaciones visuales.
- Posibles explicaciones de la confusión.

Las explicaciones se presentarán como hipótesis, sin afirmar causalidad a partir de una inspección visual.

## 10. Reproducibilidad

Cada experimento tendrá un notebook independiente, configuración registrada y resultados exportados.

Se conservarán:

- Semilla aleatoria.
- Versiones de dependencias.
- Particiones definitivas.
- Hiperparámetros.
- Historial de entrenamiento.
- Métricas.
- Matrices de confusión.
- Tiempos de ejecución.
- Checkpoints, cuando sea posible.

## 11. Limitaciones

Los resultados estarán condicionados por el tamaño y la procedencia del dataset, el desbalance de clases, la calidad de las etiquetas y los recursos disponibles en Google Colab.

Las diferencias entre modelos podrán depender de la arquitectura, los pesos preentrenados y las condiciones de entrenamiento.

Una única ejecución por configuración no permite estimar completamente la variabilidad experimental.

Los resultados no constituyen una validación agronómica del diagnóstico de enfermedades en cultivos colombianos.

# Comparación experimental de modelos preentrenados para reconocer enfermedades de la yuca

## Descripción del proyecto

Este proyecto presenta una comparación experimental de modelos de aprendizaje profundo para la clasificación de enfermedades de la yuca mediante fotografías de hojas.

Se evalúan diferentes estrategias de aprendizaje, incluyendo el entrenamiento desde cero, el aprendizaje por transferencia y el ajuste fino (fine-tuning), utilizando arquitecturas convolucionales como MobileNetV2 y ResNet50.

El estudio considera tanto el rendimiento predictivo como el costo computacional y la robustez de los modelos frente a variaciones de iluminación.

## Objetivos

### Objetivo general

Comparar experimentalmente diferentes modelos de aprendizaje profundo para el reconocimiento de enfermedades de la yuca mediante imágenes de hojas, considerando su rendimiento predictivo, eficiencia computacional y robustez.

### Objetivos específicos

- Realizar un análisis exploratorio y una depuración del conjunto de datos.
- Implementar una red neuronal convolucional como modelo baseline.
- Evaluar arquitecturas preentrenadas mediante aprendizaje por transferencia.
- Analizar el impacto del fine-tuning sobre el rendimiento predictivo.
- Comparar los modelos bajo condiciones experimentales controladas.
- Evaluar la robustez de los modelos ante variaciones de iluminación.

## Tecnologías utilizadas

- Python
- TensorFlow
- Keras
- TensorFlow Datasets
- NumPy
- Pandas
- Scikit-learn
- Matplotlib
- Seaborn
- Google Colab
- Git y GitHub

## Conjunto de datos

Se utilizó el dataset Cassava, disponible en TensorFlow Datasets, versión `0.1.0`.

Fuente: https://www.tensorflow.org/datasets/catalog/cassava

El conjunto de datos contiene fotografías de hojas de yuca clasificadas en cinco categorías.

| Clase | Descripción |
|---|---|
| CBB | Cassava Bacterial Blight |
| CBSD | Cassava Brown Streak Disease |
| CGM | Cassava Green Mite |
| CMD | Cassava Mosaic Disease |
| Healthy | Hojas saludables |

Durante el análisis exploratorio se identificaron imágenes duplicadas y visualmente similares mediante SHA-256 y hashing perceptual (pHash).

Después del proceso de depuración, se conservaron 9.361 imágenes de las 9.430 originales.

### Distribución definitiva

| Clase | Entrenamiento | Validación | Prueba | Total |
|---|---:|---:|---:|---:|
| CBB | 455 | 151 | 154 | 760 |
| CBSD | 1426 | 479 | 477 | 2382 |
| CGM | 762 | 257 | 253 | 1272 |
| CMD | 2652 | 885 | 885 | 4422 |
| Healthy | 315 | 105 | 105 | 525 |
| **Total** | **5610** | **1877** | **1874** | **9361** |

Debido al desbalance entre las clases, se estableció Macro-F1 como métrica principal de evaluación.

## Diseño experimental

Se desarrollaron cuatro experimentos principales.

| Experimento | Arquitectura | Estrategia |
|---|---|---|
| E0 | CNN personalizada | Entrenamiento desde cero |
| E1 | MobileNetV2 | Backbone preentrenado congelado |
| E2 | MobileNetV2 | Fine-tuning a partir de E1 |
| E3 | ResNet50 | Backbone preentrenado congelado |

### E0 — Modelo baseline

Red neuronal convolucional personalizada, entrenada desde cero, utilizada como referencia para determinar el aporte del aprendizaje por transferencia.

### E1 — MobileNetV2 congelada

Arquitectura MobileNetV2 preentrenada con ImageNet. Se mantuvieron congeladas las capas del backbone y se entrenó únicamente la cabeza de clasificación.

### E2 — Fine-tuning de MobileNetV2

Se utilizaron los pesos obtenidos en E1 como punto de partida. Se descongelaron parcialmente las últimas capas del backbone, manteniendo congeladas las capas de Batch Normalization y utilizando una tasa de aprendizaje menor.

### E3 — ResNet50 congelada

Arquitectura ResNet50 preentrenada con ImageNet. Se mantuvo congelado el backbone y se entrenó únicamente la cabeza de clasificación para comparar su rendimiento con MobileNetV2.

## Configuración experimental

| Parámetro | Valor |
|---|---|
| Semilla | 42 |
| Resolución de entrada | 224 × 224 píxeles |
| Batch size | 8 |
| Número de clases | 5 |
| Optimizador | Adam |
| Función de pérdida | Sparse Categorical Crossentropy |
| Learning rate E0, E1 y E3 | 0.001 |
| Learning rate E2 | 0.00001 |
| Métrica principal | Macro-F1 |
| Entorno de ejecución | Google Colab |
| Aumento de datos durante el entrenamiento | No |

Se utilizaron las mismas particiones de entrenamiento, validación y prueba para todos los experimentos, respetando el preprocesamiento específico de cada arquitectura.

El conjunto de prueba se mantuvo reservado hasta finalizar los cuatro experimentos principales.

## Resultados finales

Los siguientes resultados corresponden a la evaluación definitiva sobre las 1.874 imágenes del conjunto de prueba.

| Modelo | Accuracy | Macro-F1 |
|---|---:|---:|
| E0 — CNN baseline | 67.72 % | 45.66 % |
| E1 — MobileNetV2 | 76.20 % | 66.00 % |
| E2 — MobileNetV2 fine-tuning | **81.22 %** | **74.03 %** |
| E3 — ResNet50 | 80.68 % | 72.73 % |

### Comparaciones experimentales

| Comparación | Variación Macro-F1 | Variación Accuracy |
|---|---:|---:|
| E1 frente a E0 | +20.34 pp | +8.48 pp |
| E2 frente a E1 | +8.03 pp | +5.02 pp |
| E3 frente a E1 | +6.73 pp | +4.48 pp |

Los resultados muestran que el aprendizaje por transferencia mejora considerablemente el rendimiento respecto al modelo baseline.

El fine-tuning de MobileNetV2 permitió obtener el mejor rendimiento predictivo observado entre los cuatro experimentos.

## Eficiencia computacional

| Modelo | Entrenamiento (s) | Épocas ejecutadas | Inferencia (ms/imagen) | Parámetros | Pesos (MiB) |
|---|---:|---:|---:|---:|---:|
| E0 | 495.67 | 15 | 4.82 | 110405 | 0.42 |
| E1 | 286.50 | 7 | 10.09 | 2422597 | 9.24 |
| E2 | 228.55 | 7 | 5.48 | 2422597 | 9.24 |
| E3 | 294.63 | 7 | 11.11 | 23850641 | 90.98 |

El tiempo de entrenamiento de E2 corresponde únicamente a la etapa de fine-tuning. Su tiempo acumulado, incluyendo el entrenamiento de E1, fue de 515.06 segundos.

ResNet50 posee aproximadamente 9.85 veces más parámetros que MobileNetV2.

E2 presentó un equilibrio favorable entre rendimiento predictivo, tamaño del modelo y tiempo de inferencia observado.

Los tiempos de inferencia deben interpretarse considerando la variabilidad de los recursos computacionales disponibles en Google Colab.

## Experimento adicional de robustez

Se realizó un experimento controlado para evaluar la sensibilidad de los modelos ante variaciones de iluminación.

Se aplicó un factor multiplicativo de 0.8 sobre los valores de los píxeles de las imágenes del conjunto de prueba, equivalente a una reducción del 20 % de su intensidad.

La transformación se realizó sin modificar los pesos, las etiquetas ni las particiones del conjunto de datos.

### Resultados de robustez

| Modelo | Accuracy original | Accuracy oscurecida | Macro-F1 original | Macro-F1 oscurecido | Variación Macro-F1 |
|---|---:|---:|---:|---:|---:|
| E0 | 67.72 % | 66.92 % | 45.66 % | 44.68 % | -0.97 pp |
| E1 | 76.20 % | 75.72 % | 66.00 % | 65.09 % | -0.91 pp |
| E2 | 81.22 % | 80.26 % | 74.03 % | 72.45 % | -1.58 pp |
| E3 | 80.68 % | 81.06 % | 72.73 % | 72.76 % | +0.03 pp |

ResNet50 presentó la mayor estabilidad frente a la perturbación aplicada, mientras que MobileNetV2 con fine-tuning presentó el mejor rendimiento sobre las imágenes originales.

El ligero incremento observado en E3 no constituye evidencia suficiente para afirmar que reducir el brillo mejora sistemáticamente su rendimiento.

## Estructura del repositorio

```text
cassava-disease-model-comparison/
│
├── docs/
│   ├── 01_fuente_datos.md
│   ├── 02_inventario_clases.md
│   ├── 03_protocolo_experimental.md
│   ├── 04_hipotesis.md
│   └── bitacora.md
│
├── notebooks/
│   ├── Análisis exploratorio de datos
│   ├── Entrenamiento E0
│   ├── 03_entrenamiento_E1.ipynb
│   ├── 04_entrenamiento_E2.ipynb
│   ├── 05_entrenamiento_E3.ipynb
│   ├── 06_evaluacion_final.ipynb
│   └── 07_experimento_robustez.ipynb
│
├── results/
│   ├── eda/
│   ├── e0/
│   ├── e1/
│   ├── e2/
│   ├── e3/
│   ├── evaluacion_final/
│   └── robustez/
│
└── README.md
```

## Instrucciones de ejecución

### 1. Clonar el repositorio

```bash
git clone https://github.com/ThePretelR/cassava-disease-model-comparison.git

cd cassava-disease-model-comparison
```

### 2. Instalar las dependencias

```bash
pip install tensorflow tensorflow-datasets numpy pandas scikit-learn matplotlib seaborn pillow imagehash psutil importlib_resources
```

También es posible ejecutar los notebooks directamente en Google Colab.

### 3. Configurar el entorno

Se recomienda utilizar un entorno con aceleración por GPU para reducir los tiempos de entrenamiento e inferencia.

En Google Colab:

```text
Entorno de ejecución > Cambiar tipo de entorno de ejecución > GPU
```

### 4. Ejecutar los notebooks

Los notebooks deben ejecutarse en el siguiente orden:

1. Análisis exploratorio y depuración del dataset.
2. Entrenamiento del modelo E0.
3. Entrenamiento del modelo E1.
4. Entrenamiento del modelo E2.
5. Entrenamiento del modelo E3.
6. Evaluación final.
7. Experimento de robustez.

El experimento E2 requiere el checkpoint generado durante E1.

### 5. Evaluar los modelos entrenados

Para ejecutar la evaluación final y el experimento de robustez sin repetir los entrenamientos, se requieren los siguientes checkpoints:

```text
mejor_modelo_e0.keras
mejor_modelo_e1.keras
mejor_modelo_e2.keras
mejor_modelo_e3.keras
```

En Google Colab, los archivos deben ubicarse en el directorio `/content/`.

Los notebooks utilizan el manifiesto definitivo de imágenes conservadas para reproducir las particiones establecidas durante el análisis exploratorio.

## Evidencias experimentales

El repositorio contiene las evidencias generadas durante el desarrollo del proyecto:

- Análisis exploratorio del conjunto de datos.
- Distribución de clases.
- Registro de imágenes excluidas.
- Manifiesto definitivo de particiones.
- Historiales de entrenamiento.
- Curvas de aprendizaje.
- Matrices de confusión.
- Métricas individuales por clase.
- Predicciones de los modelos.
- Inspección de errores de clasificación.
- Comparaciones de rendimiento.
- Mediciones de costo computacional.
- Resultados del experimento de robustez.
- Bitácora del proceso experimental.

## Conclusiones

- El aprendizaje por transferencia permitió mejorar considerablemente el rendimiento respecto a la CNN entrenada desde cero.
- MobileNetV2 con fine-tuning obtuvo el mejor rendimiento predictivo, alcanzando un accuracy de 81.22 % y un Macro-F1 de 74.03 %.
- ResNet50 superó a MobileNetV2 congelada, aunque presentó un mayor costo en términos de parámetros y almacenamiento.
- MobileNetV2 con fine-tuning presentó un equilibrio favorable entre rendimiento predictivo y eficiencia computacional.
- ResNet50 presentó la mayor robustez observada frente a la reducción controlada del brillo.
- El desbalance de clases evidencia la importancia de utilizar Macro-F1 y métricas individuales por clase, en lugar de depender únicamente del accuracy.

## Limitaciones

- El conjunto de datos presenta un desbalance considerable entre las clases.
- Los experimentos se realizaron utilizando una única semilla.
- No se realizaron pruebas de significancia estadística.
- La evaluación de robustez consideró únicamente un nivel de reducción del brillo.
- Los tiempos de ejecución pueden variar según los recursos computacionales disponibles.
- No se realizó una validación externa con fotografías de cultivos colombianos.
- Los resultados no garantizan el mismo rendimiento bajo condiciones agrícolas reales.

El proyecto tiene un propósito académico y experimental. Los modelos no deben considerarse sustitutos de un diagnóstico fitosanitario profesional.

*Uso de inteligencia artificial*
Durante el desarrollo del proyecto se utilizaron herramientas de inteligencia artificial generativa solo de forma puntual y acotada: para resolver algunas soluciones de código, depurar errores concretos de programación y apoyar la documentación de los notebooks.

La IA no se empleó para la toma de decisiones técnicas (elección de protocolos, modelos, parámetros o diseño de los experimentos), ni para la interpretación de métricas, ni para la redacción de hipótesis o conclusiones; todo ello fue elaborado por los integrantes del proyecto.

Los resultados experimentales fueron obtenidos mediante la ejecución de los notebooks en Google Colab.

Las decisiones metodológicas, interpretaciones y conclusiones son de autoría propia del equipo.

## Referencias

- TensorFlow Datasets — Cassava Dataset.
  https://www.tensorflow.org/datasets/catalog/cassava

- Sandler, M., Howard, A., Zhu, M., Zhmoginov, A., & Chen, L. C. (2018). MobileNetV2: Inverted Residuals and Linear Bottlenecks.
  https://arxiv.org/abs/1801.04381

- He, K., Zhang, X., Ren, S., & Sun, J. (2016). Deep Residual Learning for Image Recognition.
  https://arxiv.org/abs/1512.03385

- Keras — Transfer Learning and Fine-tuning.
  https://keras.io/guides/transfer_learning/

## Autores

**Universidad de Córdoba**  
Facultad de Ingenierías  
Programa de Ingeniería de Sistemas  
Montería, Córdoba, Colombia  
2026

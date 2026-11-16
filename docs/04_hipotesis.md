# Hipótesis experimentales

**Proyecto:** Comparación experimental de modelos preentrenados para reconocer enfermedades de la yuca mediante fotografías de hojas.

**Fecha:** 14 de noviembre de 2026

**Dataset:** Cassava 0.1.0

**Framework:** TensorFlow/Keras

**Estado:** Hipótesis formuladas antes del entrenamiento de los modelos E0–E3.

## 1. Objetivo

Establecer hipótesis verificables sobre el desempeño de diferentes arquitecturas de redes neuronales convolucionales para clasificar fotografías de hojas de yuca en cinco categorías.

Las hipótesis permitirán orientar la comparación experimental entre un modelo base, dos arquitecturas preentrenadas y un modelo sometido a ajuste fino parcial.

La métrica principal será macro-F1, complementada con accuracy, recall por clase y costo computacional.

## 2. Hipótesis general

Los modelos convolucionales preentrenados en ImageNet obtendrán un mejor desempeño de clasificación que una CNN pequeña entrenada desde cero, debido a la reutilización de características visuales aprendidas previamente.

Sin embargo, las diferencias de rendimiento dependerán de la arquitectura, la estrategia de entrenamiento y la distribución de las clases.

## 3. Hipótesis por experimento

### H1. Experimento E0: Baseline CNN

**Hipótesis:** Una CNN pequeña entrenada desde cero podrá aprender características visuales útiles para clasificar enfermedades de la yuca, pero tendrá un desempeño macro-F1 inferior al de los modelos preentrenados.

**Fundamento:** El modelo deberá aprender sus características visuales utilizando únicamente las imágenes del conjunto de entrenamiento, mientras que los modelos preentrenados disponen de representaciones adquiridas previamente.

**Predicción verificable:** El macro-F1 de E0 será inferior al de E1 y E3 en el conjunto de prueba.

### H2. Experimento E1: MobileNetV2 congelada

**Hipótesis:** MobileNetV2, utilizada como extractor de características congelado, superará el desempeño de la CNN base y ofrecerá un costo computacional relativamente bajo.

**Fundamento:** MobileNetV2 fue diseñada para utilizar recursos computacionales de manera eficiente y permite reutilizar representaciones visuales aprendidas en ImageNet.

**Predicción verificable:** E1 obtendrá un macro-F1 superior al de E0 y requerirá menos tiempo de inferencia por imagen que E3, bajo las condiciones de hardware definidas.

### H3. Experimento E2: Fine-tuning parcial

**Hipótesis:** Descongelar parcialmente las últimas capas de MobileNetV2 permitirá adaptar mejor las características visuales al dominio de las hojas de yuca, mejorando el desempeño respecto a E1.

**Fundamento:** El ajuste fino permite modificar parte de las representaciones previamente aprendidas para responder a las características del nuevo conjunto de datos.

**Predicción verificable:** E2 obtendrá un macro-F1 superior al de E1, aunque requerirá un tiempo adicional de entrenamiento.

### H4. Experimento E3: ResNet50 congelada

**Hipótesis:** ResNet50 podrá alcanzar un desempeño competitivo o superior al de MobileNetV2 en la clasificación de enfermedades de la yuca, pero con mayor costo computacional.

**Fundamento:** ResNet50 dispone de una arquitectura residual profunda y una mayor capacidad de representación, aunque esto no garantiza una mejor generalización en todos los conjuntos de datos.

**Predicción verificable:** E3 obtendrá un macro-F1 igual o superior al de E1, pero tendrá un mayor número de parámetros y un tiempo de inferencia superior.

## 4. Hipótesis del experimento adicional

### H5. Robustez ante variaciones de brillo

**Hipótesis:** Una reducción moderada del brillo de las imágenes provocará una disminución del rendimiento de clasificación de los modelos evaluados.

**Fundamento:** Las modificaciones de iluminación pueden alterar la visibilidad de patrones de color, textura y lesiones que las redes neuronales utilizan para clasificar imágenes.

**Predicción verificable:** El macro-F1 calculado sobre las imágenes con brillo reducido será inferior al obtenido sobre las imágenes originales.

La perturbación consistirá en multiplicar los valores de intensidad por un factor de 0,8, respetando el intervalo válido de píxeles.

## 5. Comparaciones previstas

| Comparación | Propósito |
|---|---|
| E0 frente a E1 | Evaluar el aporte del aprendizaje por transferencia |
| E1 frente a E2 | Evaluar el efecto del fine-tuning parcial |
| E1 frente a E3 | Comparar dos arquitecturas preentrenadas bajo un protocolo equivalente |
| Imágenes originales frente a modificadas | Evaluar sensibilidad a una variación de brillo |

## 6. Criterios de evaluación

Las hipótesis se contrastarán mediante resultados experimentales.

Se utilizarán:

- Macro-F1 como métrica principal.
- Accuracy global.
- Recall y F1 por clase.
- Matrices de confusión.
- Tiempo de entrenamiento.
- Tiempo de inferencia.
- Número de parámetros y tamaño del modelo.

Una hipótesis se considerará respaldada cuando los resultados observados coincidan con la dirección de su predicción. En caso contrario, se considerará no respaldada por la ejecución realizada.

Dado que inicialmente se contempla una ejecución por configuración, las diferencias observadas no se interpretarán automáticamente como estadísticamente significativas.

## 7. Consideraciones metodológicas

Las hipótesis se establecen antes de entrenar los modelos E0–E3.

La selección de hiperparámetros se realizará utilizando el conjunto de validación, sin utilizar las etiquetas del conjunto de prueba para tomar decisiones de entrenamiento.

Antes de ejecutar los experimentos se verificará el tratamiento de imágenes duplicadas y posibles inconsistencias entre particiones.

Los resultados permitirán aceptar provisionalmente o rechazar las predicciones planteadas, sin asumir de antemano que una arquitectura específica será la mejor.

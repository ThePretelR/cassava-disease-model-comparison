# Inventario de clases del dataset Cassava

## 1. Objetivo

Identificar las categorías del conjunto de datos Cassava y registrar la distribución de imágenes del conjunto de entrenamiento.

## 2. Categorías identificadas

El dataset contiene cinco categorías:

| Identificador | Etiqueta | Descripción |
|---|---|---|
| 0 | cbb | Cassava Bacterial Blight |
| 1 | cbsd | Cassava Brown Streak Disease |
| 2 | cgm | Cassava Green Mottle |
| 3 | cmd | Cassava Mosaic Disease |
| 4 | healthy | Hoja saludable |

Los identificadores corresponden al orden de etiquetas obtenido mediante TensorFlow Datasets.

## 3. Distribución de entrenamiento

| Clase | Imágenes | Porcentaje |
|---|---:|---:|
| CBB | 466 | 8,24 % |
| CBSD | 1.443 | 25,51 % |
| CGM | 773 | 13,67 % |
| CMD | 2.658 | 46,99 % |
| Healthy | 316 | 5,59 % |
| **Total** | **5.656** | **100 %** |

Los porcentajes están redondeados a dos decimales.

## 4. Características generales de las imágenes

Se analizaron las dimensiones originales de las fotografías del conjunto de entrenamiento.

- Imágenes analizadas: 5.656.
- Resoluciones diferentes: 574.
- Altura promedio: 572,88 píxeles.
- Anchura promedio: 610,93 píxeles.
- Relación ancho/alto promedio: 1,12.

También se calcularon estadísticas de brillo y valores medios de los canales RGB para cada clase.

## 5. Control preliminar de duplicados

Se analizaron 9.430 imágenes mediante hashes SHA-256.

**Resultados:**

- Hashes únicos: 9.388.
- Hashes compartidos entre entrenamiento y validación: 11.
- Hashes compartidos entre entrenamiento y prueba: 10.
- Hashes compartidos entre validación y prueba: 4.

En el análisis mediante hash perceptual (pHash) se identificaron 9.381 valores únicos y 82 registros asociados a valores repetidos.

Estas coincidencias deben revisarse visualmente. También se observaron imágenes idénticas asociadas a etiquetas diferentes, por lo que es necesario investigar posibles inconsistencias de etiquetado.

## 6. Evidencia reproducible

Los análisis fueron realizados mediante Python, TensorFlow Datasets, NumPy, Pandas, Matplotlib e ImageHash en Google Colab.

El notebook de exploración y los resultados exportados permitirán reproducir los análisis.

**Estado:** Inventario inicial completado. Verificación de duplicados y consistencia de etiquetas pendiente.

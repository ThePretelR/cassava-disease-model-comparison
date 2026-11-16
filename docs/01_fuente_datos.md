# Fuente de datos del proyecto

## 1. Identificación del conjunto de datos

**Proyecto:** Comparación experimental de modelos preentrenados para reconocer enfermedades de la yuca mediante fotografías de hojas.

**Dataset seleccionado:** Cassava (iCassava 2019).

**Fuente:** TensorFlow Datasets (TFDS).

**URL:** https://www.tensorflow.org/datasets/catalog/cassava

**Versión utilizada:** 0.1.0.

**Biblioteca de desarrollo:** TensorFlow/Keras.

**Entorno:** Google Colab.

## 2. Descripción del dataset

El conjunto de datos Cassava contiene fotografías de hojas de yuca clasificadas en cinco categorías, correspondientes a cuatro enfermedades y una categoría de hojas saludables.

El dataset se utiliza en este proyecto para comparar experimentalmente modelos de clasificación de imágenes mediante aprendizaje profundo.

## 3. Cantidad de imágenes

| Partición | Cantidad |
|---|---:|
| Entrenamiento | 5.656 |
| Validación | 1.889 |
| Prueba | 1.885 |
| **Total** | **9.430** |

Se utilizarán las particiones publicadas por TensorFlow Datasets, sujetas a la revisión de calidad y posibles fugas de información.

## 4. Verificación de descarga

La descarga y preparación del dataset se realizaron correctamente en Google Colab mediante `tensorflow_datasets`.

**Versión verificada:** `cassava:0.1.0`.

**Resultado:** 9.430 imágenes cargadas y distribuidas en entrenamiento, validación y prueba.

Durante la primera ejecución se presentó un error relacionado con la ausencia de la biblioteca `importlib_resources`. Después de instalar la dependencia, la carga del dataset se completó correctamente.

## 5. Resoluciones

En las 5.656 imágenes de entrenamiento se encontraron 574 resoluciones diferentes.

| Característica | Valor |
|---|---:|
| Altura mínima | 347 píxeles |
| Altura máxima | 1.240 píxeles |
| Anchura mínima | 420 píxeles |
| Anchura máxima | 1.622 píxeles |
| Altura promedio | 572,88 píxeles |
| Anchura promedio | 610,93 píxeles |

## 6. Procedencia y licencia

**Procedencia:** iCassava 2019, distribuido mediante TensorFlow Datasets.

**Licencia:** Pendiente de verificar y registrar a partir de la documentación original.

No se incorporarán imágenes de otras fuentes sin documentar su procedencia y compatibilidad.

## 7. Control de calidad

Se realizó una revisión exploratoria inicial que incluyó distribución de clases, dimensiones, brillo, canales RGB y detección de duplicados.

Se identificaron coincidencias exactas entre particiones y algunos casos con etiquetas diferentes para el mismo contenido visual. Estos hallazgos requieren verificación antes de definir el tratamiento definitivo de los datos.

**Estado:** Dataset descargado y verificado; revisión de integridad y duplicados pendiente de finalizar.

## 8. Referencia

TensorFlow Datasets. *Cassava*. https://www.tensorflow.org/datasets/catalog/cassava

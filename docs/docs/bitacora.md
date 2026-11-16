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

## Próximas actividades

- [ ] Verificar las imágenes duplicadas y los conflictos de etiquetas.
- [ ] Finalizar el EDA y exportar evidencias.
- [ ] Confirmar las particiones definitivas.
- [ ] Implementar y entrenar E0.
- [ ] Iniciar E1 con MobileNetV2.
- [ ] Registrar métricas y tiempos de ejecución.
- [ ] Preparar evidencias para la revisión intermedia.

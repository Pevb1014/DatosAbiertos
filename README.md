# 🛠️ Procesamiento y Limpieza de Datos Viales (Notebook Colab)

Este documento describe el proceso de pre-procesamiento, limpieza y estandarización de tres conjuntos de datos clave relacionados con la señalización vial. El objetivo es preparar los datos para análisis posteriores, asegurando la consistencia y completitud de la información.

## 📦 Datasets Procesados

El notebook aborda la limpieza de los siguientes archivos CSV:

1.  **`PR Señal Horizontal.csv`**: Datos de señalización horizontal (marcas viales).
2.  **`Postes Referencia.csv`**: Datos de puntos de referencia geolocalizados.
3.  **`PR Señal Vertical.csv`**: Datos de señalización vertical (letreros).
4.  **`SIV Señal Vertical.csv`**: Datos con problemas de formato de texto (líneas partidas).

---

## ⚙️ Estructura del Procesamiento

### 1. PR Señal Horizontal

Este proceso se centra en la estandarización de los puntos de referencia (`PR`).

* **Verificación Inicial**: Se verifica la cantidad de separadores (comas) por línea para detectar registros mal formados.
* **Eliminación de Columnas**: Se eliminan columnas consideradas no relevantes:
    * `objectid`, `codigo`, `ubicacion`, `clasificac`, `tipo_pintu`, `territoria`, `shape_leng`, `z`, `st_length(shape)`.
* **Manejo de Nulos y PR**:
    * Se eliminan registros sin `via` o aquellos que carecen de `pr_inicial` y `pr_final` a la vez.
    * **Imputación Lógica de PR**:
        * Si `pr_inicial` es nulo, se establece como `pr_final - 1` (mínimo 0).
        * Si `pr_final` es nulo, se establece como `pr_inicial + 1`.
* **Ingeniería de Características**: Se extrae el **número** y el **año** del contrato a partir de la columna `contrato` (ej: `123_2024` -> `numero_contrato: 123`, `anio_contrato: 2024`).
* **Salida**: `PR Señal Horizontal Limpio.xlsx`

### 2. Postes Referencia

Este proceso se enfoca en la validación y enriquecimiento geográfico de los postes.

* **Verificación Inicial**: Se verifica la cantidad de separadores.
* **Eliminación de Columnas**: Se eliminan columnas sin relevancia:
    * `objectid`, `distancia`, `lado`, `material`, `fechainstalacion`, `fuente`, `fechafuente`, `altura`, `estadociclovida`.
* **Conversión de Tipos y Limpieza**:
    * Columnas como `postereferencia`, `estado`, `latitud`, y `longitud` se convierten a tipos numéricos, forzando errores a `NaN`.
    * Se eliminan registros con valores nulos en `latitud` o `longitud`.
* **Filtro Geográfico**: Se eliminan registros cuyas coordenadas (`latitud`/`longitud`) están fuera del rango geográfico plausible para Colombia (aprox. **Latitud** entre -4.3 y 13.5; **Longitud** entre -82 y -66).
* **Enriquecimiento**: Se añade la columna `maps` con un hipervínculo de Excel a Google Maps para visualizar la ubicación.
* **Salida**: `Postes Referencia Limpio.xlsx`

### 3. PR Señal Vertical

Proceso similar al de Señal Horizontal, con un manejo de fechas específico.

* **Verificación Inicial**: Se verifica la cantidad de separadores.
* **Eliminación de Columnas**: Se eliminan columnas sin relevancia:
    * `objectid`, `codigo`, `lado`, `tipo_senal`, `codigo_sen`, `forma_sena`, `territoria`, `estado`, `limpieza`, `cimiento`, `soporte`, `visibilida`, `mat_placa`, `sustento`, `z`.
* **Filtro de Nulos**: Se eliminan registros que no tengan una `via` válida o que no tengan un `pr`.
* **Conversión de Fecha**: La columna `fecha_insp` (almacenada en milisegundos) se convierte a un objeto de fecha, aplicando la zona horaria **`America/Bogota`**.
* **Ingeniería de Características**: Se extrae el **número** y el **año** del contrato, similar al punto 1.
* **Salida**: `PR Señal Vertical Limpio.xlsx`

### 4. SIV Señal Vertical (Corrección de CSV)

Este proceso utiliza utilidades de Python (módulos `csv` y lógica de patrones) para corregir problemas de formato.

* **Detección de Continuación**: Se aplica lógica de patrones (basada en si la línea está vacía, empieza con coma o no empieza con un dígito) para identificar y unir registros que se encuentran **divididos en múltiples líneas**.
* **Corrección de Comas Internas**: Se manejan los casos donde el texto de una columna contiene comas, lo que resulta en un número de columnas mayor al esperado. Estos fragmentos se unen y se limpian antes de guardarlos en la columna de texto.
* **Salida**: `SIV Señal Vertical_Limpio.csv` (CSV, no Excel, debido a la naturaleza de la corrección de formato).

---

## 💻 Dependencias

Este notebook requiere las siguientes librerías de Python:

* `pandas`
* `numpy`
* `csv`
* `statistics` (módulo base de Python)

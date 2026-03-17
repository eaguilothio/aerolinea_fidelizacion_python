# NOTAS_PROCESO — Aerolinea_Fidelidad_PY

Registro técnico del proceso completo. Útil para replicar el proyecto o como referencia futura.

---

## Fase 0: Comprensión de los datos

Antes de escribir una sola línea de código, se leen y documentan los dos archivos para entender qué contiene cada uno y cuál es su relación.

1. **Customer_Flight_Activity.csv** — actividad mensual del cliente dentro del programa:
    - `loyalty_number` — clave de unión
   - `total_flights` — variable clave: actividad de vuelo mensual del cliente

2. **Customer_Profile.csv** — perfil demográfico y posición dentro del programa:
   - `loyalty_number` — clave de unión
   - `loyalty_card` — tipo de tarjeta: Star, Nova o Aurora
   - `clv` — Customer Lifetime Value: valor total que aporta el cliente a la aerolínea
   - `province` — ubicación geográfica del cliente

---

## Fase 1: Exploración y limpieza

Se aplica el mismo esquema a cada archivo por separado antes de unirlos:
**dimensión → tipos de datos y nulos → inspección → calidad → limpieza → guardar**

3. **Dimensión:** `df.shape` informa del número de filas y columnas. `df.columns.tolist()` lista los nombres exactos de cada variable.

4. **Tipos de datos y nulos:** `df.info()` muestra el tipo de dato de cada columna y cuántos valores no nulos tiene. Es el primer paso para detectar columnas mal tipadas o con datos faltantes.

5. **Inspección visual:** `df.head()` y `df.tail()` sirven para verificar que la carga fue correcta y detectar valores anómalos al principio y al final del archivo.

6. **Limpieza:**
   - Crear siempre una copia antes de modificar: `df_clean = df_raw.copy()`
   - Eliminar duplicados: `df.drop_duplicates()`
   - Normalizar nombres a snake_case:

     ```python
     df.columns = df.columns.str.lower().str.replace(" ", "_")
     ```

     Evita errores con mayúsculas, espacios y facilita el autocompletado.
   - Seleccionar solo las columnas necesarias para el análisis — reduce ruido y peso del DataFrame.
   - Normalizar texto en columnas categóricas:

     ```python
     df[col] = df[col].str.lower().str.strip()
     ```

7. **Guardar los limpios:**

   ```python
   df_clean.to_csv(os.path.join(output_path, "nombre_clean.csv"), index=False)
   ```

   Los archivos originales en `data/` nunca se tocan. Los resultados van siempre a `output/`.

---

## Fase 2: Merge

8. **Unir los dos archivos:** Se usa LEFT JOIN para conservar toda la actividad de vuelo y añadir el perfil cuando existe.

   ```python
   df = flight_clean.merge(profile_clean, on='loyalty_number', how='left', validate='m:1')
   ```

   - `on='loyalty_number'` — columna de unión presente en ambos archivos
   - `how='left'` — conserva todos los registros del archivo de vuelo aunque no tengan perfil
   - `validate='m:1'` — garantiza que cada `loyalty_number` aparece una sola vez en el perfil; si hay duplicados, Pandas lanza error en lugar de silenciar el problema

   > Antes del merge, convertir la columna de unión al mismo tipo en ambos DataFrames:
   > `df['loyalty_number'] = df['loyalty_number'].astype(str)`
   > Un int en un archivo y un string en el otro produce un join vacío sin ningún error visible.

---

## Fase 3: Perfil del cliente típico

El tipo de variable determina la herramienta de análisis:
- **Variable continua** → `describe()` + histograma + boxplot
- **Variable categórica** → `value_counts()` + gráfico de barras (> 4 categorías) o quesito (≤ 4 categorías)

9. **`total_flights` — ¿cuánto vuela un cliente al mes?** Variable continua.

    ```python
    flight_clean['total_flights'].describe().round(2)
    sns.histplot(...)   # distribución
    sns.boxplot(...)    # outliers
    ```

    La distribución es asimétrica con cola derecha. La mediana es más representativa que la media.

10. **`clv` — ¿cuánto vale un cliente?** Variable continua.

    ```python
    profile_clean['clv'].describe().round(2)
    ```

    Distribución también asimétrica. Un grupo pequeño concentra una parte desproporcionada del valor.

11. **`province` — ¿dónde viven los clientes?** Variable categórica con 11 valores → gráfico de barras horizontal.

    ```python
    prov = profile_clean['province'].value_counts()
    (prov / prov.sum() * 100).round(1)
    sns.barplot(x=prov.values, y=prov.index, palette="Blues_d")
    ```

---

## Fase 4: ¿Quién es el cliente más valioso?

12. **Evitar la relación 1:m en el análisis de perfil:** El merge produce un DataFrame donde cada cliente aparece tantas veces como registros de vuelo tiene. Para analizar variables del perfil (como CLV), hay que eliminar los duplicados primero:

    ```python
    clientes = df.drop_duplicates('loyalty_number')
    ```

    Si no se hace, los cálculos de CLV estarán inflados — cada cliente se contará múltiples veces.

13. **CLV por tarjeta:**

    ```python
    clientes.groupby('loyalty_card')['clv'].median().sort_values(ascending=False)
    ```

    Se usa la mediana en lugar de la media porque la distribución del CLV tiene outliers que distorsionan el promedio.

14. **Vuelos por tarjeta:**

    ```python
    df.groupby('loyalty_card')['total_flights'].median()
    ```
    Se usa la mediana en lugar de la media porque la distribución del total flights tiene outliers que distorsionan el promedio.

15. **Distribución geográfica de Aurora:**

    ```python
    aurora = clientes[clientes['loyalty_card'] == 'aurora']
    (aurora['province'].value_counts() / len(aurora) * 100).round(1)
    ```

    Filtrar primero, luego calcular porcentajes sobre el subconjunto — no sobre el total de clientes.

# Programa de fidelización de una aerolínea con python

Análisis del perfil del cliente más valioso del programa de fidelización de una aerolínea canadiense.  
**Herramienta:** Python · Pandas · Seaborn

---

## Objetivo

Identificar quién es el cliente de alto valor, cuánto vuela y dónde vive —  
para que la aerolínea sepa exactamente dónde y cómo invertir en retención.

---

## Dataset

- **Fuente:** Dos archivos CSV del programa de fidelización de una aerolínea canadiense
- **Registros:** Actividad mensual de vuelo + perfil demográfico por cliente
- **Variables clave:** `total_flights`, `clv`, `loyalty_card`, `province`

| Archivo | Descripción |
|---------|-------------|
| `Customer_Flight_Activity.csv` | Actividad mensual de vuelos |
| `Customer_Profile.csv` | Perfil del cliente |

---

## Fases del proyecto

| Fase | Descripción |
|------|-------------|
| 0. Comprensión de los datos | Revisión de variables y estructura de cada archivo |
| 1. Exploración y limpieza | Dimensión, tipos, nulos, calidad y limpieza por archivo |
| 2. Perfil del cliente típico | Análisis univariable de `total_flights`, `clv` y `province` |
| 3. Merge | Unión LEFT JOIN de actividad + perfil sobre `loyalty_number` |
| 4. Cliente más valioso | Cruce de `loyalty_card` con CLV, vuelos y distribución geográfica |

---

## Estructura del proyecto

```
📂 aerolinea_fidelizacion_python/
├── data/
│   ├── Customer_Flight_Activity.csv   → Dataset original — nunca se modifica
│   └── Customer_Profile.csv           → Dataset original — nunca se modifica
├── output/
│   ├── flight_clean.csv               → Actividad limpia y columnas seleccionadas
│   ├── profile_clean.csv              → Perfil limpio y columnas seleccionadas
│   └── df_aerolinea.csv               → DataFrame unificado para el análisis
└── aerolinea_archivo.ipynb            → Notebook principal
```

---

## Principales hallazgos

- **Actividad de vuelo:** La mayoría de clientes vuela poco cada mes. Hay un grupo reducido con actividad muy alta. La mediana es más representativa que la media.
- **CLV:** Un grupo pequeño de clientes concentra una parte desproporcionada del valor del negocio. Son el segmento a retener.
- **Geografía:** Ontario (32,3%) y British Columbia (26,3%) concentran el 58,6% de los clientes. El patrón se repite en todos los segmentos de tarjeta.
- **Aurora vs. el resto:** Aurora tiene el CLV más alto, pero vuela prácticamente igual que Star y Nova (diferencia de 1-2 vuelos al mes). La razón para retener a Aurora es económica, no de frecuencia — los incentivos basados en vuelos no funcionan para este segmento.
- **Acciones recomendadas:**
  - **Retener Aurora:** Beneficios premium — embarque prioritario, acceso a salas VIP y upgrades de clase sin coste. Atención personalizada, no masiva: el cliente VIP no quiere sentirse uno más.
  - **Hacer crecer Aurora:** Ontario y British Columbia concentran clientes en todos los niveles. Ofrecer una prueba temporal de beneficios Aurora a los clientes Star y Nova que alcancen un umbral de gasto.

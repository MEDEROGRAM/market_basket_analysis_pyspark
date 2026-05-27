# Market Basket Analysis: Escalando Reglas de Asociación de Pandas a PySpark

## 🛒 Caso de Estudio: Sano y Fresco | De la Lógica Analítica a la Ingeniería de Datos a Gran Escala

[![Apache Spark](https://shields.io)](https://apache.org)
[![Python](https://shields.io)](https://python.org)
[![Pandas](https://shields.io)](https://pydata.org)
[![Data Engineering](https://shields.io)](#)

---

## 1. Origen y Motivación del Proyecto
Este repositorio nace como una evolución de un excelente caso práctico impartido en la academia **DS4B (Data Science for Business)**. El proyecto original proporciona una base analítica y matemática impecable para entender el descubrimiento de reglas de asociación (*Market Basket Analysis*) utilizando algoritmos clásicos.

Mi motivación con este repositorio no es simplemente replicar el ejercicio, sino llevarlo un paso más allá aplicando prácticas de **Ingeniería de Datos y Big Data**. El objetivo principal fue tomar la sólida lógica de negocio enseñada en DS4B y rediseñar su arquitectura para que sea capaz de procesar volúmenes masivos de datos en entornos distribuidos en la nube (**Cloud Computing**), superando las limitaciones de hardware de una computadora personal.

---

## 2. El Reto: De un Enfoque Local a la Nube
La implementación original (orientada a Data Science local) utiliza librerías de un solo nodo como Pandas y estructuras imperativas. Si bien es el enfoque perfecto para aprender la matemática subyacente, en un entorno corporativo real presenta retos críticos de escalabilidad:

*   **Uso Intensivo de Memoria RAM:** El uso de funciones como `pd.get_dummies()` crea matrices dispersas inmensas (una columna por cada producto). Si el catálogo corporativo crece a miles de SKUs, la memoria del nodo maestro inevitablemente colapsa (**Out Of Memory Error**).
*   **Complejidad Computacional:** El cálculo de la confianza se realiza iterando pares de productos mediante `itertools.combinations` anidado en un bucle `for`. Esta evaluación secuencial, fila por fila, tiene una **complejidad de orden cuadrático \(O(N^2)\)**. Ante un histórico de millones de tickets de compra, el tiempo de ejecución crece exponencialmente, volviendo el modelo inviable para producción diaria.

---

## 3. Mi Aporte: Arquitectura Distribuida con PySpark
Para hacer este pipeline apto para producción en clústeres reales (como **Databricks** o **AWS EMR**), reescribí la solución completa utilizando la API de DataFrames de **Apache Spark**. 

### 🚀 Mejoras Arquitectónicas Implementadas:

1.  **Eliminación de la Matriz Dispersa:** En lugar de crear cientos de columnas temporales, Spark procesa los datos en su formato vertical transaccional nativo, ahorrando gigabytes de memoria en el procesamiento.
2.  **Self-Join Distribuido (Sin Bucles `for`):** Se sustituyó el bucle imperativo por un cruce relacional de la tabla consigo misma (`JOIN`). Al aplicar una **condición de orden lexicográfico (\(A < B\))**, Spark calcula todas las combinaciones válidas en paralelo, distribuyendo la carga en múltiples nodos (*Workers*) y eliminando duplicados automáticamente.
3.  **Poda Analítica (Inspiración en Apriori):** Por cuestiones de optimización frente al *Data Skew* y el *Shuffle Cartesiano* en el clúster, implementé un filtrado temprano. El pipeline calcula el soporte individual primero y descarta los productos estadísticamente irrelevantes antes de realizar el *Self-Join* masivo.
4.  **Computación Vectorial:** El Soporte, la Confianza y el *Lift* ya no se calculan deteniendo el script por cada par, sino de manera vectorial usando operaciones lógicas de columnas y funciones de agregación agrupadas, impactando a millones de registros en segundos.

---

## 4. Estructura del Repositorio

```text
├── data/
│   └── .gitkeep                     # Carpeta destinada a alojar los outputs generados (ej. CSV de reglas)
├── notebooks/
│   ├── 01_TPE_MarketBasket_Pandas.ipynb  # Solución original de DS4B en Pandas (Referencia matemática)
│   └── 02_MarketBasket_PySpark.ipynb     # Solución refactorizada paso a paso para visualización distribuida
└── src/
    └── market_basket_pyspark_optimized.py # Código limpio en script puro, listo para orquestación en producción
```

---

## 5. Agradecimientos
Un agradecimiento especial a mis profesores y al equipo de **DS4B** por plantear este caso de uso inmersivo (el dataset de *Sano y Fresco*) y por sentar las sólidas bases de negocio que hicieron posible que yo pudiera llevar a cabo esta práctica de optimización y Data Engineering.

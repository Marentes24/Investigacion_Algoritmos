# Algoritmos y estructuras de búsqueda: análisis comparativo

## Resumen

Este trabajo estudia siete técnicas para resolver el problema de la búsqueda: búsqueda lineal, binaria, *Jump Search*, *Interpolation Search*, *Exponential Search*, árboles binarios de búsqueda balanceados y tablas hash. Se describe el funcionamiento de cada una, se analiza su complejidad temporal y espacial, y se propone una metodología experimental para compararlas. Finalmente se presentan criterios para elegir la técnica adecuada según las características de los datos.

**Palabras clave:** búsqueda, complejidad algorítmica, Big-O, estructuras de datos, tablas hash, árboles balanceados.

## Tabla de contenidos

1. [Introducción](#1-introducción)
2. [Marco teórico](#2-marco-teórico)
3. [Metodología](#3-metodología)
4. [Resultados](#4-resultados)
5. [Discusión](#5-discusión)
6. [Conclusiones](#6-conclusiones)
7. [Referencias](#7-referencias)

---

## 1. Introducción

### 1.1 Planteamiento del problema

Dado un conjunto de datos y un valor objetivo, el problema de la búsqueda consiste en encontrar la posición del valor o determinar que no existe. Es una de las operaciones más frecuentes en computación, por lo que su eficiencia impacta directamente en el rendimiento de aplicaciones, bases de datos y sistemas.

### 1.2 Objetivo general

Comparar siete técnicas de búsqueda en términos de funcionamiento, complejidad y escenarios de aplicación.

### 1.3 Objetivos específicos

- Describir el funcionamiento y el pseudocódigo de cada técnica.
- Analizar su complejidad temporal y espacial en el mejor caso, caso promedio y peor caso.
- Definir una metodología de pruebas de rendimiento reproducible.
- Establecer criterios de selección según el tipo de datos y las operaciones requeridas.

---

## 2. Marco teórico

### 2.1 Notación Big-O

La notación Big-O describe cómo crece el número de operaciones de un algoritmo a medida que aumenta el tamaño `n` de la entrada. Permite comparar algoritmos de forma independiente del hardware.

| Notación | Nombre | Ejemplo en este trabajo |
|---|---|---|
| O(1) | Constante | Hash Table (promedio) |
| O(log log n) | Doblemente logarítmica | Interpolation Search (promedio) |
| O(log n) | Logarítmica | Binaria, Exponential, BST balanceado |
| O(√n) | Raíz cuadrada | Jump Search |
| O(n) | Lineal | Búsqueda lineal |

### 2.2 Clasificación de las técnicas

| Grupo | Técnicas |
|---|---|
| Sobre datos sin orden | Búsqueda lineal |
| Sobre arreglos ordenados | Binaria, Jump, Interpolation, Exponential |
| Estructuras de datos especializadas | BST balanceado, Hash Table |

### 2.3 Búsqueda sobre datos sin orden

#### 2.3.1 Búsqueda lineal

Recorre los elementos uno por uno hasta encontrar el objetivo o llegar al final. No requiere orden y funciona con cualquier colección, incluidas las listas enlazadas. Es adecuada para listas pequeñas o para una única búsqueda.

```python
def busqueda_lineal(datos, objetivo):
    for i, valor in enumerate(datos):
        if valor == objetivo:
            return i
    return -1
```

- **Tiempo:** O(n) en promedio y peor caso; O(1) en el mejor caso.
- **Espacio:** O(1).

### 2.4 Búsqueda sobre arreglos ordenados

Las técnicas de esta sección requieren que los datos estén **ordenados** y que exista acceso aleatorio por índice.

#### 2.4.1 Búsqueda binaria

Compara el objetivo con el elemento central y descarta la mitad del rango en cada paso. Es muy rápida, pero exige ordenar previamente y es poco práctica en listas enlazadas.

```python
def busqueda_binaria(datos, objetivo):
    bajo, alto = 0, len(datos) - 1
    while bajo <= alto:
        medio = (bajo + alto) // 2
        if datos[medio] == objetivo:
            return medio
        if datos[medio] < objetivo:
            bajo = medio + 1
        else:
            alto = medio - 1
    return -1
```

- **Tiempo:** O(log n).
- **Espacio:** O(1) en la versión iterativa.

#### 2.4.2 Jump Search

Salta en bloques de tamaño √n hasta pasar el valor buscado y luego realiza una búsqueda lineal dentro del bloque anterior. Resulta útil cuando retroceder es costoso, y es un punto intermedio entre la lineal y la binaria.

```python
import math


def jump_search(datos, objetivo):
    n = len(datos)
    paso = int(math.sqrt(n))
    anterior = 0

    while anterior < n and datos[min(paso, n) - 1] < objetivo:
        anterior = paso
        paso += int(math.sqrt(n))
        if anterior >= n:
            return -1

    for i in range(anterior, min(paso, n)):
        if datos[i] == objetivo:
            return i
    return -1
```

- **Tiempo:** O(√n).
- **Espacio:** O(1).

#### 2.4.3 Interpolation Search

Variante de la binaria que estima la posición del objetivo según su valor, de forma análoga a buscar una palabra en un diccionario. La posición estimada es:

```
pos = bajo + ((objetivo - datos[bajo]) * (alto - bajo)) / (datos[alto] - datos[bajo])
```

Funciona muy bien con datos distribuidos uniformemente y degrada a O(n) si están muy sesgados.

```python
def interpolation_search(datos, objetivo):
    bajo, alto = 0, len(datos) - 1
    while bajo <= alto and datos[bajo] <= objetivo <= datos[alto]:
        if datos[alto] == datos[bajo]:
            return bajo if datos[bajo] == objetivo else -1
        pos = bajo + ((objetivo - datos[bajo]) * (alto - bajo)
                      // (datos[alto] - datos[bajo]))
        if datos[pos] == objetivo:
            return pos
        if datos[pos] < objetivo:
            bajo = pos + 1
        else:
            alto = pos - 1
    return -1
```

- **Tiempo:** O(log log n) en promedio con distribución uniforme; O(n) en el peor caso.
- **Espacio:** O(1).

#### 2.4.4 Exponential Search

Primero localiza el rango donde podría estar el objetivo duplicando el índice (1, 2, 4, 8, ...) y luego aplica búsqueda binaria en ese rango. Es eficiente cuando el elemento está cerca del inicio y sirve para listas de tamaño desconocido o muy grande.

```python
def busqueda_exponencial(datos, objetivo):
    n = len(datos)
    if n == 0:
        return -1
    if datos[0] == objetivo:
        return 0

    limite = 1
    while limite < n and datos[limite] <= objetivo:
        limite *= 2

    bajo, alto = limite // 2, min(limite, n - 1)
    while bajo <= alto:
        medio = (bajo + alto) // 2
        if datos[medio] == objetivo:
            return medio
        if datos[medio] < objetivo:
            bajo = medio + 1
        else:
            alto = medio - 1
    return -1
```

- **Tiempo:** O(log n).
- **Espacio:** O(1).

### 2.5 Estructuras de datos de búsqueda

#### 2.5.1 BST balanceado (AVL y Rojo-Negro)

Un árbol binario de búsqueda (BST) almacena los valores menores a la izquierda y los mayores a la derecha de cada nodo. Si se insertan datos ya ordenados, degenera en una lista y la búsqueda pasa a O(n). Los árboles balanceados se reequilibran mediante **rotaciones** tras cada inserción o eliminación para mantener la altura en O(log n).

| Variante | Criterio de balance | Característica |
|---|---|---|
| AVL | La diferencia de alturas entre subárboles es como máximo 1 | Más estricto: búsquedas más rápidas, más rotaciones al modificar |
| Rojo-Negro | Reglas de coloreado de nodos | Más relajado: inserciones y borrados más baratos |

Mantienen el orden, lo que permite recorrido in-order, mínimo y máximo, sucesor y predecesor, y búsquedas por rango. Se usan, por ejemplo, en `TreeMap` de Java y en índices de bases de datos.

- **Tiempo:** O(log n) para búsqueda, inserción y eliminación.
- **Espacio:** O(n).

#### 2.5.2 Hash Table

Utiliza una **función hash** para transformar cada clave en un índice de un arreglo, lo que permite un acceso casi directo. Conceptos clave:

- **Colisiones:** dos claves distintas que producen el mismo índice. Se resuelven mediante encadenamiento (una lista por casilla) o direccionamiento abierto (sondeo lineal, cuadrático o doble hash).
- **Factor de carga** (`n / tamaño`): cuanto mayor es, más colisiones se producen.
- **Rehashing:** redimensionar la tabla y reinsertar los elementos cuando el factor de carga supera un umbral.

No conserva el orden de los datos. En Python, `dict` y `set` están implementados con tablas hash.

- **Tiempo:** O(1) en promedio; O(n) en el peor caso.
- **Espacio:** O(n).

---

## 3. Metodología

### 3.1 Criterios de comparación

Cada técnica se evalúa según:

1. Complejidad temporal (mejor caso, promedio y peor caso).
2. Complejidad espacial.
3. Requisitos sobre los datos (orden, distribución, acceso aleatorio).
4. Soporte para datos dinámicos (inserciones y eliminaciones).
5. Operaciones adicionales (rangos, orden, mínimo y máximo).

### 3.2 Diseño experimental

- **Lenguaje:** Python 3.
- **Tamaños de entrada:** 10³, 10⁴, 10⁵ y 10⁶ elementos.
- **Tipos de datos:** enteros ordenados con distribución uniforme y con distribución sesgada (por ejemplo, exponencial).
- **Casos de prueba:** objetivo al inicio, al medio, al final y ausente.
- **Repeticiones:** varias ejecuciones por caso, reportando el promedio.
- **Medición:** `time.perf_counter()` o el módulo `timeit`.

### 3.3 Script de pruebas

El siguiente script asume que las funciones de la sección 2 están definidas en el mismo archivo o importadas. Para la tabla hash se usa `dict` de Python. El BST balanceado puede incluirse si se implementa un AVL propio.

```python
import random
import time


def medir(funcion, datos, objetivo, repeticiones=20):
    inicio = time.perf_counter()
    for _ in range(repeticiones):
        funcion(datos, objetivo)
    return (time.perf_counter() - inicio) / repeticiones


def comparar(n, uniforme=True):
    if uniforme:
        datos = list(range(0, n * 2, 2))
    else:
        datos = sorted(int(random.expovariate(1 / 1000) * 1000)
                       for _ in range(n))

    objetivos = {
        "inicio": datos[0],
        "medio": datos[n // 2],
        "final": datos[-1],
        "ausente": -1,
    }
    tabla_hash = {valor: i for i, valor in enumerate(datos)}
    algoritmos = {
        "lineal": busqueda_lineal,
        "binaria": busqueda_binaria,
        "jump": jump_search,
        "interpolation": interpolation_search,
        "exponential": busqueda_exponencial,
        "hash": lambda d, x: tabla_hash.get(x, -1),
    }

    for nombre, funcion in algoritmos.items():
        for caso, objetivo in objetivos.items():
            t = medir(funcion, datos, objetivo)
            print(f"n={n:>8} | {nombre:<14} | {caso:<8} | {t * 1e6:10.2f} µs")


if __name__ == "__main__":
    for n in (10**3, 10**4, 10**5, 10**6):
        comparar(n)
```

---

## 4. Resultados

### 4.1 Resultados teóricos

| Técnica | Mejor caso | Caso promedio | Peor caso | Espacio | ¿Requiere orden? |
|---|---|---|---|---|---|
| Lineal | O(1) | O(n) | O(n) | O(1) | No |
| Binaria | O(1) | O(log n) | O(log n) | O(1) | Sí |
| Jump Search | O(1) | O(√n) | O(√n) | O(1) | Sí |
| Interpolation | O(1) | O(log log n) | O(n) | O(1) | Sí (uniforme) |
| Exponential | O(1) | O(log n) | O(log n) | O(1) | Sí |
| BST balanceado | O(1) | O(log n) | O(log n) | O(n) | Se mantiene solo |
| Hash Table | O(1) | O(1) | O(n) | O(n) | No |

### 4.2 Resultados experimentales

> Esta sección se completa con las mediciones obtenidas al ejecutar el script de la sección 3.3.

| n | Lineal | Binaria | Jump | Interpolation | Exponential | Hash |
|---|---|---|---|---|---|---|
| 10³ | | | | | | |
| 10⁴ | | | | | | |
| 10⁵ | | | | | | |
| 10⁶ | | | | | | |

---

## 5. Discusión

### 5.1 Guía de selección

| Situación | Técnica recomendada |
|---|---|
| Lista pequeña o desordenada, búsqueda única | Lineal |
| Arreglo ordenado y estático | Binaria |
| Retroceder en los datos es costoso | Jump Search |
| Datos numéricos ordenados y uniformes | Interpolation |
| Tamaño desconocido o elemento cercano al inicio | Exponential |
| Datos dinámicos que requieren orden o búsquedas por rango | BST balanceado |
| Acceso por clave sin importar el orden | Hash Table |

### 5.2 Compromisos observados

- **Velocidad frente a flexibilidad:** la tabla hash ofrece el acceso promedio más rápido, pero no permite recorrer los datos en orden ni hacer búsquedas por rango, algo que sí logra el BST balanceado.
- **Costo de ordenar:** las técnicas sobre arreglos ordenados solo compensan si los datos ya están ordenados o se realizan muchas búsquedas tras una única ordenación.
- **Dependencia de la distribución:** Interpolation Search puede superar a la binaria con datos uniformes, pero es la más sensible a distribuciones sesgadas.
- **Datos dinámicos:** los arreglos ordenados penalizan las inserciones; el BST balanceado y la tabla hash las manejan de forma eficiente.

---

## 6. Conclusiones

- No existe un algoritmo de búsqueda universalmente mejor: la elección depende de si los datos están ordenados, de si cambian con frecuencia y de qué operaciones se necesitan.
- Para datos estáticos y ordenados, la búsqueda binaria y sus variantes son una opción sólida y de bajo consumo de memoria.
- Para datos dinámicos, los BST balanceados y las tablas hash son las alternativas habituales: los primeros conservan el orden y las segundas ofrecen el mejor tiempo promedio.
- Las pruebas de rendimiento reales permiten validar el análisis teórico, especialmente en técnicas sensibles a la distribución de los datos como Interpolation Search.

---

## 7. Referencias

- Cormen, T., Leiserson, C., Rivest, R., Stein, C. *Introduction to Algorithms* (CLRS).
- Sedgewick, R., Wayne, K. *Algorithms*.
- [GeeksforGeeks: Searching Algorithms](https://www.geeksforgeeks.org/searching-algorithms/)
- [VisuAlgo: visualización de algoritmos y estructuras de datos](https://visualgo.net/)

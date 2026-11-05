# Algoritmos y estructuras de búsqueda

Investigación sobre siete técnicas para resolver el problema de la **búsqueda**: dado un conjunto de datos y un valor objetivo, encontrar su posición (o determinar que no existe).

## Índice

1. [Introducción](#introducción)
2. [Búsqueda lineal](#1-búsqueda-lineal)
3. [Búsqueda binaria](#2-búsqueda-binaria)
4. [Jump Search](#3-jump-search)
5. [Interpolation Search](#4-interpolation-search)
6. [Exponential Search](#5-exponential-search)
7. [BST balanceado](#6-bst-balanceado-avl-y-rojo-negro)
8. [Hash Table](#7-hash-table)
9. [Comparativa](#comparativa-de-complejidad)
10. [¿Cuándo usar cada uno?](#cuándo-usar-cada-uno)
11. [Conclusiones](#conclusiones)
12. [Referencias](#referencias)

---

## Introducción

Buscar es una de las operaciones más frecuentes en computación. La eficiencia se mide con la **notación Big-O**, que describe cómo crece el número de operaciones al aumentar el tamaño `n` de los datos.

Los algoritmos se pueden agrupar así:

| Grupo | Algoritmos |
|---|---|
| Sobre datos **sin orden** | Búsqueda lineal |
| Sobre arreglos **ordenados** | Binaria, Jump, Interpolation, Exponential |
| **Estructuras** de datos especializadas | BST balanceado, Hash Table |

---

## 1. Búsqueda lineal

**Idea:** recorrer los elementos uno por uno hasta encontrar el objetivo o llegar al final.

- **Requisitos:** ninguno (no necesita orden).
- **Ventajas:** simplicidad, funciona con cualquier colección, incluso listas enlazadas.
- **Desventajas:** lenta con grandes volúmenes de datos.
- **Casos de uso:** listas pequeñas, datos desordenados, una única búsqueda.

```python
def busqueda_lineal(datos, objetivo):
    for i, valor in enumerate(datos):
        if valor == objetivo:
            return i
    return -1
```

**Complejidad:** tiempo O(n), espacio O(1).

---

## 2. Búsqueda binaria

**Idea:** comparar con el elemento central y descartar la mitad del rango en cada paso.

- **Requisitos:** datos **ordenados** y acceso aleatorio (arreglos).
- **Ventajas:** muy rápida, con crecimiento logarítmico.
- **Desventajas:** exige ordenar primero; poco práctica en listas enlazadas.
- **Casos de uso:** búsquedas repetidas sobre datos estáticos.

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

**Complejidad:** tiempo O(log n), espacio O(1) en la versión iterativa.

---

## 3. Jump Search

**Idea:** saltar en bloques de tamaño √n hasta pasar el valor buscado, y luego hacer una búsqueda lineal dentro del bloque anterior.

- **Requisitos:** datos ordenados.
- **Ventajas:** solo retrocede dentro de un bloque; útil cuando moverse hacia atrás es costoso.
- **Desventajas:** más lenta que la binaria.
- **Casos de uso:** medios donde el retroceso es caro; punto intermedio entre lineal y binaria.

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

**Complejidad:** tiempo O(√n), espacio O(1).

---

## 4. Interpolation Search

**Idea:** variante de la binaria que **estima la posición** según el valor buscado, como cuando se abre un diccionario cerca del final para buscar una palabra con "Z".

Fórmula de la posición estimada:

```
pos = bajo + ((objetivo - datos[bajo]) * (alto - bajo)) / (datos[alto] - datos[bajo])
```

- **Requisitos:** datos ordenados y, idealmente, **distribuidos uniformemente**.
- **Ventajas:** muy rápida con distribución uniforme.
- **Desventajas:** degrada a O(n) con datos muy sesgados.
- **Casos de uso:** tablas numéricas grandes y uniformes (por ejemplo, IDs secuenciales).

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

**Complejidad:** promedio O(log log n), peor caso O(n), espacio O(1).

---

## 5. Exponential Search

**Idea:** encontrar primero el rango donde podría estar el valor, duplicando el índice (1, 2, 4, 8, ...), y luego aplicar búsqueda binaria en ese rango.

- **Requisitos:** datos ordenados.
- **Ventajas:** eficiente cuando el elemento está cerca del inicio; sirve para listas muy grandes o de **tamaño desconocido o infinito**.
- **Desventajas:** más compleja que la binaria simple.
- **Casos de uso:** flujos de datos ordenados sin tamaño conocido.

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

**Complejidad:** tiempo O(log n), espacio O(1).

---

## 6. BST balanceado (AVL y Rojo-Negro)

**Idea:** un **árbol binario de búsqueda** (BST) guarda los menores a la izquierda y los mayores a la derecha de cada nodo. Si se inserta en orden, degenera en una lista (O(n)). Los árboles **balanceados** se reequilibran con **rotaciones** tras cada inserción o eliminación para mantener la altura en O(log n).

| Variante | Criterio de balance | Característica |
|---|---|---|
| **AVL** | La diferencia de alturas entre subárboles es como máximo 1 | Más estricto: búsquedas más rápidas, más rotaciones al modificar |
| **Rojo-Negro** | Reglas de coloreado de nodos | Más relajado: inserciones y borrados más baratos |

- **Ventajas:** mantiene el orden; permite recorrido in-order, mínimo/máximo, sucesor/predecesor y búsquedas por rango.
- **Desventajas:** implementación compleja; usa más memoria por nodo.
- **Casos de uso:** datos dinámicos que requieren orden (índices de bases de datos, `TreeMap` en Java, `std::map` en C++).

**Complejidad:** búsqueda, inserción y eliminación en O(log n); espacio O(n).

---

## 7. Hash Table

**Idea:** una **función hash** transforma la clave en un índice de un arreglo, permitiendo acceso casi directo.

Temas clave a investigar:

- **Función hash:** debe ser rápida y distribuir las claves de forma uniforme.
- **Colisiones:** dos claves distintas con el mismo índice. Se resuelven con:
  - **Encadenamiento** (listas en cada casilla).
  - **Direccionamiento abierto** (sondeo lineal, cuadrático, doble hash).
- **Factor de carga** (`n / tamaño`): cuando es alto, aumentan las colisiones.
- **Rehashing:** redimensionar la tabla y reinsertar los elementos.

**Ventajas:** búsqueda, inserción y borrado en O(1) promedio.
**Desventajas:** no mantiene orden; peor caso O(n) con muchas colisiones; consume memoria extra.
**Casos de uso:** diccionarios, cachés, conjuntos, conteo de frecuencias.

```python
# En Python, dict y set están implementados con tablas hash
inventario = {"manzana": 10, "pera": 5}
print("manzana" in inventario)  # O(1) promedio
```

**Complejidad:** promedio O(1), peor caso O(n), espacio O(n).

---

## Comparativa de complejidad

| Algoritmo | Mejor caso | Caso promedio | Peor caso | Espacio | ¿Requiere orden? |
|---|---|---|---|---|---|
| Lineal | O(1) | O(n) | O(n) | O(1) | No |
| Binaria | O(1) | O(log n) | O(log n) | O(1) | Sí |
| Jump Search | O(1) | O(√n) | O(√n) | O(1) | Sí |
| Interpolation | O(1) | O(log log n) | O(n) | O(1) | Sí (uniforme) |
| Exponential | O(1) | O(log n) | O(log n) | O(1) | Sí |
| BST balanceado | O(1) | O(log n) | O(log n) | O(n) | Se mantiene solo |
| Hash Table | O(1) | O(1) | O(n) | O(n) | No |

---

## ¿Cuándo usar cada uno?

| Situación | Recomendación |
|---|---|
| Lista pequeña o desordenada, búsqueda única | Lineal |
| Arreglo ordenado y estático | Binaria |
| Retroceder es costoso | Jump Search |
| Datos numéricos ordenados y uniformes | Interpolation |
| Tamaño desconocido o elemento cerca del inicio | Exponential |
| Datos que cambian y necesitan orden o rangos | BST balanceado |
| Acceso por clave sin importar el orden | Hash Table |

---

## Conclusiones

- No existe un algoritmo universalmente mejor: la elección depende de si los datos están **ordenados**, si **cambian con frecuencia** y de qué operaciones se necesitan.
- Las búsquedas sobre arreglos ordenados (binaria y derivadas) son excelentes para datos **estáticos**.
- Para datos **dinámicos**, los BST balanceados y las tablas hash son las opciones habituales; los primeros conservan el orden y las segundas ofrecen el acceso promedio más rápido.
- Medir con pruebas de rendimiento reales ayuda a confirmar la teoría, especialmente para Interpolation Search, cuyo rendimiento depende de la distribución de los datos.

---

## Referencias

- Cormen, T., Leiserson, C., Rivest, R., Stein, C. *Introduction to Algorithms* (CLRS).
- Sedgewick, R., Wayne, K. *Algorithms*.
- [GeeksforGeeks: Searching Algorithms](https://www.geeksforgeeks.org/searching-algorithms/)
- [VisuAlgo: visualización de algoritmos y estructuras de datos](https://visualgo.net/)

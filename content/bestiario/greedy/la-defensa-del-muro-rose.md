---
title: La defensa del Muro Rose
---

# La defensa del Muro Rose

## Contexto

El cuerpo de Ingenieros de la Legión de Reconocimiento fue asignado a proteger el **Muro Rose** ante un inminente ataque de titanes.

A lo largo del muro se instalaron cañones de defensa. Cada cañón tiene un alcance limitado, que le permite cubrir una porción específica del muro. Sin embargo, activar un cañón consume una gran cantidad de pólvora y recursos, por lo que solo deben encenderse los estrictamente necesarios para proteger toda la sección crítica del muro.

La tarea consiste en diseñar un algoritmo de defensa **ávido** que determine qué cañones deben activarse para que toda la muralla quede bajo protección, minimizando la cantidad de cañones encendidos.

## Enunciado

El tramo de muro a defender está representado por el intervalo:

$$[L, R]$$

Cada cañón posee:

- una posición $x_i$, que indica su ubicación sobre el muro, medida en metros desde el punto $0$;
- un alcance $r_i$, medido en metros.

Por lo tanto, el cañón $i$ cubre el segmento:

$$[x_i - r_i, x_i + r_i]$$

Además, se asegura que:

- es posible cubrir todo el tramo $[L, R]$ con los cañones disponibles;
- todos los valores son positivos y reales.

## Objetivo

Activar la menor cantidad posible de cañones de forma que cada punto del muro dentro del intervalo $[L, R]$ esté protegido por al menos un cañón.

La salida debe indicar:

1. el número mínimo de cañones activados;
2. la lista de cañones seleccionados, ya sea por índice o por posición.

## Ejemplo

Tramo a defender:

$$[L, R] = [0, 20]$$

Los ingenieros disponen de los siguientes cañones:

| Cañón | Posición $(x_i)$ | Alcance $(r_i)$ | Segmento cubierto |
|---|---:|---:|---|
| C1 | 2 | 3 | $[-1, 5]$ |
| C2 | 7 | 5 | $[2, 12]$ |
| C3 | 14 | 4 | $[10, 18]$ |
| C4 | 17 | 6 | $[11, 23]$ |
| C5 | 20 | 2 | $[18, 22]$ |
| C6 | 5 | 1 | $[4, 6]$ |

Salida esperada:

```text
Cañones seleccionados: C1, C2 y C4
Número mínimo: 3 cañones activados
```

<details>
<summary>Idea de resolución</summary>

El problema puede verse como una variante de **cobertura de intervalos**: se necesita cubrir completamente el intervalo $[L, R]$ usando la menor cantidad posible de segmentos.

La estrategia ávida consiste en mantener el punto más a la izquierda que todavía falta cubrir. En cada paso, se elige, entre todos los cañones cuyo inicio de cobertura sea menor o igual a ese punto, aquel que llegue más lejos hacia la derecha.

Es decir:

1. convertir cada cañón en un intervalo $[x_i - r_i, x_i + r_i]$;
2. ordenar los intervalos por su extremo izquierdo;
3. comenzar desde $L$;
4. entre todos los intervalos que empiezan antes o en el punto actual, elegir el que tenga mayor extremo derecho;
5. avanzar el punto actual hasta ese extremo derecho;
6. repetir hasta cubrir $R$.

</details>

<details>
<summary>Pseudocódigo</summary>

```text
cubrirMuro(cañones, L, R):
    intervalos = []

    para cada cañón i:
        inicio = x_i - r_i
        fin = x_i + r_i
        agregar (inicio, fin, i) a intervalos

    ordenar intervalos por inicio ascendente

    seleccionados = []
    actual = L
    j = 0

    mientras actual < R:
        mejorFin = actual
        mejorCañon = null

        mientras j < cantidad(intervalos) y intervalos[j].inicio <= actual:
            si intervalos[j].fin > mejorFin:
                mejorFin = intervalos[j].fin
                mejorCañon = intervalos[j]
            j++

        si mejorCañon es null:
            error: no se puede cubrir el muro

        agregar mejorCañon a seleccionados
        actual = mejorFin

    devolver seleccionados
```

</details>

<details>
<summary>Justificación de optimalidad</summary>

En cada paso, el algoritmo considera todos los cañones que pueden cubrir el primer punto del muro que aún no está protegido. Cualquier solución válida necesariamente debe elegir alguno de esos cañones, porque si no, ese punto quedaría descubierto.

Entre esas opciones, elegir el cañón que llega más lejos nunca perjudica la solución: cubre al menos tanto como cualquier otra alternativa posible para ese mismo punto. Por lo tanto, deja un tramo restante menor o igual para cubrir en los siguientes pasos.

Este argumento permite reemplazar la primera elección de una solución óptima por la elección ávida sin aumentar la cantidad total de cañones. Repitiendo el razonamiento en cada iteración, se obtiene una solución óptima.

</details>

<details>
<summary>Complejidad</summary>

Sea $n$ la cantidad de cañones disponibles.

- Convertir cañones en intervalos cuesta $O(n)$.
- Ordenar los intervalos cuesta $O(n \log n)$.
- Recorrer los intervalos cuesta $O(n)$, porque cada intervalo se analiza a lo sumo una vez.

Por lo tanto, la complejidad temporal total es:

$$O(n \log n)$$

La complejidad espacial es:

$$O(n)$$

por almacenar los intervalos y la solución seleccionada.

</details>

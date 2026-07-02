---
title: 'LeetCode #0420 - Strong Password Checker - Greedy'
tags: ['b/leetcode']
---

## Técnicas utilizadas
* **Greedy con análisis de casos por longitud:** la técnica consiste en tomar decisiones óptimas locales en cada paso basándose únicamente en el tamaño actual de la cadena. En lugar de realizar una búsqueda exhaustiva, el algoritmo aplica un conjunto de reglas fijas y prioridades matemáticas que garantizan alcanzar el estado de "contraseña fuerte" con el menor gasto posible de operaciones básicas. 

## Idea de la solución
La solución clasifica la contraseña en tres casos excluyentes según su tamaño ($n$), aplicando la acción local más eficiente para cada escenario:

1. **Contraseñas cortas ($n < 6$):** Se soluciona usando solo inserciones. El resultado es el valor máximo entre los caracteres que faltan para llegar a 6 y los tipos de caracteres obligatorios (mayúscula, minúscula o número) ausentes.
2. **Contraseñas medianas ($6 \le n \le 20$):** Se soluciona usando solo reemplazos. Se calculan las sustituciones necesarias para romper las rachas ($\lfloor L / 3 \rfloor$). El total de pasos es el máximo entre la suma de estos reemplazos y los tipos faltantes.
3. **Contraseñas largas ($n > 20$):** Se calculan los borrados obligatorios para bajar el tamaño a 20. Para ahorrar pasos, se eliminan caracteres dando prioridad a las rachas de longitud múltiplo de 3, luego las de resto 1 y al final resto 2. Al llegar a 20, se suman los reemplazos que queden pendientes.


## Código

```pseudocode
FUNCION strongPasswordCheckerGreedy(password)
    n ← tamaño(password)

    // Calcular tipos de caracteres faltantes
    tieneMinus ← 0, tieneMayus ← 0, tieneDigito ← 0

    PARA i desde 0 HASTA n-1 HACER
        c ← password(i)
        SI c es minúscula ENTONCES
            tieneMinus ← 1
        SI c es mayúscula ENTONCES
            tieneMayus ← 1
        SI c es un dígito ENTONCES
            tieneDigito ← 1;
    FIN PARA

    tiposFaltantes ← 3 - (tieneMinus + tieneMayus + tieneDigito)

    // Caso 1: Contraseña corta
    SI n < 6 ENTONCES
        RETORNAR max(6 - n, tiposFaltantes)
    FIN SI

    // Contar bloques repetidos y clasificarlos por residuos de división por 3
    pasosReemplazo ← 0
    necesitaUnBorrado ← 0
    necesitaDosBorrados ← 0

    i ← 0
    MIENTRAS i < n HACER
        tamBloque ← 1 // tamaño de bloque de repetidos

        MIENTRAS i+1 < n Y password(i) == password(i+1) HACER
            tamBloque ← tamBloque + 1
            i ← i + 1
        FIN MIENTRAS

        SI tamBloque >= 3 ENTONCES
            pasosReemplazo ← pasosReemplazo + ParteEntera(tamBloque / 3)

            SI tamBloque MOD 3 == 0 ENTONCES
                necesitaUnBorrado ← necesitaUnBorrado + 1
            SINO SI tamBloque MOD 3 == 1 ENTONCES
                necesitaDosBorrados ← necesitaDosBorrados + 1
            FIN SI
        FIN SI
        i ← i + 1
    FIN MIENTRAS

    // Caso 2: Contraseña mediana
    SI n <= 20 ENTONCES
        RETORNAR max(pasosReemplazo, tiposFaltantes)
    FIN SI

    // Caso 3: Contraseña larga (n > 20)
    tamSobrante ← n - 20
    totalBorrados ← tamSobrante // Guardamos el costo fijo de las eliminaciones requeridas

    // Prioridad 1: Grupos 3k (1 borrado = 1 reemplazo menos)
    SI tamSobrante > 0 Y necesitaUnBorrado > 0 ENTONCES
        numBorrados ← min(tamSobrante, necesitaUnBorrado)
        tamSobrante ← tamSobrante - numBorrados
        pasosReemplazo ← pasosReemplazo - numBorrados
    FIN SI

    // Prioridad 2: Grupos 3k+1 (2 borrados = 1 reemplazo menos)
    SI tamSobrante > 0 Y necesitaDosBorrados > 0 ENTONCES
        numBorrados ← min(tamSobrante, necesitaDosBorrados * 2)
        tamSobrante ← tamSobrante - numBorrados
        pasosReemplazo ← pasosReemplazo - ParteEntera(numBorrados / 2)
    FIN SI

    // Prioridad 3: Cualquier remanente (3 borrados = 1 reemplazo menos)
    SI tamSobrante > 0 Y pasosReemplazo > 0 ENTONCES
        numBorrados ← min(tamSobrante, pasosReemplazo * 3)
        tamSobrante ← tamSobrante - numBorrados
        pasosReemplazo ← pasosReemplazo - ParteEntera(numBorrados / 3)
    FIN SI

    // Resultado final: eliminaciones obligatorias + lo que sea mayor entre reemplazos e insuficiencia de tipos

    RETORNAR totalBorrados + max(pasosReemplazo, tiposFaltantes)
FIN FUNCION

```

## Traza de ejemplo

Usaremos de ejemplo el caso 2.<br>

**Contraseña analizada:** `"aaaaaabcccc"`

**Longitud:** 11

Primero el programa recorre los caracteres  para chequear los tipos:
- Contiene minúsculas ('a', 'b', 'c') -> tieneMinus = 1
- Contiene mayúsculas -> tieneMayus = 0
- Contiene dígitos -> tieneDigito = 0

**Cálculo de tipos faltantes:**
tiposFaltantes = 3 - (1 + 0 + 0) = 2

Como tamaño es 11, NO entra al Caso 1 (tamaño < 6).

**Análisis de las rachas o caracteres repetidos:**

| i | Carácter | ¿Igual al sig.? | Long. racha | Acción |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 'a' | SI | 2 | Avanzo |
| 1 | 'a' | SI | 3 | Avanzo |
| 2 | 'a' | SI | 4 | Avanzo |
| 3 | 'a' | SI | 5 | Avanzo |
| 4 | 'a' | SI | 6 | Avanzo |
| 5 | 'a' | NO | 6 | Fin grupo: `pasosReemplazo` += 2, `necesitaUnBorrado` = 1 |
| 6 | 'b' | NO | 1 | Fin grupo: sin acción |
| 7 | 'c' | SI | 2 | Avanzo |
| 8 | 'c' | SI | 3 | Avanzo |
| 9 | 'c' | SI | 4 | Avanzo |
| 10 | 'c' | NO | 4 | Fin grupo: `pasosReemplazo` += 1, `necesitaDosBorrados` = 1 |

* **Cálculos finales:** `pasosReemplazo` = 3, `tiposFaltantes` = 2.
* **Aplicación Caso 2:** Como el tamaño (11) es $\leq 20$, calculamos $max(pasosReemplazo, tiposFaltantes) \rightarrow max(3, 2) = 3$.
* **Resultado:** 3 operaciones.

## Complejidad

### Temporal:
En el algoritmo destacan dos bucles.<br>
El primero para el conteo de tipos de caracteres por toda la cadena, lo que nos da una complejidad $T(N)$ para el peor caso, siendo N la cantidad de caracteres en “password”.<br>
El segundo para el conteo de bloques repetidos, usa bucles anidados: el bucle externo ($A$) inicia en i = 0, y el bucle interno ($B$) inicia en i + 1.
* En el caso de que “password” tenga todos sus caracteres diferentes, cuando el bucle $B$ vea si el caracter en la posición i es igual al de i+1, nunca avanzará. Entonces tenemos $N$ pasos para el bucle $A$ y 0 para el $B$. $\mathcal{O}(N+0)$
* Para el caso en que “password” tenga todos sus caracteres iguales, el bucle $B$ iniciará en i+1 y avanzará hasta i = $N-1$, cuando el bucle $A$ avance en 1 se cortará. Esto nos da 1 paso para el bucle $A$ y N-1 pasos para el $B$. $\mathcal{O}(1+N-1)$

En cualquier combinación intermedia, la suma de las vueltas que da el bucle externo más las que da el bucle interno siempre va a ser igual a N. Por eso la complejidad temporal será exactamente igual a $\mathcal{O}(N)$.

### Espacial:
Debido a que el algoritmo no utiliza ninguna estructura de datos para almacenamiento, sino únicamente variables primitivas para asignaciones y como contadores, la complejidad espacial es constante: $\mathcal{O}(1)$.


## Cuándo usar esta técnica

### Favorable cuando
* Se busca una solución directa, determinista y de alto rendimiento que no requiera explorar todas las combinaciones posibles de caracteres en un árbol de decisiones.
* Es ideal cuando las restricciones del problema permiten definir reglas de decisión fijas basadas en el tamaño de la entrada.

### Limitaciones
* La estrategia Greedy funciona exclusivamente porque las operaciones básicas (insertar, borrar, reemplazar) tienen todas el mismo costo unitario ($1$). Si cada acción tuviera una ponderación de costo diferente o condicionada, las reglas fijas de prioridad matemática fallarían y se requeriría obligatoriamente Programación Dinámica.

### Comparación con la solución de Programación Dinámica

La versión Greedy es mucho más eficiente en términos de complejidad temporal y espacial, presentando una implementación más concisa y legible. Sin embargo, la versión de [programación dinámica](420_strong_password_checker-programacion-dinamica.md) es más flexible ante cambios en las reglas o costos del problema.

## Referencias
N/A

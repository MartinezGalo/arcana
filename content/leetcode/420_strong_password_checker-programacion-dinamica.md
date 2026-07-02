---
title: 'LeetCode #0420 - Strong Password Checker - Programación Dinámica'
tags: ['b/leetcode']
---

## Técnicas utilizadas
**Programación Dinámica (Top-Down con Memorización):** Modela el problema mediante subproblemas recursivos. Utiliza un diccionario de memoria ([Map](../grimorio/data-structures/map.md)) para almacenar las soluciones de estados combinados ya calculados, transformando un árbol de decisión exponencial en uno polinomial.



## Idea de la solución

Se simplifica el análisis de la contraseña reduciéndola a un conjunto de componentes aislados: cantidad de tipos faltantes y una lista con las longitudes de las `rachas` de caracteres repetidos ($\ge 3$), abstrayéndose de los caracteres específicos.

**Longitud de la contraseña ($n$) y Operaciones Obligatorias:**
   * Si $n < 6$, existe un déficit de caracteres. Las `inserciones obligatorias` se calculan como $6 - n$.
   * Si $n > 20$, existe un exceso de caracteres. Los `borrados obligatorios` se calculan como $n - 20$.
   * Si $6 \le n \le 20$, las operaciones obligatorias por tamaño son $0$.

**Longitud de una Racha ($L$) y la cantidad de `Reemplazos necesarios`:**
   * Una racha de caracteres idénticos de longitud $L$ requiere inicialmente de $\lfloor L / 3 \rfloor$ reemplazos para ser completamente eliminada. Por ejemplo, `"aaa"` ($L=3$) requiere $3 // 3 = 1$ reemplazo (`"aBa"`), mientras que `"aaaaaaa"` ($L=7$) requiere $7 // 3 = 2$ reemplazos (`"aaBaaCa"`).

**El Impacto de los Borrados sobre las Rachas y Reemplazos:**
   * Borrar caracteres dentro de una racha reduce su longitud, lo que a su vez puede disminuir los reemplazos necesarios. Sin embargo, el impacto no es lineal y depende del residuo matemático de la racha ($L \pmod 3$):
     * Si $L \pmod 3 == 0$ (ej. $L=3$), borrar **1** carácter reduce la racha a tamaño 2, eliminando la necesidad de reemplazos inmediatamente (ahorro de 1 reemplazo por 1 borrado).
     * Si $L \pmod 3 == 1$ (ej. $L=4$), se necesitan borrar **2** caracteres para reducir los reemplazos.
     * Si $L \pmod 3 == 2$ (ej. $L=5$), se necesitan borrar **3** caracteres para reducir los reemplazos.
   * La Programación Dinámica se encarga de probar todas las distribuciones posibles de los `borrados obligatorios` sobre las distintas rachas para encontrar **la combinación que cause la mayor reducción de reemplazos pendientes**.

**La Ecuación Maestra Unificada:**
   * Las operaciones finales se componen de los `borrados_obligatorios` más el valor máximo entre: la **cantidad de reemplazos optimizada**, los **tipos de caracteres faltantes** y las **inserciones obligatorias**. Esto se debe a que una **inserción** o un **reemplazo** estratégicos pueden solucionar simultáneamente un problema de racha y aportar un tipo de carácter faltante (por ejemplo, insertar o reemplazar por una mayúscula en medio de `"aaa"` -> `"aaB"`).

$$
 modificaciones\ minimas = borrados\_obligatorios + \max(reemplazos, tipos\_faltantes, inserciones\_obligatorias)
$$


## Código

```python
def strong_password_checker(password: str) -> int:
    n = len(password)

    # Obtiene la cantidad de tipos de caracteres faltantes (mayúscula, minúscula, dígito)
    tipos_faltantes = caracteres_faltantes(password)
    # Obtiene una lista con la longitud de las rachas de caracteres repetidos
    rachas = obtener_longitud_rachas(password, n)

    # Si la contraseña es demasiado larga, necesitamos borrar caracteres.
    borrados_obligatorios = max(0, n - 20)

    # Si la contraseña es demasiado corta, necesitamos insertar caracteres.
    inserciones_obligatorias = max(0, 6 - n)
    
    # Mapa para memoización de resultados
    memo = {}

    # Optimiza la cantidad de reemplazos necesarios considerando los borrados disponibles
    reemplazos_restantes = optimizar_rachas(0, borrados_obligatorios, rachas, memo)
    
    # El resultado final es la suma de los borrados obligatorios, los reemplazos restantes 
    # y los caracteres faltantes o inserciones necesarias.
    return borrados_obligatorios + max(reemplazos_restantes, tipos_faltantes, inserciones_obligatorias)

def caracteres_faltantes(password: str) -> int:
    mayus, minus, digito = 0, 0, 0
    for c in password:
        if c.isupper():
            mayus = 1
        elif c.islower():
            minus = 1
        elif c.isdigit():
            digito = 1
    return 3 - (mayus + minus + digito)


def obtener_longitud_rachas(password: str, n: int) -> list:
    rachas = []
    i = 0
    while i < n:
        j = i
        while j < n and password[j] == password[i]:
            j += 1
        longitud_racha = j - i
        if longitud_racha >= 3:
            rachas.append(longitud_racha)
        i = j
    return rachas


# La funcion busca optimizar la cantidad de reemplazos necesarios considerando los borrados disponibles
def optimizar_rachas(idx: int, borrados_disponibles: int, rachas: list, memo: dict) -> int:
    # Caso base: si pasamos por todas las rachas, no se necesitan más reemplazos
    if idx == len(rachas):
        return 0
        
    key = f"{idx}-{borrados_disponibles}"
    # Si ya calculamos el resultado para esta clave, lo devolvemos desde la memoización
    if key in memo:
        return memo[key]
        
    racha_actual = rachas[idx]
    # Inicializamos el mínimo de reemplazos necesarios como infinito
    minimo_reemplazos = float('inf')
    
    # Los últimos 2 caracteres de una racha no hacen falta borrarlos para cortarla, por lo que
    # el máximo de borrados útiles es la longitud de la racha menos 2, si no es mayor que los borrados disponibles.
    max_borrados_utiles = min(borrados_disponibles, racha_actual - 2)
    
    # Ramificamos: no borramos nada, borramos 1, borramos 2, ..., hasta max_borrados_utiles
    for b in range(max_borrados_utiles + 1):

        nueva_longitud = racha_actual - b

        # Reemplazos necesarios para cortar la racha despues de borrar b caracteres. 
        # Si la nueva longitud es menor que 3, no se necesitan reemplazos.
        reemplazos_necesarios = 0 if nueva_longitud < 3 else nueva_longitud // 3

        # Calculamos el costo futuro llamando recursivamente a la función para la siguiente racha
        reemplazos_siguiente_racha = optimizar_rachas(idx + 1, borrados_disponibles - b, rachas, memo)

        # Actualizamos el mínimo de reemplazos necesarios considerando la opción actual
        minimo_reemplazos = min(minimo_reemplazos, reemplazos_necesarios + reemplazos_siguiente_racha)
        
    # Guardamos el resultado en la memoización para evitar cálculos repetidos
    memo[key] = minimo_reemplazos
    return minimo_reemplazos


```
## Traza de ejemplo

Instancia de prueba: `password = "aaaaaaaaabccccccddd123456789"` ($n = 28$)

`faltan_tipos = 1` (Tiene minúsculas y dígitos; falta mayúscula).

`rachas = [9, 6, 3]` (Índices de racha 0, 1 y 2 respectivamente, correspondientes a 'a', 'c' y 'd').

`borrados_obligatorios` = $= \max(0, 28 - 20) = \mathbf{8}$. 

`inserciones_obligatorias = 0`.

Llamada inicial: `optimizar_rachas(idx=0, borrados_disponibles=8, rachas, memo)`

| Estado (idx-borrados_disponibles) | Acción / Resultado | ¿Calculado o Memorizado? |
|-----------------------------------|--------------------|--------------------------|
| 0-8 | Evalúa racha 0 ("aaaaaaaaa", $L = 9$).<br>`max_borrados_utiles` = $\min(8, 9-2) = 7$.<br>Prueba $b=0$ (no borra).<br>`reemplazos_necesarios` $= 3$.<br>Llama a (1-8). | Calculando |
| 0-8 → 1-8 | Evalúa racha 1 ("cccccc", $L = 6$).<br>`max_borrados_utiles` = $\min(8, 6-2) = 4$.<br>Prueba $b=0$ (no borra).<br>`reemplazos_necesarios` $= 2$.<br>Llama a (2-8). | Calculando |
| 0-8 → 1-8 → 2-8 | Última racha ("ddd", $L = 3$).<br>`max_borrados_utiles` = $\min(8, 3-2) = 1$.<br> $b=0\rightarrow 1$ reemplazo.<br> $b=1\rightarrow 0$ reemplazos.<br>Guarda memo["2-8"] = 0.<br>Retorna 0. | Calculando y guarda en memo |
| 0-8 → 1-8 | Evalúa racha 1 ("cccccc", $L = 6$).<br>`minimo_reemplazos` local = $2 \text{ (local)} + 0 \text{ (futuro)} = 2$.<br>Avanza a $b=1$ ("ccc,cc").<br>`reemplazos_necesarios` $= 1$.<br>Llama a (2-7). | Calculando |
| 0-8 → 1-8 → 2-7 | Última racha ("ddd", $L = 3$).<br>`max_borrados_utiles` = $\min(7, 3-2) = 1$.<br> $b=0\rightarrow 1$ reemplazo.<br> $b=1\rightarrow 0$ reemplazos.<br>Guarda memo["2-7"] = 0.<br>Retorna 0. | Calculando y guarda en memo |
| 0-8 → 1-8 | Evalúa racha 1 ("cccccc", $L = 6$).<br>`minimo_reemplazos` local = $1 \text{ (local)} + 0 \text{ (futuro)} = 1$.<br>Avanza a $b=2$ ("ccc,c").<br>`reemplazos_necesarios` $= 1$.<br>Llama a (2-6). | Calculando |
| 0-8 → 1-8 → 2-6 | Última racha ("ddd", $L = 3$).<br>`max_borrados_utiles` = $\min(6, 3-2) = 1$.<br> $b=0\rightarrow 1$ reemplazo.<br> $b=1\rightarrow 0$ reemplazos.<br>Guarda memo["2-6"] = 0.<br>Retorna 0. | Calculando y guarda en memo |
| 0-8 → 1-8 | Evalúa racha 1 ("cccccc", $L = 6$).<br>Avanza a $b=3$ ("ccc,").<br>`reemplazos_necesarios` $= 1$.<br>Llama a (2-5). | Calculando |
| 0-8 → 1-8 → 2-5 | Última racha ("ddd", $L = 3$).<br>`max_borrados_utiles` = $\min(5, 3-2) = 1$.<br> $b=0\rightarrow 1$ reemplazo.<br> $b=1\rightarrow 0$ reemplazos.<br>Guarda memo["2-5"] = 0.<br>Retorna 0. | Calculando y guarda en memo |
| 0-8 → 1-8 | Evalúa racha 1 ("cccccc", $L = 6$).<br>Avanza a $b=4$ ("cc").<br>`reemplazos_necesarios` $= 0$.<br>Llama a (2-4). | Calculando |
| 0-8 → 1-8 → 2-4 | Última racha ("ddd", $L = 3$).<br>`max_borrados_utiles` = $\min(4, 3-2) = 1$.<br> $b=0\rightarrow 1$ reemplazo.<br> $b=1\rightarrow 0$ reemplazos.<br>Guarda memo["2-4"] = 0.<br>Retorna 0. | Calculando y guarda en memo |
| 0-8 → 1-8| Evalúa racha 1 ("cccccc", $L = 6$).<br>`minimo_reemplazos` local = $0 \text{ (local)} + 0 \text{ (futuro)} = 0$.<br>Guarda memo["1-8"]<br>Retorna `minimo_reemplazos` = 0. | Calculando |
| 0-8 | Evalúa racha 0 ("aaaaaaaaa", $L = 9$).<br>`minimo_reemplazos` local = $3 \text{ (local)} + 0 \text{ (futuro)} = 3$.<br>Avanza a $b=1$ ("aaa,aaa,aa").<br>`reemplazos_necesarios` $= 2$.<br>Llama a (1-7). | Calculando |
| 0-8 → 1-7 | Evalúa racha 1 ("cccccc", $L = 6$).<br>`max_borrados_utiles` = $\min(7, 6-2) = 4$.<br>Prueba $b=0$ (no borra).<br>`reemplazos_necesarios` $= 2$.<br>Llama a (2-7). | Calculando |
| 0-8 → 1-7 → 2-7 | Evita volver a procesar el árbol.<br>Devuelve memo["2-7"] que es 0 en $\mathcal{O}(1)$. | ¡MEMORIZADO! |
| 0-8 → 1-7 | Evalúa racha 1 ("cccccc", $L = 6$).<br>Avanza a $b=1$ ("ccc,cc").<br>`reemplazos_necesarios` $= 1$.<br>Llama a (2-6). | Calculando |
| 0-8 → 1-7 → 2-6 | Evita volver a procesar el árbol.<br>Devuelve memo["2-6"] que es 0 en $\mathcal{O}(1)$. | ¡MEMORIZADO! |
|...|...|...|


Tras evaluar todas las ramificaciones posibles de los bucles, el algoritmo determina que la distribución óptima para consumir los $8$ `borrados_obligatorios` consiste en aplicar 7 borrados en la racha 0 (dejándola en tamaño $2 \rightarrow 0$ reemplazos), 0 borrados en la racha 1 (dejándola en tamaño $6 \rightarrow 2$ reemplazos) y 1 borrado en la racha 2 (dejándola en tamaño $2 \rightarrow 0$ reemplazos). Logrando así un total de $2$ `reemplazos_restantes` tras optimizar los borrados.

Resultado final mediante la Ecuación Maestra:

$$
\text{modificaciones mínimas} = \text{borrados obligatorios} + \max(\text{reemplazos restantes}, \text{tipos faltantes}, \text{inserciones obligatorias})
$$

$$
8 + \max(2, 1, 0) = 8 + 2 = \mathbf{10}
$$



Como los $2$ reemplazos que quedaron pendientes tras optimizar los borrados son numéricamente mayores que el único tipo de carácter que nos faltaba ($2 > 1$), esos mismos $2$ reemplazos se eligen estratégicamente. Al momento de modificar los caracteres para romper la racha restante de las 'c', uno de esos cambios se realiza introduciendo la letra mayúscula requerida. De esta manera, se soluciona el problema de contenido y el de estructura en simultáneo, sin necesidad de agregar operaciones extras al total.

## Complejidad
### Temporal
$\mathcal{O}(R \times B \times L)$, donde $R$ es la cantidad de rachas, $B$ son los borrados obligatorios ($n - 20$) y $L$ es la longitud máxima de una racha. El espacio de estados del mapa está estrictamente acotado por $R \times B$. Para resolver cada estado de forma única, el bucle for realiza a lo sumo $L$ iteraciones.
### Espacial
$\mathcal{O}(R \times B) + \mathcal{O}(R)$. Requiere memoria para almacenar el mapa memo con las combinaciones únicas de estados. Adicionalmente, el stack de llamadas de la recursividad consume espacio proporcional a la cantidad de rachas ($R$).

## Cuándo usar esta técnica
### Favorable cuando
- El espacio de estados de las variables es acotado (idx y borrados tienen límites máximos pequeños debido a las restricciones físicas del problema), lo que evita que la estructura de memorización crezca descontroladamente.
- Se busca un código altamente mantenible y adaptativo, donde cambiar las reglas básicas (como el tamaño máximo de la contraseña o la longitud de las rachas toleradas) no requiera rediseñar la lógica matemática desde cero.
### Limitaciones
- No escala bien si las restricciones del problema cambian a longitudes extremadamente grandes (ej. $n \ge 10^5$), ya que la creación dinámica de cadenas de texto para las llaves del Map y el almacenamiento masivo de estados provocarían un consumo de memoria excesivo.
- Esta solución realiza trabajo redundante al explorar y evaluar ramificaciones de borrado ineficientes que el enfoque codicioso descarta de antemano mediante reglas de prioridad fijas.

### Comparación con la solución Greedy
El enfoque [Greedy](420_strong_password_checker-greedy.md) resuelve el escenario en $\mathcal{O}(n)$ de tiempo y $\mathcal{O}(1)$ de espacio utilizando una prioridad matemática rígida basada en el módulo de la longitud de las rachas ($L \pmod 3$).

La solución por Programación Dinámica es ligeramente más costosa en memoria, pero tiene la enorme ventaja de ser mucho más intuitiva de diseñar y deducir, ya que no requiere descubrir "el truco matemático oculto" para coordinar las prioridades de los borrados; la DP encuentra la combinación óptima de forma natural explorando inteligentemente el espacio de soluciones.

## Referencias
N/A
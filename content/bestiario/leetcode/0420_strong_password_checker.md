---
title: 'LeetCode #0420 - Strong Password Checker'
tags: ['b/leetcode']
---
## Nombre y enunciado

El problema nos pide calcular el número **mínimo** de operaciones (inserciones, eliminaciones o reemplazos) necesarias para transformar una cadena de texto en una "contraseña fuerte". Para que una contraseña se considere fuerte, debe cumplir tres reglas estrictas:
* Tener una longitud de entre 6 y 20 caracteres inclusive.
* Contener al menos una letra **minúscula**, una letra **mayúscula** y un **dígito**.
* No tener tres caracteres iguales consecutivos (por ejemplo, `"Baaab1"` es inválido por tener `"aaa"`).

[Problema original](https://leetcode.com/problems/strong-password-checker/)

---

## Intuición
Lo que hace a este problema tan interesante es que no se resuelve con un algoritmo estándar de libro, sino que la dificultad radica en que las decisiones dependen de la longitud de la cadena. 
* Si la contraseña es **corta** ($n < 6$), se realizan inserciones para alcanzar la longitud mínima.
* Si es del **tamaño requerido** ($6 \le n \le 20$), la longitud es ideal y el problema se resuelve puramente mediante reemplazos estratégicos para romper las rachas.
* Si es **larga** ($n > 20$), se deben realizar eliminaciones para reducir la longitud, y estas eliminaciones deben ser cuidadosamente priorizadas sobre las rachas de caracteres para minimizar los reemplazos necesarios después.

### Definición formal
* **Entrada:** Una única variable `password` de tipo `String` que consiste en letras, dígitos, puntos (`.`) y signos de exclamación (`!`).
* **Salida:** Un número entero (`int`) que representa la cantidad mínima de modificaciones necesarias para convertirla en una contraseña segura.
* **Restricciones:** $1 \le \text{longitud de la contraseña} \le 50$

---

## Ejemplos concretos
Para entender la lógica, separamos en tres casos según la longitud ($n$):

### Caso 1 (Contraseña Corta, $n < 6$)
**`password = "aaa1"`**
* **Longitud:** 4 (Faltan 2 para llegar a 6).
* **Tipos faltantes:** 1 (Tiene minúscula y dígito, falta mayúscula).
* **Rachas:** 
    * `"aaa"` de longitud 3 (3/3 = 1 reemplazo).

> **Resolución:** La cantidad mínima de modificaciones es **2**, ya que son los caracteres que faltan para llegar a 6. Uno de esos caracteres puede ser una mayúscula, que es el tipo de carácter faltante, y puede ser colocado estratégicamente dentro de la racha `"aaa"` para cortarla. Por ejemplo, insertando una `'B'` y una `'c'` queda `"aaBa1c"`. 
> 
> **Total de operaciones = 2.**

### Caso 2 (Longitud Correcta, $6 \le n \le 20$)
**`password = "aaaaaabcccc"`**
* **Longitud:** 11 (Tamaño ideal).
* **Tipos faltantes:** 2 (Mayúscula y dígito).
* **Rachas:** 
    * `"aaaaaa"` de longitud 6 (6/3 = 2 reemplazos).
    * `"cccc"` de longitud 4 (4/3 = 1 reemplazo).

> **Resolución:** Como la longitud es ideal, no queremos insertar ni borrar, solo se reemplazan caracteres. La racha `"aaaaaa"` requiere 2 reemplazos ($6 // 3 = 2$) y la racha `"cccc"` requiere 1 reemplazo ($4 // 3 = 1$), sumando un total de 3 reemplazos. Podemos aprovechar estos mismos reemplazos para introducir la mayúscula y el dígito que nos faltan, por ejemplo `"aaDaa1bccdc"`. 
> 
> **Total de operaciones = 3.**

### Caso 3 (Contraseña Larga, $n > 20$)
**`password = "aaaaaaaaabccccccddd123456789"`**
* **Longitud:** 28 (Sobran 8 caracteres).
* **Tipos faltantes:** 1 (mayúscula).
* **Rachas:** 
  * `"aaaaaaaaa"` de longitud 9 (9/3 = 3 reemplazos).
  * `"cccccc"` de longitud 6 (6/3 = 2 reemplazos).
  * `"ddd"` de longitud 3 (3/3 = 1 reemplazo).

> **Resolución:** Como la contraseña supera los 20 caracteres, estamos obligados a hacer **8 eliminaciones**. En lugar de eliminar al azar, usamos esos borrados obligatorios dentro de las rachas para reducir la necesidad de reemplazos. La distribución óptima consiste en eliminar **7 caracteres de la racha de 9** (dejándola en tamaño 2, lo que reduce sus reemplazos de 3 a 0), **0 caracteres de la racha de 6** (dejándola en tamaño 6, lo que mantiene 2 reemplazos) y **1 carácter de la racha de 3** (dejándola en tamaño 2, reduciendo sus reemplazos de 1 a 0).<br>Sumando los **8 borrados obligatorios** y los **2 reemplazos** que quedan pendientes para las rachas reducidas, con lo que a la vez agregamos el tipo faltante, se obtiene el total de operaciones.<br><br>
**Total de operaciones = $8 \text{ (borrados)} + \max(2 \text{ (reemplazos)}, 1 \text{ (tipo faltante)}, 0) = \mathbf{10}$.**

---

## Por dónde empezar

1. Una primera aproximación razonable es identificar las tres lógicas independientes según el tamaño de la contraseña ($n < 6$, $6 \le n \le 20$ y $n > 20$). Diseñar rutinas simples para calcular los caracteres faltantes o los reemplazos necesarios en base a las rachas de letras repetidas permite entender el comportamiento básico del problema.
2. El siguiente paso es estructurar el caso de contraseñas largas utilizando **Greedy**. Esto implica programar un orden de prioridad estricto para las eliminaciones basado en el resto de la división por 3 del tamaño de cada racha ($L \pmod 3$), asegurando que cada borrado reduzca al máximo los reemplazos posteriores.
3. Finalmente, una vez que la cadena se reduce al tamaño permitido de 20 caracteres mediante estas eliminaciones óptimas, el problema se unifica con el escenario mediano, sumando los reemplazos remanentes para obtener el mínimo absoluto de operaciones.

---

## Soluciones disponibles

- [[0420_strong_password_checker-greedy]]
- [[0420_strong_password_checker-programacion-dinamica]]
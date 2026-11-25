
# Bitácora de Requerimientos y Auditoría IA — Sesión 27
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Indexación de Arreglos en TAC  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353             
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de desplazamientos base + offset  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar el ejercicio manualmente, se me complicó organizar correctamente los temporales para calcular los desplazamientos de los arreglos. En el arreglo de una dimensión tuve que multiplicar el índice por el ancho del elemento y después utilizar ese resultado para acceder a la posición correspondiente. En el arreglo de dos dimensiones fue necesario respetar el orden de la operación i * filas + j antes de multiplicar por el ancho. También tuve que cuidar que cada temporal tuviera un número diferente y que las instrucciones se guardaran en las posiciones correctas del arreglo de cadenas. Otra dificultad fue considerar el límite de las líneas disponibles, ya que no se deben escribir más instrucciones de las que permite el arreglo.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC, utilizando el encabezado `api_sesion27.h` y compilación con `-Wall -Wextra -Werror`. La implementación debe ser modular y no debe incluir `main()`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores, con experiencia en generación de código intermedio TAC. |
| **Especificación (E)** | Implementar `emitir_acceso()` para arreglos de una dimensión y `emitir_acceso_2d()` para arreglos de dos dimensiones. En 1D se debe multiplicar el índice por el ancho del elemento y después generar la lectura indexada. En 2D se debe calcular primero i * filas + j, multiplicar el resultado por el ancho y generar la lectura. Cada temporal debe ser único y el parámetro `max` debe respetarse para evitar escribir más líneas de las disponibles. Si `base` es NULL, no se deben emitir instrucciones y se debe regresar 0. |
| **Output (O)** | Código C completo y modular, respetando exactamente las firmas del encabezado, sin `main()`, utilizando `snprintf()` y explicando la complejidad temporal y espacial. |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en compiladores y generación de código intermedio TAC.

CONTEXTO:
Estoy trabajando en la Sesión 27 de Lenguajes y Autómatas II. Necesito implementar dos funciones en lenguaje C99 para generar instrucciones TAC relacionadas con la indexación de arreglos de una y dos dimensiones.

ROL:
Compórtate como un programador experto en C y compiladores. Escribe código claro, modular, seguro y compatible con GCC.

ESPECIFICACIÓN:
Implementa exactamente las siguientes funciones declaradas en api_sesion27.h:

int emitir_acceso(char lineas[][64], int max, const char *base, const char *indice, int ancho, int *temporales);

int emitir_acceso_2d(char lineas[][64], int max, const char *base, const char *i, const char *j, int filas, int ancho, int *temporales);

Requisitos:
1. Para arreglos 1D, genera primero el desplazamiento multiplicando el índice por el ancho del elemento:
   _tK = indice * ancho
2. Después genera la lectura indexada:
   _tK+1 = base[_tK]
3. Para arreglos 2D, calcula primero:
   _tK = i * filas
4. Después aplana el índice:
   _tK+1 = _tK + j
5. Multiplica el índice lineal por el ancho:
   _tK+2 = _tK+1 * ancho
6. Finalmente genera la lectura:
   _tK+3 = base[_tK+2]
7. Respeta el orden i * filas + j. No intercambies las dimensiones.
8. Cada temporal nuevo debe incrementar el contador recibido mediante temporales.
9. Si base es NULL, no generes instrucciones y devuelve 0.
10. Respeta el número máximo de líneas indicado por max. Para 1D se necesitan 2 líneas y para 2D se necesitan 4.
11. Utiliza snprintf() para escribir en los búferes de 64 caracteres.
12. No utilices memoria dinámica ni variables globales.
13. No incluyas main(), porque el harness ya contiene esa función.
14. Conserva exactamente las firmas de las funciones.
15. El código debe compilar sin advertencias con:
    gcc -std=c11 -Wall -Wextra -Werror

OUTPUT:
Devuelve el código C completo para ia_sesion27_c.c, incluyendo los encabezados necesarios. Explica brevemente la complejidad temporal y espacial de las funciones y menciona cualquier caso borde que deba revisarse.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion27_c.c - Fase 2: El Cientifico
 *
 * Solución optimizada y robusta que cumple con el contrato api_sesion27.h.
 */
#include "api_sesion27.h"
#include <stdio.h>
#include <string.h>

int emitir_acceso(char lineas[][64], int max, const char *base, const char *indice, int ancho, int *temporales) {
    if (base == NULL || max < 2) {
        return 0;
    }

    int t_offset = (*temporales)++;
    int t_dest = (*temporales)++;

    snprintf(lineas[0], 64, "_t%d = %s * %d", t_offset, indice, ancho);
    snprintf(lineas[1], 64, "_t%d = %s[_t%d]", t_dest, base, t_offset);

    return 2;
}

int emitir_acceso_2d(char lineas[][64], int max, const char *base, const char *i, const char *j, int filas, int ancho, int *temporales) {
    if (base == NULL || max < 4) {
        return 0;
    }

    int t_mul = (*temporales)++;
    int t_flat = (*temporales)++;
    int t_offset = (*temporales)++;
    int t_dest = (*temporales)++;

    snprintf(lineas[0], 64, "_t%d = %s * %d", t_mul, i, filas);
    snprintf(lineas[1], 64, "_t%d = _t%d + %s", t_flat, t_mul, j);
    snprintf(lineas[2], 64, "_t%d = _t%d * %d", t_offset, t_flat, ancho);
    snprintf(lineas[3], 64, "_t%d = %s[_t%d]", t_dest, base, t_offset);

    return 4;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Arreglos multidimensionales en TAC.

**Entorno de compilación:** GCC 13.3.0 (Ubuntu 24.04).

### Salida del harness

**Prueba de la implementación manual (`artesano_sesion27_c.c`):**
```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion27.md
```

**Prueba de la implementación de IA (`ia_sesion27_c.c`):**
```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion27.md
```

* [x] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código de la IA, observé que valida si `base` es NULL y si el parámetro `max` permite generar las instrucciones necesarias. Sin embargo, no comprueba si otros punteros, como `lineas`, `temporales`, `indice`, `i` o `j`, son NULL. Si alguno de ellos no contiene una dirección válida, las operaciones con esos punteros podrían provocar un error. También noté que el harness no prueba esos casos, por lo que no puedo asegurar que estén resueltos solamente con el resultado de 6/6. La mejora sería agregar validaciones de esos parámetros antes de utilizarlos y realizar pruebas adicionales con punteros NULL y capacidades insuficientes. Así identifiqué una limitación que no aparece en las seis pruebas proporcionadas.

---

**Firma digital:** josue6martin

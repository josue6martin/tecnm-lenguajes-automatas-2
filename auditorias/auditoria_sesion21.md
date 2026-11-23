
# Bitácora de Requerimientos y Auditoría IA — Sesión 21
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Fundamentos de Código de 3 Direcciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Representación en Cuádruplos (op,a,b,r)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar el código manualmente, se me complicó un poco respetar el tamaño del buffer al generar los temporales y los cuádruplos. Utilicé `sprintf()` para escribir los datos, pero después identifiqué que esta función no limita la cantidad de caracteres que escribe, por lo que podría provocar un desbordamiento si el espacio disponible no es suficiente. También tuve que cuidar que los argumentos que llegan como NULL se representaran con un guion bajo y que el contador aumentara después de generar cada temporal.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 con GCC, utilizando la interfaz definida en `api_sesion21.h`, sin bibliotecas no estándar y compilando con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar `nuevo_temporal()` para generar nombres únicos como `_t0`, `_t1`, `_t2`, utilizando e incrementando el contador. Implementar `emitir_cuadruplo()` para escribir instrucciones con el formato `(op, a1, a2, res)`. Los argumentos NULL deben representarse con `_`. Ambas funciones deben respetar la capacidad del buffer y no utilizar memoria dinámica ni variables globales. |
| **Output (O)** | Código modular en C, respetando las firmas originales, sin incluir `main()`, con manejo de límites de memoria y análisis de complejidad. |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C y compiladores.

Necesito implementar dos funciones para la Sesión 21 de Lenguajes y Autómatas II, cuyo objetivo es generar código de tres direcciones mediante la representación de instrucciones en cuádruplos.

Utiliza C11 compatible con GCC y respeta exactamente las siguientes firmas:

void nuevo_temporal(char *dst, int cap, int *contador);

void emitir_cuadruplo(char *dst, int cap, const char *op,
                      const char *a1, const char *a2, const char *res);

Requisitos:

1. La función nuevo_temporal() debe generar nombres de temporales con el formato _t0, _t1, _t2, etc.
2. Debe utilizar el valor actual del contador y aumentarlo después de generar el temporal.
3. La función emitir_cuadruplo() debe generar una cadena con el formato:
   (op, a1, a2, res)
4. Cuando cualquiera de los argumentos del cuádruplo sea NULL, debe escribirse un guion bajo (_).
5. Ambas funciones deben escribir directamente en el buffer dst.
6. Deben validar que dst no sea NULL y que cap sea mayor que cero.
7. nuevo_temporal() también debe validar que contador no sea NULL.
8. Utiliza snprintf() para respetar la capacidad del buffer y evitar desbordamientos.
9. No utilices memoria dinámica, variables globales ni incluyas main().
10. Incluye únicamente el código necesario para implementar las funciones.

Respeta el contrato de api_sesion21.h y asegúrate de que el código compile sin advertencias con:
gcc -std=c11 -Wall -Wextra -Werror

Finalmente, explica brevemente la complejidad de ambas funciones.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion21_c.c - Fase 2: El Cientifico
 *
 * Solucion profesional optimizada, utilizando snprintf para asegurar
 * la proteccion de limites de memoria (buffer overflow) respetando el limite 'cap'.
 */
#include "api_sesion21.h"
#include <stdio.h>
#include <string.h>

void nuevo_temporal(char *dst, int cap, int *contador) {
    if (dst != NULL && cap > 0 && contador != NULL) {
        snprintf(dst, (size_t)cap, "_t%d", *contador);
        (*contador)++;
    }
}

void emitir_cuadruplo(char *dst, int cap, const char *op, const char *a1, const char *a2, const char *res) {
    if (dst != NULL && cap > 0) {
        const char *real_op  = (op != NULL)  ? op  : "_";
        const char *real_a1  = (a1 != NULL)  ? a1  : "_";
        const char *real_a2  = (a2 != NULL)  ? a2  : "_";
        const char *real_res = (res != NULL) ? res : "_";

        snprintf(dst, (size_t)cap, "(%s, %s, %s, %s)", real_op, real_a1, real_a2, real_res);
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones aritméticas compuestas

### Salida del harness

**Ejecución de Artesano:**
```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md
```

**Ejecución de la IA:**
```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _) OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md
```

- [x] Caso base y límites probados sin *Crash*.
- [x] No se utiliza memoria dinámica y `snprintf()` limita las escrituras al tamaño del buffer.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código identifiqué que el harness utiliza buffers suficientemente grandes, por lo que no comprueba qué sucede cuando la capacidad es demasiado pequeña. Aunque la IA utiliza `snprintf()` para evitar escribir fuera del buffer, no verifica si el texto se alcanzó a guardar completo. En el caso de `nuevo_temporal()`, el contador aumenta incluso si el nombre quedó incompleto por falta de espacio. Para mejorar este comportamiento, se debe comprobar el valor que devuelve `snprintf()` y aumentar el contador solamente cuando el temporal se haya escrito completo. También sería conveniente probar buffers pequeños y argumentos NULL para comprobar estos casos adicionales. Las seis pruebas proporcionadas sí fueron superadas, pero no cubren todos esos escenarios.

---
**Firma digital:** josue6martin

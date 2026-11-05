
# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353      
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Evaluador recursivo de operaciones  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Durante la implementación manual, una de las principales dificultades que encontré fue controlar correctamente los errores al evaluar expresiones aritméticas de manera recursiva. Especialmente tuve que cuidar que las operaciones de división y módulo verificaran primero si el divisor era cero, ya que de lo contrario el programa podría fallar. También tuve que comprender cómo propagar los errores desde los nodos hijos hasta el nodo principal sin continuar realizando operaciones inválidas. Para solucionarlo, utilicé condiciones `if`, validaciones de punteros `NULL` y el parámetro `error`, asignando el valor 1 cuando ocurre una división entre cero y 2 cuando se encuentra un nodo inválido. Con esto logré evaluar correctamente las expresiones y evitar errores durante la ejecución.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC en Google Colab, utilizando bibliotecas estándar y compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores, especializado en evaluación recursiva de AST y detección de errores semánticos. |
| **Especificación (E)** | Implementar `evaluar(ASTNode *n, int *error)` para evaluar operaciones `+`, `-`, `*`, `/` y `%`. Detectar división o módulo entre cero asignando `error=1`, identificar nodos inválidos con `error=2` y propagar los errores sin continuar la evaluación del padre. |
| **Output (O)** | Solo código modular en C, sin `main()`, compatible con `api_sesion07.h` y el test harness. Complejidad temporal O(n) y memoria auxiliar O(h), donde h es la altura del árbol. |

### 2.2 Prompt Estructurado

```text
Contexto (C):
Estoy realizando la Sesión 07 de Lenguajes y
Autómatas II (SCD-1016), utilizando C99 y GCC
en Google Colab.

El código debe compilar correctamente con:
-Wall -Wextra -Werror

Se deben utilizar únicamente bibliotecas estándar
del lenguaje C.

Rol (R):
Actúa como un ingeniero de software especializado
en compiladores, árboles sintácticos abstractos,
evaluación recursiva y detección de errores
semánticos.

Especificación (E):
Implementa la siguiente función respetando
exactamente su firma:

int evaluar(ASTNode *n, int *error);

Requisitos:
1. Utilizar la estructura ASTNode definida
   en api_sesion07.h.
2. Evaluar expresiones aritméticas recursivamente.
3. Soportar los operadores +, -, *, / y %.
4. Si el nodo es numérico, devolver su valor.
5. Evaluar primero el subárbol izquierdo
   y después el derecho.
6. Verificar que el divisor sea diferente
   de cero antes de realizar / o %.
7. Si ocurre división o módulo entre cero,
   asignar error = 1.
8. Si el nodo es NULL o inválido,
   asignar error = 2.
9. Propagar los errores de los nodos hijos
   sin continuar evaluando el nodo padre.
10. Validar que el puntero error no sea NULL.
11. No modificar la firma del contrato.
12. No incluir la función main().

Casos esperados:
- (2+3)*4 = 20
- 7/0 = error 1
- 7%0 = error 1
- evaluar(NULL) = error 2
- 10-2-3 = 5
- (8/2)+1 = 5

Output (O):
Devuelve el código fuente en C para
ia_sesion07_c.c, compatible con
api_sesion07.h y harness_sesion07_c.c.

Incluye un análisis breve de complejidad:
Tiempo O(n) y memoria auxiliar O(h).
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion07_c.c - Fase 2: El Cientifico
 * Evaluador recursivo optimizado con manejo
 * estricto de errores.
 */

#include "api_sesion07.h"
#include <stddef.h>

int evaluar(ASTNode *n, int *error) {

    if (error == NULL) {
        return 0;
    }

    // Si ya existe un estado de error previo
    // propagado, detenemos la evaluacion.
    if (*error != 0) {
        return 0;
    }

    // Validacion de puntero nulo (error 2)
    if (n == NULL) {
        *error = 2;
        return 0;
    }

    // Evaluacion de nodo numerico
    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    // Evaluacion de operador binario
    if (n->tipo == NODO_BINOP) {

        int izq = evaluar(n->izq, error);

        if (*error != 0) {
            return 0;
        }

        int der = evaluar(n->der, error);

        if (*error != 0) {
            return 0;
        }

        switch (n->op) {

            case '+':
                return izq + der;

            case '-':
                return izq - der;

            case '*':
                return izq * der;

            case '/':
                if (der == 0) {
                    *error = 1;
                    return 0;
                }
                return izq / der;

            case '%':
                if (der == 0) {
                    *error = 1;
                    return 0;
                }
                return izq % der;

            default:
                *error = 2;
                return 0;
        }
    }

    *error = 2;
    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Expresiones complejas y división por 0

* [x] Caso base y límites superados sin *Crash* en las pruebas ejecutadas.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

**Resultados obtenidos en Google Colab:**

```text
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===
Caso                               esperado:              obtenido:              resultado
[ 1] (2+3)*4                            esperado: 20           obtenido: 20           OK
[ 2] 7/0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 3] 7%0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 4] arbol NULL -> error 2              esperado: 2            obtenido: 2            OK
[ 5] 10-2-3                             esperado: 5            obtenido: 5            OK
[ 6] (8/2)+1                            esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion07.md
```

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código generado por la IA, observé que cumplía correctamente con las operaciones aritméticas y detectaba los errores de división y módulo entre cero. Sin embargo, identifiqué que no contemplaba otro caso límite: el desbordamiento de enteros durante operaciones como suma, resta y multiplicación. También existe un caso especial al dividir el valor mínimo de un entero entre -1, ya que el resultado puede quedar fuera del rango permitido. Para solucionar estas limitaciones, sería necesario agregar validaciones utilizando los límites definidos en `limits.h` antes de realizar las operaciones. Además, comprobé que la IA validaba los punteros nulos y propagaba correctamente los errores de los nodos hijos. Finalmente, ejecuté el test harness en Google Colab y obtuve 6/6 pruebas superadas tanto en mi implementación manual como en la generada por IA. Esta actividad me ayudó a comprender mejor cómo funcionan los evaluadores de expresiones, la recursividad y la importancia de detectar errores antes de ejecutar operaciones que puedan provocar fallos.

---

**Firma digital:** Josue Efrain

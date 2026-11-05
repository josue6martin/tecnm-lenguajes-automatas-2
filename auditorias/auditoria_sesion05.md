
# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Recorridos del AST y Notación Polaca  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353       
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Impresión en Postorden / RPN  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Durante la implementación manual, una de las principales dificultades que encontré fue realizar correctamente el recorrido postorden del árbol sintáctico, respetando el orden izquierdo, derecho y raíz. También tuve que cuidar el manejo de los buffers para evitar escribir fuera de sus límites y asegurar que las expresiones quedaran separadas por un solo espacio. Para solucionarlo, utilicé funciones recursivas, `snprintf()` y `strncpy()`, además de validar los punteros `NULL` y la capacidad del buffer. Con esto logré convertir las expresiones del AST a notación polaca inversa sin imprimir directamente en pantalla.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC en Google Colab, utilizando bibliotecas estándar y compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores, especializado en árboles sintácticos, recursividad y manejo de memoria. |
| **Especificación (E)** | Implementar `postorden(ASTNode *n, char *buf, int cap)` respetando el recorrido izquierdo, derecho y raíz. Convertir un AST a notación polaca inversa (RPN), separar los elementos mediante un espacio, evitar espacios finales y respetar la capacidad del buffer. Si el árbol es `NULL`, devolver una cadena vacía. |
| **Output (O)** | Solo código modular en C, sin `main()`, compatible con `api_sesion05.h` y el test harness. Complejidad temporal O(n) y memoria auxiliar O(h), donde h representa la altura del árbol. |

### 2.2 Prompt Estructurado
```text
Contexto (C):
Estoy realizando la Sesión 05 de Lenguajes y
Autómatas II (SCD-1016), utilizando C99 y GCC
en Google Colab.

El código debe compilar correctamente con:
-Wall -Wextra -Werror

Se deben utilizar únicamente bibliotecas estándar
del lenguaje C.

Rol (R):
Actúa como un ingeniero de software especializado
en compiladores, árboles sintácticos abstractos
(AST), recursividad y manejo seguro de buffers.

Especificación (E):
Implementa la siguiente función respetando
exactamente su firma:

void postorden(ASTNode *n, char *buf, int cap);

Requisitos:
1. Utilizar la estructura ASTNode definida
   en api_sesion05.h.
2. Recorrer el árbol en postorden:
   izquierdo, derecho y raíz.
3. Convertir el AST a notación polaca inversa.
4. Guardar el resultado en el buffer recibido.
5. No utilizar printf() para mostrar resultados.
6. Separar los números y operadores con
   un solo espacio.
7. No dejar espacios al final de la cadena.
8. Respetar la capacidad máxima del buffer.
9. Garantizar la terminación con '\0'.
10. Si el árbol es NULL, generar cadena vacía.
11. Validar que buf no sea NULL y cap sea positivo.
12. No modificar la firma establecida.
13. No incluir la función main().

Casos esperados:
- Hoja 42: "42"
- 3+(5*2): "3 5 2 * +"
- Árbol NULL: ""
- (1+2)*(3+4): "1 2 + 3 4 + *"
- 8-3: "8 3 -"

Output (O):
Devuelve el código fuente en C para
ia_sesion05_c.c, compatible con
api_sesion05.h y harness_sesion05_c.c.

Incluye un análisis breve de complejidad:
Tiempo O(n) y memoria auxiliar O(h).
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion05_c.c - Fase 2: El Cientifico
 *
 * Recorrido postorden y notacion polaca inversa.
 *
 * Implementacion utilizando un buffer compartido.
 */

#include "api_sesion05.h"
#include <stdio.h>
#include <string.h>

/* Funcion auxiliar para recorrer el AST */
static void postorden_helper(ASTNode *n, char *buf,
                            int cap, int *offset) {
    if (n == NULL) return;

    // Recorrido Postorden: Izquierda, Derecha
    postorden_helper(n->izq, buf, cap, offset);
    postorden_helper(n->der, buf, cap, offset);

    // Agregar espacio antes del siguiente token
    if (*offset > 0 && *offset < cap - 1) {
        buf[*offset] = ' ';
        (*offset)++;
        buf[*offset] = '\0';
    }

    // Escribir el valor o el operador actual
    int escritos = 0;

    if (n->tipo == NODO_NUM) {
        escritos = snprintf(
            buf + *offset,
            cap - *offset,
            "%d",
            n->valor
        );
    } else if (n->tipo == NODO_BINOP) {
        escritos = snprintf(
            buf + *offset,
            cap - *offset,
            "%c",
            n->op
        );
    }

    if (escritos > 0) {
        *offset += escritos;
    }
}

/* Funcion principal del recorrido postorden */
void postorden(ASTNode *n, char *buf, int cap) {
    if (buf == NULL || cap <= 0) return;

    buf[0] = '\0';

    int offset = 0;

    postorden_helper(n, buf, cap, &offset);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Validación contra desbordamiento de pila

* [x] Caso base y límites superados sin *Crash* en las pruebas realizadas.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada para los casos probados.

**Resultados obtenidos en Google Colab:**

```text
=== AUDITORIA SESION 05 - Recorrido postorden y RPN ===
Caso                               esperado:              obtenido:              resultado
[ 1] hoja 42                            esperado: 42           obtenido: 42           OK
[ 2] 3+(5*2)                            esperado: 3 5 2 * +    obtenido: 3 5 2 * +    OK
[ 3] arbol NULL                         esperado:              obtenido:              OK
[ 4] (1+2)*(3+4)
     esperado: 1 2 + 3 4 + *
     obtenido: 1 2 + 3 4 + *
     -> OK
[ 5] 8-3                                esperado: 8 3 -        obtenido: 8 3 -        OK

RESULTADO: 5/5 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion05.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, observé que realizaba correctamente el recorrido postorden y producía las expresiones en notación polaca inversa. Sin embargo, identifiqué que no manejaba adecuadamente los casos donde el buffer era demasiado pequeño. La función `snprintf()` puede devolver una cantidad de caracteres mayor que el espacio disponible y, al sumarla directamente al desplazamiento, podría provocar accesos fuera de los límites del buffer. Para solucionar este problema, sería necesario comprobar el espacio disponible antes de escribir y actualizar el desplazamiento únicamente con los caracteres realmente almacenados. También observé que la recursividad puede consumir demasiada memoria de pila si el árbol es excesivamente profundo, por lo que sería importante probar estos casos por separado. Finalmente, ejecuté el test harness en Google Colab y comprobé que tanto mi implementación manual como la generada por IA superaron las 5 pruebas establecidas. Esta actividad me ayudó a comprender mejor los recorridos de árboles, la recursividad y la importancia de controlar los límites de memoria al trabajar con cadenas en C.

---
**Firma digital:** Josue Martin

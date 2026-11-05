
# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Anatomía de un Árbol Sintáctico (AST)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353      
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de nodo binario (ASTNode)  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Durante la implementación manual, una de las dificultades que encontré fue comprender cómo conectar los nodos de un árbol sintáctico utilizando punteros. También tuve que asegurarme de reservar correctamente la memoria con `malloc()` e inicializar los campos de cada nodo para evitar errores. Otra dificultad fue implementar la evaluación recursiva, ya que primero se deben evaluar los nodos hijos antes de realizar la operación del nodo principal. Para solucionarlo, utilicé la estructura `ASTNode`, validaciones de punteros `NULL` y condiciones para identificar los diferentes tipos de nodos y operadores.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC en Google Colab, utilizando bibliotecas estándar y compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores, especializado en estructuras de datos, punteros y árboles sintácticos abstractos. |
| **Especificación (E)** | Implementar un AST basado en la gramática `E -> E + T \| T`, utilizando los tipos `NODO_NUM` y `NODO_BINOP`. Crear nodos numéricos y binarios, conservar sus hijos izquierdo y derecho, y evaluar expresiones con operadores `+`, `-` y `*`. La función `evaluar(NULL)` debe devolver -1. |
| **Output (O)** | Solo código modular en C, sin `main()`, compatible con `api_sesion04.h` y el test harness. Complejidad temporal O(n) y espacial O(h) durante la evaluación recursiva. |

### 2.2 Prompt Estructurado
```text
Contexto (C):
Estoy realizando la Sesión 04 de Lenguajes y
Autómatas II (SCD-1016), utilizando C99 y GCC
en Google Colab.

El código debe compilar correctamente utilizando
-Wall -Wextra -Werror y únicamente bibliotecas
estándar del lenguaje C.

Rol (R):
Actúa como un ingeniero de software especializado
en compiladores, estructuras de datos, punteros
y árboles sintácticos abstractos (AST).

Especificación (E):
Implementa las siguientes funciones respetando
exactamente sus firmas:

ASTNode *crear_numero(int v);
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der);
int evaluar(ASTNode *n);

Requisitos:
1. Utilizar la estructura ASTNode definida
   en api_sesion04.h.
2. Diferenciar los nodos numéricos y binarios
   utilizando NODO_NUM y NODO_BINOP.
3. Reservar memoria dinámica mediante malloc().
4. Verificar si la asignación de memoria falla.
5. Inicializar correctamente todos los campos.
6. Conservar los punteros de los hijos izquierdo
   y derecho en los nodos binarios.
7. Evaluar expresiones mediante recursividad.
8. Soportar los operadores +, - y *.
9. evaluar(NULL) debe devolver -1 sin provocar
   errores de segmentación.
10. Devolver -1 si el operador no es válido.
11. No modificar las firmas del contrato.
12. No incluir la función main().

Output (O):
Devuelve únicamente el código fuente en C para
el archivo ia_sesion04_c.c.

El código debe ser modular, compatible con
api_sesion04.h y harness_sesion04_c.c.

Incluye un análisis de complejidad:
- Creación de nodos: O(1) en tiempo y espacio.
- Evaluación del AST: O(n) en tiempo.
- Memoria auxiliar de evaluación: O(h),
  donde h es la altura del árbol.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion04_c.c - Fase 2: El Cientifico
 *
 * Implementacion de un arbol sintactico
 * abstracto (AST) utilizando nodos binarios.
 */

#include "api_sesion04.h"
#include <stdlib.h>

/*
 * Creacion de un nodo numerico en el AST.
 * Se reserva memoria dinamicamente y
 * se inicializan todos los miembros.
 */
ASTNode *crear_numero(int v) {
    ASTNode *nodo = malloc(sizeof(ASTNode));

    if (!nodo) {
        return NULL;
    }

    nodo->tipo = NODO_NUM;
    nodo->valor = v;
    nodo->op = '\0';
    nodo->izq = NULL;
    nodo->der = NULL;

    return nodo;
}

/*
 * Creacion de un nodo de operacion binaria.
 * Asigna el operador y conecta los subarboles
 * izquierdo y derecho.
 */
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der) {
    ASTNode *nodo = malloc(sizeof(ASTNode));

    if (!nodo) {
        return NULL;
    }

    nodo->tipo = NODO_BINOP;
    nodo->valor = 0;
    nodo->op = op;
    nodo->izq = izq;
    nodo->der = der;

    return nodo;
}

/*
 * Evaluacion recursiva del AST.
 * Soporta suma, resta y multiplicacion.
 * Devuelve -1 si el nodo es NULL
 * o si el operador no esta soportado.
 */
int evaluar(ASTNode *n) {
    if (n == NULL) {
        return -1;
    }

    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    if (n->tipo == NODO_BINOP) {
        int izq_val = evaluar(n->izq);
        int der_val = evaluar(n->der);

        switch (n->op) {
            case '+':
                return izq_val + der_val;

            case '-':
                return izq_val - der_val;

            case '*':
                return izq_val * der_val;

            default:
                return -1;
        }
    }

    return -1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Árbol vacío y evaluación de nodo raíz

* [x] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

**Resultados obtenidos en Google Colab:**

```text
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                               esperado:              obtenido:              resultado
[ 1] evaluar(nodo 42)                   esperado: 42           obtenido: 42           OK
[ 2] evaluar(3+5*2)                     esperado: 13           obtenido: 13           OK
[ 3] hijo izquierdo de (* 5 2)          esperado: 5            obtenido: 5            OK
[ 4] operador guardado                  esperado: *            obtenido: *            OK
[ 5] evaluar(NULL)                      esperado: -1           obtenido: -1           OK
[ 6] evaluar(10-4)                      esperado: 6            obtenido: 6            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion04.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, observé que las funciones cumplían con los requisitos principales y evaluaban correctamente los árboles sintácticos. Sin embargo, identifiqué una limitación que no estaba contemplada: cuando un nodo binario tiene un hijo `NULL`, la función `evaluar()` devuelve -1 para ese hijo, pero después podría utilizar ese valor como si fuera un número válido. Para evitar este problema, sería necesario validar que ambos hijos existan antes de realizar la operación. También observé que la IA reserva memoria con `malloc()`, pero no incluye una función para liberar los nodos mediante `free()`, por lo que sería importante agregar una rutina de liberación para evitar fugas de memoria. Finalmente, ejecuté el test harness en Google Colab y comprobé que tanto mi implementación manual como la generada por IA superaron las 6 pruebas. Esta actividad me permitió comprender mejor el uso de punteros, la memoria dinámica y la recursividad para construir y evaluar árboles sintácticos.

---
**Firma digital:** Josue Martin

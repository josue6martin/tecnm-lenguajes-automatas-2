
# Bitácora de Requerimientos y Auditoría IA — Sesión 22
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Generación de TAC para Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353            
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Traductor de nodos AST a TAC  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar el traductor manualmente, tuve algunas dificultades para comprender cómo recorrer correctamente el árbol AST sin cambiar el orden de las operaciones. Para solucionarlo utilicé un recorrido en postorden, donde primero se procesan los nodos hijos y después se genera la instrucción del operador. También tuve que cuidar la creación de los temporales para que siguieran un orden consecutivo, como _t0, _t1 y _t2. Otra dificultad fue controlar el tamaño de los arreglos para no escribir más líneas de las permitidas y evitar que las cadenas superaran los 63 caracteres disponibles.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Conservación de precedencia de operadores |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en sistemas y compiladores.

CONTEXTO:
Estoy desarrollando la práctica de la Sesión 22 de Lenguajes y Autómatas II.
Necesito implementar un traductor de árboles de expresión AST a código
de tres direcciones (TAC), utilizando C99 y GCC.

ROL:
Actúa como un programador experto en C y desarrollo de compiladores.

ESPECIFICACIONES:
1. Respeta exactamente la interfaz de api_sesion22.h.
2. Implementa la función:
   int generar_tac(const Nodo *raiz, char lineas[][64], int max);
3. Realiza un recorrido recursivo en postorden.
4. Primero procesa los nodos izquierdo y derecho y después genera
   la instrucción del operador.
5. Cada operador debe generar exactamente una instrucción TAC.
6. Las hojas deben devolver su valor sin generar instrucciones.
7. Genera temporales consecutivos (_t0, _t1, _t2...).
8. Conserva el orden de las operaciones del árbol AST.
9. No escribas más de max líneas.
10. No superes los 63 caracteres por línea.
11. Maneja correctamente una raíz NULL.
12. Evita accesos fuera de los límites de los arreglos.
13. Utiliza snprintf para escribir las instrucciones.
14. No incluyas main(), porque el harness proporciona el suyo.

OUTPUT:
Entrega código C modular y compatible con:
gcc -std=c11 -Wall -Wextra -Werror.

Incluye funciones auxiliares para manejar los temporales y recorrer
el árbol. Explica la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion22.h"
#include <stdio.h>
#include <string.h>

typedef struct {
    char nombre[64][16];
    int usados;
} Pool;

static const char *obtener_temporal_ia(Pool *p, int *contador) {
    if (p->usados >= 64) {
        return "_t?";
    }

    snprintf(p->nombre[p->usados],
             sizeof(p->nombre[0]),
             "_t%d", (*contador)++);

    return p->nombre[p->usados++];
}

static const char *resolver_nodo(
    const Nodo *n,
    char lineas[][64],
    int max,
    int *total,
    Pool *p,
    int *contador
) {
    if (!n) {
        return "";
    }

    if (n->tipo == 'H') {
        return n->valor;
    }

    const char *op_izq = resolver_nodo(
        n->izq, lineas, max, total, p, contador
    );

    const char *op_der = resolver_nodo(
        n->der, lineas, max, total, p, contador
    );

    const char *temp = obtener_temporal_ia(p, contador);

    if (*total < max) {
        snprintf(
            lineas[*total],
            64,
            "%s = %s %c %s",
            temp,
            op_izq,
            n->op,
            op_der
        );

        (*total)++;
    }

    return temp;
}

int generar_tac(const Nodo *raiz, char lineas[][64], int max) {
    if (!raiz) {
        return 0;
    }

    Pool pool;
    pool.usados = 0;

    int contador_temporales = 0;
    int lineas_emitidas = 0;

    resolver_nodo(
        raiz,
        lineas,
        max,
        &lineas_emitidas,
        &pool,
        &contador_temporales
    );

    return lineas_emitidas;
}
```

**Análisis del código:**

- Se utiliza una estructura `Pool` para almacenar los nombres de los temporales.
- La función `obtener_temporal_ia()` genera los temporales consecutivos.
- La función `resolver_nodo()` recorre el árbol en postorden.
- Las hojas devuelven directamente su valor.
- Los operadores generan una instrucción TAC utilizando `snprintf`.
- La función `generar_tac()` inicializa los contadores y devuelve el número de instrucciones generadas.

**Complejidad teórica:**

- Tiempo: O(n), porque cada nodo del árbol se visita una sola vez.
- Espacio: O(n) en el peor caso, considerando la recursividad y el almacenamiento de temporales, con un límite de 64 temporales en esta implementación.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Expresiones anidadas con paréntesis.

### Salida del harness

```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks* en los casos probados.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar la implementación de la IA, identifiqué que no se contemplaba completamente el caso de árboles con más de 64 operadores, debido a que el arreglo de temporales tiene una capacidad limitada. También observé que era necesario revisar qué sucede cuando el valor de max es cero o negativo, ya que no debe intentarse escribir en un arreglo sin espacio disponible. Aunque el harness pasó las seis pruebas, estos casos adicionales no fueron evaluados directamente. Por eso revisé las condiciones de los contadores y los límites de los arreglos para identificar posibles problemas. Esto me permitió entender que no solamente se debe comprobar que el programa funcione con los ejemplos solicitados, sino también revisar cómo responde ante entradas fuera de los casos habituales.

---

**Firma digital:** josue6martin

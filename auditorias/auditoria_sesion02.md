
# Bitácora de Requerimientos y Auditoría IA — Sesión 02

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Lista enlazada de tokens en Heap

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Durante la implementación manual identifiqué que la principal dificultad era administrar correctamente la memoria dinámica para crear y conectar los nodos de una lista enlazada de tokens.
>
> Fue necesario utilizar `malloc()` para reservar memoria en el heap y comprobar que la asignación se realizara correctamente antes de almacenar cada lexema.
>
> Otro aspecto importante fue controlar los punteros, especialmente el campo `siguiente`, que debía inicializarse en `NULL` para identificar el final de la lista y evitar accesos inválidos.
>
> También tuve que considerar el límite de 16 caracteres del arreglo `lexema`, utilizando `strncpy()` y agregando explícitamente el carácter `\0` para evitar desbordamientos.
>
> Finalmente, fue necesario recorrer la lista y liberar cada nodo mediante `free()`, guardando previamente la dirección del siguiente nodo para no acceder a memoria que ya había sido liberada.
>
> Después de completar la implementación manual, ejecuté el Test Harness en Google Colab y obtuve 6 de 6 pruebas superadas.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C99, compilador GCC 13.3.0 y bibliotecas estándar de C. Entorno Google Colab. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Implementar una lista enlazada de tokens utilizando `malloc()`, `free()`, punteros y gestión segura de memoria dinámica, respetando las funciones declaradas en `api_sesion02.h`. |
| **Output (O)** | Código modular en C99, sin función `main()`, compatible con el Test Harness y acompañado de un análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software de sistemas
y compiladores experto en lenguaje C99 y gestión
de memoria dinámica.

Necesito desarrollar una implementación modular
de una lista enlazada de tokens almacenada en el heap.

El código debe respetar exactamente las siguientes
funciones declaradas en api_sesion02.h:

NodoToken *crear_nodo(const char *lexema);
void agregar_token(NodoToken **cabeza, const char *lexema);
int contar_tokens(NodoToken *cabeza);
void liberar_lista(NodoToken *cabeza);

La estructura NodoToken contiene un arreglo de
16 caracteres llamado lexema y un puntero siguiente.

La implementación debe cumplir los siguientes
requisitos:

1. Utilizar lenguaje C99 y bibliotecas estándar.
2. Reservar memoria dinámica con malloc().
3. Verificar que malloc() no devuelva NULL.
4. Validar punteros NULL antes de utilizarlos.
5. Copiar los lexemas sin desbordar el arreglo.
6. Garantizar la terminación de cada cadena con '\0'.
7. Inicializar el campo siguiente en NULL.
8. Agregar los nuevos tokens al final de la lista.
9. Conservar el orden de inserción de los tokens.
10. Contar correctamente los nodos de la lista.
11. Retornar cero cuando la lista esté vacía.
12. Liberar todos los nodos utilizando free().
13. Evitar accesos a memoria después de liberarla.
14. Permitir liberar_lista(NULL) sin provocar fallos.
15. No utilizar arreglos globales ni incluir main().
16. Compilar sin advertencias con:
    -Wall -Wextra -Werror.

Entrega únicamente el código modular compatible
con api_sesion02.h y un análisis de complejidad
temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

El siguiente código corresponde a la implementación
de la inteligencia artificial utilizada durante
la Fase 2 en Google Colab.

**Archivo:** `ia_sesion02_c.c`

```c
/* ia_sesion02_c.c - Fase 2: El Cientifico
 *
 * Implementacion de una lista enlazada
 * de tokens utilizando memoria dinamica.
 */

#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

NodoToken *crear_nodo(const char *lexema) {
    if (!lexema) return NULL;

    NodoToken *nuevo = (NodoToken *)malloc(sizeof(NodoToken));
    if (!nuevo) return NULL;

    // Copia segura de cadenas evitando desbordamiento.
    strncpy(nuevo->lexema, lexema, sizeof(nuevo->lexema) - 1);
    nuevo->lexema[sizeof(nuevo->lexema) - 1] = '\0';

    nuevo->siguiente = NULL;

    return nuevo;
}

void agregar_token(NodoToken **cabeza, const char *lexema) {
    if (!cabeza) return;

    NodoToken *nuevo = crear_nodo(lexema);
    if (!nuevo) return;

    if (*cabeza == NULL) {
        *cabeza = nuevo;
    } else {
        NodoToken *actual = *cabeza;

        while (actual->siguiente != NULL) {
            actual = actual->siguiente;
        }

        actual->siguiente = nuevo;
    }
}

int contar_tokens(NodoToken *cabeza) {
    int contador = 0;
    NodoToken *actual = cabeza;

    while (actual) {
        contador++;
        actual = actual->siguiente;
    }

    return contador;
}

void liberar_lista(NodoToken *cabeza) {
    NodoToken *actual = cabeza;

    while (actual) {
        NodoToken *siguiente = actual->siguiente;
        free(actual);
        actual = siguiente;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Prevención de Use-After-Free, cadenas nulas, creación de nodos, conteo de tokens, orden de inserción y liberación de listas vacías.

**Comandos utilizados en Google Colab:**

Compilación y ejecución de la implementación manual:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion02_c.c artesano_sesion02_c.c -o test_artesano && ./test_artesano
```

Compilación y ejecución de la implementación generada por IA:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion02_c.c ia_sesion02_c.c -o test_ia && ./test_ia
```

**Resultados obtenidos en ambas implementaciones:**

```text
=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===
Caso                               esperado:              obtenido:              resultado
[ 1] crear_nodo("if")->lexema           esperado: if           obtenido: if           OK
[ 2] contar_tokens(3 tokens)            esperado: 3            obtenido: 3            OK
[ 3] orden: segundo token               esperado: dos          obtenido: dos          OK
[ 4] ultimo->siguiente == NULL          esperado: 1            obtenido: 1            OK
[ 5] contar_tokens(NULL)                esperado: 0            obtenido: 0            OK
[ 6] liberar_lista(NULL)                esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion02.md
```

**Verificación de requisitos:**

* [x] Caso base y límites evaluados superados sin *Crash*.
* [x] Los casos probados no mostraron accesos fuera de límites ni fallos durante la liberación. No se ejecutaron sanitizadores para descartar exhaustivamente *Memory Leaks* o *Use-After-Free*.
* [x] La complejidad teórica coincide con la implementación analizada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Durante la auditoría observé que la inteligencia artificial implementó correctamente la creación, inserción, conteo y liberación de nodos, logrando superar las seis pruebas del Test Harness. Sin embargo, identifiqué algunas limitaciones que no fueron evaluadas directamente.
>
> Una de ellas es el manejo de lexemas que superan los 15 caracteres. Aunque se utilizó `strncpy()` para evitar escribir fuera del arreglo de 16 caracteres, los textos demasiado largos se recortan, lo que podría ocasionar pérdida de información durante el análisis léxico.
>
> También identifiqué que el Test Harness comprueba la ejecución de `liberar_lista(NULL)` y una liberación normal, pero no utiliza herramientas especializadas para detectar fugas de memoria o accesos después de una liberación.
>
> Como medida de protección, la implementación verifica los punteros NULL, comprueba el resultado de `malloc()`, garantiza la terminación de las cadenas mediante `\0` y guarda el puntero al siguiente nodo antes de utilizar `free()`.
>
> Para fortalecer la auditoría, sería conveniente agregar pruebas con lexemas largos y utilizar herramientas como AddressSanitizer o Valgrind, con el objetivo de comprobar la seguridad de memoria en situaciones adicionales.
>
> Finalmente, confirmé que tanto la implementación manual como la generada por IA superaron las seis pruebas del Test Harness, obteniendo el resultado de 6/6 y cumpliendo con los requisitos funcionales establecidos para la sesión.
> 

---

**Firma digital:** josue6martin

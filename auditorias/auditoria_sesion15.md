
# Bitácora de Requerimientos y Auditoría IA — Sesión 15

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Bloques Anidados y Ciclo de Vida  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353            
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Operaciones Push y Pop de ámbito

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar esta actividad, tuve algunas dificultades para comprender cómo se debían conectar los ámbitos mediante punteros. También tuve que tener cuidado al reservar memoria para cada nuevo ámbito utilizando `malloc()`, ya que cada vez que se abre un bloque se crea una nueva estructura relacionada con su padre. Otro detalle importante fue utilizar correctamente `free()` al cerrar un bloque, porque primero se debe guardar la dirección del padre para no perder la referencia. Comprendí que no se debe liberar el ámbito raíz, ya que es el punto de inicio de toda la cadena.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. Uso de estructuras enlazadas y memoria dinámica. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Implementar las operaciones `push_ambito()` y `pop_ambito()` para crear y cerrar ámbitos anidados, conservar la relación con el padre y liberar correctamente la memoria al salir de un bloque. |
| **Output (O)** | Código modular en C, respetando las firmas de `api_sesion15.h`, sin función `main()` y con análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en la Sesión 15 de Lenguajes y Autómatas II.
Necesito implementar un sistema de ámbitos anidados en lenguaje C,
utilizando estructuras enlazadas y memoria dinámica para controlar
el ciclo de vida de los identificadores.

Rol:
Actúa como un ingeniero de software especializado en C y desarrollo
de compiladores.

Especificación:
Implementa las siguientes funciones respetando exactamente sus firmas:

Ambito *push_ambito(Ambito *actual, const char *nombre);
Ambito *pop_ambito(Ambito *actual);

La estructura Ambito contiene:
- const char *nombre;
- struct Ambito *padre;

La función push_ambito() debe:
- Recibir el ámbito actual y el nombre del nuevo bloque.
- Devolver NULL si el ámbito actual o el nombre son NULL.
- Reservar memoria para un nuevo ámbito utilizando malloc().
- Guardar el nombre recibido.
- Enlazar el nuevo ámbito con su padre.
- Devolver el puntero al nuevo ámbito.
- Comprobar si la reserva de memoria falla.

La función pop_ambito() debe:
- Recibir el ámbito actual.
- Devolver NULL si recibe NULL o si se intenta cerrar la raíz.
- Guardar primero la dirección del padre.
- Liberar únicamente el ámbito actual utilizando free().
- Devolver el puntero al padre.

Restricciones:
- Utilizar C11 compatible con GCC.
- Incluir api_sesion15.h.
- No modificar las estructuras ni las firmas.
- No incluir main().
- No utilizar bibliotecas no estándar.
- Compilar sin advertencias con -Wall -Wextra -Werror.
- Evitar fugas de memoria y accesos inválidos.

Output:
Entrega el código modular de ambas funciones con comentarios
explicativos y un análisis de complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion15_c.c - Fase 2: El Cientifico
 *
 * Solución optimizada proporcionada por la IA asistida.
 */

#include "api_sesion15.h"
#include <stdlib.h>

Ambito *push_ambito(Ambito *actual, const char *nombre) {
    if (actual == NULL || nombre == NULL) {
        return NULL;
    }

    // Reservar memoria para el nuevo ámbito.
    Ambito *nuevo = malloc(sizeof(*nuevo));

    if (nuevo == NULL) {
        return NULL;
    }

    // Asignar el nombre y enlazar con el padre.
    nuevo->nombre = nombre;
    nuevo->padre = actual;

    return nuevo;
}

Ambito *pop_ambito(Ambito *actual) {
    // Verificar que exista un ámbito y que no sea la raíz.
    if (actual == NULL || actual->padre == NULL) {
        return NULL;
    }

    // Guardar la dirección del padre antes de liberar.
    Ambito *padre = actual->padre;

    // Liberar el ámbito actual.
    free(actual);

    // Regresar al ámbito anterior.
    return padre;
}
```

**Análisis de complejidad:**

- Complejidad temporal de `push_ambito()`: O(1), porque únicamente valida los datos, reserva memoria y asigna los valores.
- Complejidad temporal de `pop_ambito()`: O(1), porque guarda el padre y libera únicamente el ámbito actual.
- Complejidad espacial: O(1) por operación, porque cada nuevo ámbito reserva una sola estructura de tamaño constante.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Aislamiento de variables fuera de bloque.

### Salida del harness

```text
=== AUDITORIA SESION 15 - Push y pop de ambitos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre del ambito nuevo            esperado: if           obtenido: if           OK
[ 2] profundidad con dos push           esperado: 3            obtenido: 3            OK
[ 3] pop devuelve el padre              esperado: 1            obtenido: 1            OK
[ 4] pop sobre la raiz                  esperado: 1            obtenido: 1            OK
[ 5] push con padre NULL                esperado: 1            obtenido: 1            OK
[ 6] profundidad tras pop               esperado: 2            obtenido: 2            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion15.md
```

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks* en las operaciones probadas.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código generado por la IA, observé que era importante considerar qué sucedía cuando se intentaba cerrar el ámbito raíz o cuando se recibía un puntero NULL. También revisé el manejo de la memoria para asegurarme de que primero se guardara la dirección del padre antes de liberar el ámbito actual. Otro detalle fue comprobar que solamente se liberara la estructura del ámbito y no el nombre, porque este se recibe como un puntero constante y no fue reservado por la función. Revisé estos puntos y comprobé su funcionamiento con el harness. Finalmente, las seis pruebas se ejecutaron correctamente y se obtuvo un resultado de 6/6.

---

**Firma digital:** josue6martin

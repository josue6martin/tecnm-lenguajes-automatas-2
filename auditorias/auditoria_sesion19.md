
# Bitácora de Requerimientos y Auditoría IA — Sesión 19
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Estructuras (struct) y Miembros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Offsets de miembros en estructuras  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar el código manual, se me complicó entender cómo calcular la posición de cada campo dentro de una estructura, porque no todos los tipos de datos ocupan el mismo espacio en memoria. Tuve que considerar la alineación de cada miembro y agregar padding cuando era necesario para que el siguiente campo comenzara en una posición correcta. También fue importante calcular el relleno final para que el tamaño total de la estructura fuera múltiplo de la alineación más grande.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Alineación de memoria por palabras |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C y compiladores.

Necesito implementar dos funciones para calcular los offsets de los miembros y el tamaño total de una estructura.

Contexto:
- Utiliza C99/C11 compatible con GCC.
- Incluye la cabecera api_sesion19.h.
- El código debe compilar con -Wall -Wextra -Werror.
- No incluyas una función main(), porque el harness ya contiene una.

Especificación:
1. Implementa:
   int offset_campo(const Campo *campos, int total, const char *nombre);

   Debe recorrer los campos de la estructura y calcular la posición de cada uno.
   Antes de colocar un campo, debe alinear el offset actual según el tamaño de su tipo.
   Debe devolver el offset del campo solicitado.
   Si el campo no existe, la lista es NULL, el nombre es NULL o hay un tipo desconocido, debe devolver -1.

2. Implementa:
   int tamano_struct(const Campo *campos, int total);

   Debe calcular el tamaño total de la estructura considerando:
   - El tamaño de cada campo.
   - La alineación individual de cada miembro.
   - El padding entre campos.
   - El padding final hasta el múltiplo de la alineación más grande.

   Una estructura sin campos debe devolver 0.
   Una lista NULL o un tipo desconocido debe devolver -1.

Restricciones:
- Respeta exactamente las firmas declaradas en api_sesion19.h.
- No modifiques la estructura Campo.
- No agregues main().
- Utiliza funciones auxiliares cuando sea necesario.
- Mantén el código ordenado y modular.

Al final, explica brevemente la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion19.h"
#include <string.h>
#include <stddef.h>

/* Determina el tamaño de alineacion de manera eficiente */
static inline int obtener_tamano_alineacion(const char *tipo) {
    if (!tipo) return -1;
    switch (tipo[0]) {
        case 'c': /* char */
            if (strcmp(tipo, "char") == 0) return 1;
            break;
        case 's': /* short */
            if (strcmp(tipo, "short") == 0) return 2;
            break;
        case 'i': /* int */
            if (strcmp(tipo, "int") == 0) return 4;
            break;
        case 'f': /* float */
            if (strcmp(tipo, "float") == 0) return 4;
            break;
        case 'd': /* double */
            if (strcmp(tipo, "double") == 0) return 8;
            break;
    }
    return -1;
}

int offset_campo(const Campo *campos, int total, const char *nombre) {
    if (!campos || total <= 0 || !nombre) return -1;

    int offset = 0;
    for (int i = 0; i < total; i++) {
        int align = obtener_tamano_alineacion(campos[i].tipo);
        if (align == -1) return -1;

        /* Formula eficiente para redondear el offset hacia arriba */
        offset = (offset + align - 1) / align * align;

        if (strcmp(campos[i].nombre, nombre) == 0) {
            return offset;
        }
        offset += align;
    }
    return -1;
}

int tamano_struct(const Campo *campos, int total) {
    if (!campos || total <= 0) return 0;

    int offset = 0;
    int max_align = 1;

    for (int i = 0; i < total; i++) {
        int align = obtener_tamano_alineacion(campos[i].tipo);
        if (align == -1) return -1;

        if (align > max_align) {
            max_align = align;
        }

        offset = (offset + align - 1) / align * align;
        offset += align;
    }

    /* Padding final de la estructura */
    offset = (offset + max_align - 1) / max_align * max_align;
    return offset;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Acceso a miembros inexistentes

### Salida del harness
```text
=== AUDITORIA SESION 19 - Offsets de miembros y tamano de struct ===
Caso                               esperado:              obtenido:              resultado
[ 1] struct char e int: b               esperado: 4            obtenido: 4            OK
[ 2] struct de tres campos: c           esperado: 8            obtenido: 8            OK
[ 3] tamano total con padding           esperado: 12           obtenido: 12           OK
[ 4] double seguido de char             esperado: 16           obtenido: 16           OK
[ 5] miembro inexistente                esperado: -1           obtenido: -1           OK
[ 6] struct sin campos                  esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion19.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, observé que no se valida si el nombre de algún miembro es NULL antes de utilizar strcmp(). Esto podría provocar un error al intentar comparar un nombre que no existe. También noté que la función tamano_struct() devuelve 0 cuando la lista de campos es NULL, aunque el contrato indica que debe devolver -1 en ese caso. Para corregirlo, se deben agregar las validaciones correspondientes y diferenciar una lista NULL de una estructura válida que simplemente no tiene campos. Aunque el harness pasó las seis pruebas, estos casos adicionales también son importantes para que el programa sea más seguro.

---

**Firma digital:** josue6martin

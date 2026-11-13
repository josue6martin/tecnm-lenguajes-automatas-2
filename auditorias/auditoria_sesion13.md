
# Bitácora de Requerimientos y Auditoría IA — Sesión 13

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Pila de tablas de símbolos (Stack Scopes).

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar el código manualmente, tuve que tener cuidado con la forma en que se relacionan los ámbitos mediante el apuntador padre. Una de las dificultades fue lograr que la búsqueda comenzara en el ámbito actual y continuara hacia los ámbitos anteriores sin confundir las variables que tienen el mismo nombre. También tuve que considerar el límite máximo de 8 nombres por ámbito y evitar accesos fuera del arreglo.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Implementar una pila de ámbitos enlazados que permita declarar identificadores en el ámbito actual y realizar búsquedas jerárquicas, respetando el ocultamiento de variables (Shadowing). |
| **Output (O)** | Código modular en C que implemente `declarar()` y `buscar()`, respetando las firmas de `api_sesion13.h`, además de su análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en sistemas
y compiladores.

Necesito implementar una pila de ámbitos enlazados en lenguaje C
para la materia Lenguajes y Autómatas II.

Utiliza C99/C11 y respeta las siguientes condiciones:

1. Implementa las funciones declarar() y buscar().
2. declarar() debe registrar los nombres únicamente en el
   ámbito actual.
3. No debe permitir nombres duplicados dentro del mismo ámbito.
4. Debe permitir que un mismo nombre exista en diferentes
   ámbitos para respetar el Shadowing.
5. buscar() debe comenzar en el ámbito actual y continuar
   recorriendo los ámbitos padre hasta encontrar el nombre.
6. Si no encuentra el identificador debe devolver NULL.
7. Considera el límite AMBITO_MAX de 8 elementos.
8. Valida los apuntadores NULL y los nombres vacíos.
9. Respeta exactamente las firmas declaradas en api_sesion13.h.
10. No incluyas main(), ya que el harness proporciona esa función.
11. Evita errores de memoria y advertencias de compilación.

Entrega únicamente el código modular en C y explica
la complejidad temporal y espacial de las funciones.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion13.h"
#include <string.h>

int declarar(Ambito *ambito, const char *nombre) {
    if (!ambito || !nombre || *nombre == '\0') {
        return -1;
    }

    if (ambito->total < 0 || ambito->total > AMBITO_MAX) {
        return -1;
    }

    if (ambito->total >= AMBITO_MAX) {
        return -1;
    }

    for (int i = 0; i < ambito->total; ++i) {
        if (ambito->nombres[i] != NULL &&
            strcmp(ambito->nombres[i], nombre) == 0) {
            return 0;
        }
    }

    ambito->nombres[ambito->total++] = nombre;
    return 1;
}

const char *buscar(const Ambito *ambito, const char *nombre) {
    if (!ambito || !nombre || *nombre == '\0') {
        return NULL;
    }

    const Ambito *actual = ambito;

    while (actual != NULL) {
        if (actual->total < 0 || actual->total > AMBITO_MAX) {
            return NULL;
        }

        for (int i = 0; i < actual->total; ++i) {
            if (actual->nombres[i] != NULL &&
                strcmp(actual->nombres[i], nombre) == 0) {
                return actual->nombre;
            }
        }

        actual = actual->padre;
    }

    return NULL;
}
```

**Análisis de complejidad:**

- `declarar()`: O(n), porque recorre los nombres del ámbito actual para comprobar duplicados.
- `buscar()`: O(d × n), donde d es la cantidad de ámbitos y n la cantidad máxima de nombres por ámbito.
- Espacio adicional: O(1), porque no se reserva memoria dinámica.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Búsqueda jerárquica de identificadores.

### Salida del harness

```text
=== AUDITORIA SESION 13 - Pila de ambitos y busqueda jerarquica ===
Caso                               esperado:              obtenido:              resultado
[ 1] declarar(&g, "x")                  esperado: 1            obtenido: 1            OK
[ 2] buscar en el mismo ambito          esperado: global       obtenido: global       OK
[ 3] buscar subiendo al padre           esperado: global       obtenido: global       OK
[ 4] shadowing en el hijo               esperado: bloque       obtenido: bloque       OK
[ 5] tres niveles hasta la raiz         esperado: raiz         obtenido: raiz         OK
[ 6] nombre no declarado                esperado: (nulo)       obtenido: (nulo)       OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion13.md
```

- [x] Caso base y límites superados sin *Crash* en las pruebas ejecutadas.
- [x] No se observaron accesos fuera de límites ni *Memory Leaks* en las pruebas.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código generado por la IA, observé que no contemplaba completamente el caso de que el contador `total` tuviera un valor negativo o mayor al límite permitido de 8 elementos. Esto podría provocar problemas al recorrer el arreglo de nombres. Para solucionarlo agregué una validación que comprueba que el contador esté dentro del rango permitido antes de realizar las operaciones. También agregué una comprobación para evitar comparar apuntadores nulos dentro del arreglo. Después de revisar la implementación y ejecutar el harness, comprobé que los seis casos de prueba se superaron correctamente.

---

**Firma digital:** `josue6martin`

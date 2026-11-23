
# Bitácora de Requerimientos y Auditoría IA — Sesión 18
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arreglos y Verificación de Límites  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353         
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de tamaño de arreglo en memoria  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar la implementación manual me di cuenta de que no era suficiente con calcular el tamaño de los arreglos usando solamente los tipos `int`, `double` y `char`. También era necesario considerar otros tipos como `short`, `float` y punteros. Otro detalle importante fue validar que el número de elementos no fuera negativo y evitar que se aceptaran índices fuera del rango permitido. Para calcular correctamente los bytes, era necesario conocer el tamaño de cada tipo de dato.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Indexación entera no negativa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C y compiladores.

Necesito implementar dos funciones para la Sesión 18 de Lenguajes y Autómatas II.

Contexto:
- Utiliza C99/C11 compatible con GCC.
- Incluye la cabecera api_sesion18.h.
- El código debe compilar con -Wall -Wextra -Werror.
- No incluyas una función main(), porque el harness ya la contiene.

Especificación:
1. Implementa la función:
   long bytes_arreglo(const char *tipo, int elementos);

   Debe calcular el tamaño total en bytes de un arreglo según el tipo de dato y el número de elementos.
   Los tipos válidos son char, short, int, float, double y puntero.
   Utiliza sizeof() para obtener el tamaño de cada tipo.
   Si el tipo no es válido, el puntero tipo es NULL o el número de elementos es negativo, devuelve -1.
   Si el número de elementos es cero, devuelve 0.

2. Implementa la función:
   int indice_en_rango(int indice, int elementos);

   Debe devolver 1 cuando el índice sea válido, es decir, cuando esté entre 0 y elementos - 1.
   Debe devolver 0 cuando el índice sea negativo o sea igual o mayor que el número de elementos.
   También debe rechazar dimensiones no válidas.

Restricciones:
- Respeta exactamente las firmas de las funciones.
- No modifiques api_sesion18.h.
- No agregues main().
- Mantén el código sencillo, ordenado y modular.

Al final, explica brevemente la complejidad temporal y espacial de las funciones.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion18.h"
#include <string.h>

long bytes_arreglo(const char *tipo, int elementos) {
    if (tipo == NULL || elementos < 0) {
        return -1;
    }

    if (elementos == 0) {
        return 0;
    }

    long ancho = -1;

    if (strcmp(tipo, "char") == 0) {
        ancho = sizeof(char);
    } else if (strcmp(tipo, "short") == 0) {
        ancho = sizeof(short);
    } else if (strcmp(tipo, "int") == 0) {
        ancho = sizeof(int);
    } else if (strcmp(tipo, "float") == 0) {
        ancho = sizeof(float);
    } else if (strcmp(tipo, "double") == 0) {
        ancho = sizeof(double);
    } else if (strcmp(tipo, "puntero") == 0) {
        ancho = sizeof(void *);
    } else {
        return -1;
    }

    return ancho * elementos;
}

int indice_en_rango(int indice, int elementos) {
    if (elementos <= 0 || indice < 0) {
        return 0;
    }

    if (indice < elementos) {
        return 1;
    }

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Límites estáticos y dimensiones inválidas

### Salida del harness
```text
=== AUDITORIA SESION 18 - Tamano de arreglo y limites ===
Caso                               esperado:              obtenido:              resultado
[ 1] int[10]                            esperado: 40           obtenido: 40           OK
[ 2] double[3]                          esperado: 24           obtenido: 24           OK
[ 3] int[-4]                            esperado: -1           obtenido: -1           OK
[ 4] indice 0 de 5                      esperado: 1            obtenido: 1            OK
[ 5] indice 5 de 5                      esperado: 0            obtenido: 0            OK
[ 6] indice negativo                    esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion18.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código me fijé en que era importante validar también cuando el puntero `tipo` fuera NULL, ya que de lo contrario se podía producir un error al utilizar strcmp(). También consideré que un arreglo con cero elementos debe devolver cero bytes y que un índice igual al número de elementos no debe aceptarse. Con estas validaciones se evita trabajar con datos incorrectos y se respetan los límites del arreglo. Después de realizar las pruebas, tanto el código manual como el generado por IA obtuvieron 6 de 6 casos correctos.

---

**Firma digital:** josue6martin

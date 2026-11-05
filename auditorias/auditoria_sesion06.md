
# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353        
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Chequeo de compatibilidad de tipos  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Durante la implementación manual, una de las principales dificultades que encontré fue comprender cómo funciona la jerarquía de tipos de datos y determinar cuáles asignaciones son compatibles. Tuve que identificar que un tipo de menor rango puede asignarse a uno de mayor rango, pero no al contrario, ya que podría existir pérdida de información. También tuve que considerar el tipo `TIPO_ERROR` para evitar que se aceptaran asignaciones inválidas. Para solucionarlo, utilicé condiciones `if`, comparaciones entre los valores del `enum Tipo` y una función para identificar el nombre de cada tipo. Con esto logré implementar una verificación sencilla de compatibilidad de tipos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC en Google Colab, utilizando bibliotecas estándar y compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores, especializado en análisis semántico y sistemas de tipos primitivos. |
| **Especificación (E)** | Implementar una matriz de compatibilidad estática basada en la jerarquía `CHAR < INT < FLOAT`. La función `compatible(destino, origen)` debe devolver 1 para asignaciones permitidas y 0 para asignaciones incompatibles. `TIPO_ERROR` nunca debe ser compatible. La función `nombre_tipo()` debe devolver el nombre correspondiente de cada tipo. |
| **Output (O)** | Solo código modular en C, sin `main()`, compatible con `api_sesion06.h` y el test harness. Complejidad temporal O(1) y espacial O(1). |

### 2.2 Prompt Estructurado

```text
Contexto (C):
Estoy realizando la Sesión 06 de Lenguajes y
Autómatas II (SCD-1016), utilizando C99 y GCC
en Google Colab.

El código debe compilar correctamente utilizando:
-Wall -Wextra -Werror

Se deben utilizar únicamente bibliotecas estándar
del lenguaje C.

Rol (R):
Actúa como un ingeniero de software especializado
en compiladores, análisis semántico y sistemas
de tipos primitivos.

Especificación (E):
Implementa las siguientes funciones respetando
exactamente sus firmas:

int compatible(Tipo destino, Tipo origen);
const char *nombre_tipo(Tipo t);

Requisitos:
1. Utilizar el enum Tipo definido en
   api_sesion06.h.
2. Considerar los siguientes tipos:
   TIPO_ERROR = 0
   TIPO_CHAR = 1
   TIPO_INT = 2
   TIPO_FLOAT = 3
3. Respetar la jerarquía:
   CHAR < INT < FLOAT.
4. Aceptar asignaciones cuando el destino
   tenga igual o mayor rango que el origen.
5. Rechazar asignaciones que puedan provocar
   pérdida de información.
6. TIPO_ERROR nunca debe ser compatible.
7. Implementar una matriz estática de
   compatibilidad de tipos.
8. Validar los valores del enum para evitar
   accesos fuera de los límites de la matriz.
9. compatible() debe devolver 1 si la
   asignación es válida y 0 si no lo es.
10. nombre_tipo() debe devolver:
    "char", "int", "float" o "error".
11. No modificar las firmas del contrato.
12. No incluir la función main().

Casos esperados:
- compatible(INT,INT) = 1
- compatible(FLOAT,INT) = 1
- compatible(INT,FLOAT) = 0
- compatible(INT,ERROR) = 0
- nombre_tipo(INT) = "int"
- nombre_tipo(ERROR) = "error"

Output (O):
Devuelve el código fuente en C para
ia_sesion06_c.c, compatible con
api_sesion06.h y harness_sesion06_c.c.

Incluye un análisis breve de complejidad:
Tiempo O(1) y memoria O(1).
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion06_c.c - Fase 2: El Cientifico
 *
 * Tabla de compatibilidad de tipos.
 *
 * Implementacion mediante una matriz estatica
 * que permite verificar asignaciones.
 */

#include "api_sesion06.h"
#include <stddef.h>

/*
 * Analisis de limites y robustez:
 * - Matriz estatica de compatibilidad en O(1).
 * - Validacion de indices fuera de rango.
 * - TIPO_ERROR siempre es incompatible.
 */

int compatible(Tipo destino, Tipo origen) {

    // Validacion de seguridad para evitar
    // accesos fuera de los limites del enum.
    if (destino < TIPO_ERROR || destino > TIPO_FLOAT ||
        origen < TIPO_ERROR || origen > TIPO_FLOAT) {
        return 0;
    }

    // Matriz de compatibilidad.
    // Filas: Destino.
    // Columnas: Origen.

    static const int matriz_compatibilidad[4][4] = {

        // ERR, CHR, INT, FLT (Origen)

        { 0, 0, 0, 0 }, // TIPO_ERROR
        { 0, 1, 0, 0 }, // TIPO_CHAR
        { 0, 1, 1, 0 }, // TIPO_INT
        { 0, 1, 1, 1 }  // TIPO_FLOAT
    };

    return matriz_compatibilidad[destino][origen];
}

/*
 * Funcion para obtener el nombre
 * correspondiente a cada tipo primitivo.
 */

const char *nombre_tipo(Tipo t) {

    switch (t) {

        case TIPO_CHAR:
            return "char";

        case TIPO_INT:
            return "int";

        case TIPO_FLOAT:
            return "float";

        case TIPO_ERROR:
        default:
            return "error";
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Rechazo de asignaciones incompatibles

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks* en la implementación de IA para los tipos validados.
* [x] La complejidad teórica coincide con la implementada.

**Resultados obtenidos en Google Colab:**

```text
=== AUDITORIA SESION 06 - Tabla de compatibilidad de tipos ===
Caso                               esperado:              obtenido:              resultado
[ 1] compatible(INT,INT)                esperado: 1            obtenido: 1            OK
[ 2] compatible(FLOAT,INT)              esperado: 1            obtenido: 1            OK
[ 3] compatible(INT,FLOAT)              esperado: 0            obtenido: 0            OK
[ 4] compatible(INT,ERROR)              esperado: 0            obtenido: 0            OK
[ 5] nombre_tipo(INT)                   esperado: int          obtenido: int          OK
[ 6] nombre_tipo(ERROR)                 esperado: error        obtenido: error        OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion06.md
```

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código generado por la IA, observé que cumplía correctamente con los requisitos de compatibilidad de tipos y utilizaba una matriz estática para determinar cuáles asignaciones estaban permitidas. Sin embargo, identifiqué que el harness no incluía pruebas con valores fuera del rango definido en `enum Tipo`, por ejemplo, un valor negativo o uno mayor que `TIPO_FLOAT`. La IA sí contempló este problema mediante condiciones que validan los límites antes de acceder a la matriz, evitando posibles accesos fuera de rango. Para comprobar completamente esta protección, sería necesario agregar pruebas adicionales con valores inválidos. También observé que mi implementación manual utilizaba comparaciones directas entre los valores del enum, mientras que la IA empleaba una matriz de compatibilidad. Finalmente, ejecuté el test harness en Google Colab y comprobé que ambas implementaciones superaron las 6 pruebas establecidas. Esta actividad me permitió comprender mejor cómo los compiladores verifican la compatibilidad de tipos y cómo pueden prevenir asignaciones que provoquen pérdida de información.

---

**Firma digital:** Josue Efrain

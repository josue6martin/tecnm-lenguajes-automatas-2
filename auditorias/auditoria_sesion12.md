
# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de registro de símbolos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar esta actividad, lo que más se me complicó fue entender cómo calcular correctamente el espacio que necesita cada tipo de dato y en qué posición debe comenzar la siguiente variable. Tuve que considerar que no todos los tipos ocupan la misma cantidad de bytes y que algunos necesitan que su posición esté alineada. También fue importante validar que el registro existiera y que el cursor no fuera negativo para evitar errores al realizar los cálculos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Implementar funciones para obtener el tamaño de cada tipo de dato y calcular el desplazamiento de los símbolos respetando la alineación de memoria. Validar registros nulos, cursores negativos y tipos desconocidos. |
| **Output (O)** | Código modular en C, respetando las funciones declaradas en `api_sesion12.h`, sin `main()` y con análisis de complejidad. |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy trabajando en la materia Lenguajes y Autómatas II y necesito
implementar un sistema para calcular los atributos de los símbolos
y asignar sus posiciones en memoria utilizando C.

Rol:
Actúa como un ingeniero de software especializado en sistemas,
estructuras de datos y desarrollo de compiladores.

Especificación:
Implementa las siguientes funciones:

int ancho_de_tipo(const char *tipo);
int asignar_offset(Simbolo *simbolo, int cursor);

La función ancho_de_tipo debe regresar los siguientes tamaños:
- char = 1 byte.
- short = 2 bytes.
- int = 4 bytes.
- float = 4 bytes.
- double = 8 bytes.
- puntero = 8 bytes.

Si el tipo es NULL o desconocido, debe regresar -1.

La función asignar_offset debe:
- Validar que el registro no sea NULL.
- Rechazar cursores negativos.
- Obtener el tamaño mediante ancho_de_tipo.
- Calcular el siguiente offset respetando la alineación
  natural del tipo.
- Guardar el offset alineado dentro del registro.
- Devolver el cursor correspondiente al siguiente espacio
  disponible en memoria.
- Regresar -1 cuando los argumentos sean inválidos.
- Respetar las estructuras y firmas de api_sesion12.h.
- No incluir la función main().

Output:
Entrega únicamente código modular en C, compatible con
gcc -std=c11 -Wall -Wextra -Werror, y explica brevemente
la complejidad temporal de las funciones.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion12.h"
#include <string.h>

int ancho_de_tipo(const char *tipo) {
    if (!tipo) return -1;

    if (strcmp(tipo, "char") == 0) return 1;
    if (strcmp(tipo, "short") == 0) return 2;
    if (strcmp(tipo, "int") == 0) return 4;
    if (strcmp(tipo, "float") == 0) return 4;
    if (strcmp(tipo, "double") == 0) return 8;
    if (strcmp(tipo, "puntero") == 0) return 8;

    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    if (!simbolo || cursor < 0) return -1;

    int ancho = ancho_de_tipo(simbolo->tipo);
    if (ancho == -1) return -1;

    // Formula eficiente de alineacion:
    // Solo funciona si el ancho es potencia de 2 (1, 2, 4, 8).
    int offset_alineado = (cursor + ancho - 1) & ~(ancho - 1);

    simbolo->offset = offset_alineado;
    return offset_alineado + ancho;
}
```

**Análisis de complejidad:**

- `ancho_de_tipo()`: O(1), porque compara el tipo con una cantidad fija de cadenas.
- `asignar_offset()`: O(1), porque realiza operaciones aritméticas y de bits para calcular la alineación.
- Espacio adicional: O(1), porque utiliza únicamente variables locales y no reserva memoria dinámica.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Alineación de tipos y tamaños en bytes.

### Salida del harness
```text
=== AUDITORIA SESION 12 - Registro de simbolo y offsets alineados ===
Caso                               esperado:              obtenido:              resultado
[ 1] ancho_de_tipo("int")               esperado: 4            obtenido: 4            OK
[ 2] ancho_de_tipo("double")            esperado: 8            obtenido: 8            OK
[ 3] int en cursor 0                    esperado: 4            obtenido: 4            OK
[ 4] char e int: offset de b            esperado: 4            obtenido: 4            OK
[ 5] double desde cursor 4              esperado: 8            obtenido: 8            OK
[ 6] registro NULL                      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion12.md
```

- [x] Caso base y límites probados superados sin Crash.
- [x] No se observaron accesos fuera de límites ni uso de memoria dinámica que pudiera generar Memory Leaks.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código me di cuenta de que la IA no consideró qué podría pasar si se recibe un cursor demasiado grande, cercano al límite máximo de un entero. En ese caso, la operación utilizada para calcular el offset podría provocar un desbordamiento. Para atender este problema, se debe validar que exista espacio suficiente antes de realizar la suma, utilizando `INT_MAX` de la biblioteca `limits.h`. Así se evita continuar con un cálculo que pueda superar el límite permitido. Este caso no aparece en las seis pruebas del harness, por lo que fue necesario identificarlo revisando las operaciones del código.

---

**Firma digital:** `josue6martin`

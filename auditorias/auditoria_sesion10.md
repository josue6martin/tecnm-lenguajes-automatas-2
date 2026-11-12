
# Bitácora de Requerimientos y Auditoría IA — Sesión 10

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness).

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto: Integración AST y Tipado Estático*

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar el evaluador manualmente, se me complicó un poco organizar las funciones para reconocer los números y los operadores sin perder el orden de las operaciones. También tuve que cuidar que los paréntesis se interpretaran correctamente y que la multiplicación y división tuvieran prioridad sobre la suma y la resta. Otro detalle fue identificar cuándo una expresión estaba incompleta o tenía una división entre cero, para poder asignar el código de error correspondiente.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar un mini-evaluador de expresiones que reconozca números enteros, operadores aritméticos y paréntesis, respetando la precedencia y reportando errores de sintaxis y división entre cero. |
| **Output (O)** | Código modular en C que respete la firma `evaluar_expresion()` y permita evaluar expresiones, junto con un análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en C y compiladores.

CONTEXTO:
Estoy trabajando en la Sesión 10 de Lenguajes y Autómatas II.
Necesito desarrollar un mini-evaluador de expresiones aritméticas
que pueda leer una expresión desde una cadena de texto y calcular
su resultado.

ROL:
Actúa como un programador experto en C99, análisis sintáctico
y desarrollo de compiladores.

ESPECIFICACIÓN:
Implementa la siguiente función respetando exactamente su firma:

long evaluar_expresion(const char *fuente, int *error);

El programa debe cumplir con los siguientes requisitos:

- Reconocer números enteros.
- Aceptar los operadores +, -, * y /.
- Respetar la precedencia de los operadores.
- Permitir el uso de paréntesis.
- Ignorar espacios en blanco.
- Detectar división entre cero y asignar error = 1.
- Detectar errores de sintaxis y asignar error = 2.
- Manejar correctamente una fuente NULL.
- No permitir variables ni menos unario.
- No modificar la firma de la función.
- No incluir main().
- Utilizar código modular y compilar con:
  -Wall -Wextra -Werror

OUTPUT:
Devuelve el código completo en C, organizado mediante
funciones auxiliares para analizar expresiones, términos
y operaciones primarias.

Incluye una breve explicación de la complejidad temporal
y espacial del algoritmo.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion10.h"
#include <ctype.h>
#include <stddef.h>

typedef struct {
    const char *cursor;
    int error_code;
} ParserState;

static void skip_whitespace(ParserState *p) {
    while (*p->cursor == ' ' || *p->cursor == '\t' ||
           *p->cursor == '\r' || *p->cursor == '\n') {
        p->cursor++;
    }
}

static long parse_expr(ParserState *p);
static long parse_term(ParserState *p);
static long parse_primary(ParserState *p);

static long parse_primary(ParserState *p) {
    skip_whitespace(p);
    if (p->error_code) return 0;

    if (*p->cursor == '(') {
        p->cursor++;
        long val = parse_expr(p);

        if (p->error_code) return 0;

        skip_whitespace(p);

        if (*p->cursor == ')') {
            p->cursor++;
            return val;
        } else {
            p->error_code = 2;
            return 0;
        }
    }

    if (isdigit((unsigned char)*p->cursor)) {
        long val = 0;

        while (isdigit((unsigned char)*p->cursor)) {
            val = val * 10 + (*p->cursor - '0');
            p->cursor++;
        }

        return val;
    }

    p->error_code = 2;
    return 0;
}

static long parse_term(ParserState *p) {
    long val = parse_primary(p);

    if (p->error_code) return 0;

    while (1) {
        skip_whitespace(p);
        char op = *p->cursor;

        if (op == '*' || op == '/') {
            p->cursor++;
            long right = parse_primary(p);

            if (p->error_code) return 0;

            if (op == '*') {
                val *= right;
            } else {
                if (right == 0) {
                    p->error_code = 1;
                    return 0;
                }
                val /= right;
            }
        } else {
            break;
        }
    }

    return val;
}

static long parse_expr(ParserState *p) {
    long val = parse_term(p);

    if (p->error_code) return 0;

    while (1) {
        skip_whitespace(p);
        char op = *p->cursor;

        if (op == '+' || op == '-') {
            p->cursor++;
            long right = parse_term(p);

            if (p->error_code) return 0;

            if (op == '+') {
                val += right;
            } else {
                val -= right;
            }
        } else {
            break;
        }
    }

    return val;
}

long evaluar_expresion(const char *fuente, int *error) {
    int dummy_error = 0;

    if (error == NULL) {
        error = &dummy_error;
    }

    *error = 0;

    if (fuente == NULL) {
        *error = 2;
        return 0;
    }

    ParserState p = { fuente, 0 };
    long result = parse_expr(&p);

    if (p.error_code == 0) {
        skip_whitespace(&p);

        if (*p.cursor != '\0') {
            p.error_code = 2;
        }
    }

    *error = p.error_code;

    return p.error_code != 0 ? 0 : result;
}
```

**Análisis de complejidad:**

- Complejidad temporal: O(n), porque se recorre la expresión para analizar sus números y operadores.
- Complejidad espacial: O(d), donde d representa la profundidad de los paréntesis, debido a las llamadas recursivas.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Precedencia, paréntesis, división entre cero y sintaxis inválida.

### Salida del harness

```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado
[ 1] "2+3*4"                            esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                          esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1                  esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                    esperado: 2            obtenido: 2            OK
[ 5] "  7 "                             esperado: 7            obtenido: 7            OK
[ 6] "5-10"                             esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
```

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks* en las pruebas realizadas.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código de la IA, identifiqué que no contempla el desbordamiento de valores cuando se introducen números demasiado grandes o cuando una operación supera el límite del tipo `long`. Aunque las seis pruebas del harness se ejecutaron correctamente, ninguna comprueba ese tipo de situación. Lo detecté revisando las operaciones que realizan las funciones de suma, resta y multiplicación. Por ello, identifiqué esta limitación y la documenté para tenerla en cuenta al evaluar expresiones con valores más grandes. La implementación entregada conserva los resultados esperados de las pruebas establecidas por el profesor.

---

**Firma digital:** `josue6martin`

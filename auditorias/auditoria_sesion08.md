
# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353       
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Reglas de casteo implícito  

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Durante la implementación manual, una de las principales dificultades que encontré fue comprender cómo se realiza la promoción de tipos cuando se combinan diferentes datos, especialmente entre `char`, `int` y `float`. También tuve que identificar cuándo una conversión puede provocar pérdida de información, como al convertir un `float` a `int` o un `int` a `char`. Para solucionarlo, utilicé condiciones `if` y comparaciones entre los valores del `enum Tipo`, respetando la jerarquía `CHAR < INT < FLOAT`. Además, consideré el tipo `TIPO_ERROR` para evitar que una operación inválida produjera un tipo aparentemente correcto. Con esto logré implementar las reglas de promoción y detectar las conversiones que pueden perder información.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC en Google Colab, utilizando bibliotecas estándar y compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores, especializado en inferencia de tipos, conversiones implícitas y análisis semántico. |
| **Especificación (E)** | Implementar `promover(Tipo a, Tipo b)`, `hay_perdida(Tipo destino, Tipo origen)` y `nombre_tipo(Tipo t)`. Aplicar la jerarquía `CHAR < INT < FLOAT`, devolver el tipo de mayor rango, conservar `TIPO_ERROR` y detectar conversiones hacia tipos más angostos. |
| **Output (O)** | Solo código modular en C, sin `main()`, compatible con `api_sesion08.h` y el test harness. Complejidad temporal O(1) y espacial O(1). |

### 2.2 Prompt Estructurado

```text
Contexto (C):
Estoy realizando la Sesión 08 de Lenguajes y
Autómatas II (SCD-1016), utilizando C99 y GCC
en Google Colab.

El código debe compilar correctamente con:
-Wall -Wextra -Werror

Se deben utilizar únicamente bibliotecas estándar
del lenguaje C.

Rol (R):
Actúa como un ingeniero de software especializado
en compiladores, análisis semántico, inferencia
de tipos y conversiones implícitas.

Especificación (E):
Implementa las siguientes funciones respetando
exactamente sus firmas:

Tipo promover(Tipo a, Tipo b);
int hay_perdida(Tipo destino, Tipo origen);
const char *nombre_tipo(Tipo t);

Requisitos:
1. Utilizar el enum Tipo definido en
   api_sesion08.h.
2. Considerar los tipos:
   TIPO_ERROR = 0
   TIPO_CHAR = 1
   TIPO_INT = 2
   TIPO_FLOAT = 3
3. Respetar la jerarquía:
   CHAR < INT < FLOAT.
4. promover() debe devolver el tipo de mayor
   rango entre los dos operandos.
5. Si alguno de los operandos es TIPO_ERROR,
   devolver TIPO_ERROR.
6. hay_perdida() debe devolver 1 cuando
   el destino es más angosto que el origen.
7. Si no existe pérdida, devolver 0.
8. nombre_tipo() debe devolver:
   "char", "int", "float" o "error".
9. Utilizar únicamente bibliotecas estándar.
10. No modificar las firmas del contrato.
11. No incluir la función main().

Casos esperados:
- promover(CHAR,INT) = int
- promover(INT,FLOAT) = float
- promover(FLOAT,FLOAT) = float
- promover(ERROR,INT) = error
- hay_perdida(CHAR,INT) = 1
- hay_perdida(INT,CHAR) = 0

Output (O):
Devuelve el código fuente en C para
ia_sesion08_c.c, compatible con
api_sesion08.h y harness_sesion08_c.c.

Incluye un análisis breve de complejidad:
Tiempo O(1) y memoria O(1).
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion08_c.c - Fase 2: El Cientifico
 *
 * Reglas de promocion y perdida de tipos.
 *
 * Implementacion de funciones para determinar
 * el tipo comun de dos operandos y detectar
 * posibles perdidas de informacion.
 */

#include "api_sesion08.h"

/*
 * Determina el tipo de mayor rango.
 * Si alguno es TIPO_ERROR, devuelve error.
 */
Tipo promover(Tipo a, Tipo b) {
    if (a == TIPO_ERROR || b == TIPO_ERROR) {
        return TIPO_ERROR;
    }

    return (a > b) ? a : b;
}

/*
 * Detecta si una conversion puede
 * provocar perdida de informacion.
 *
 * Devuelve 1 si el destino es mas
 * angosto que el origen y 0 si no.
 */
int hay_perdida(Tipo destino, Tipo origen) {
    return (destino < origen) ? 1 : 0;
}

/*
 * Devuelve el nombre correspondiente
 * a cada tipo primitivo.
 */
const char *nombre_tipo(Tipo t) {
    switch (t) {
        case TIPO_CHAR:
            return "char";

        case TIPO_INT:
            return "int";

        case TIPO_FLOAT:
            return "float";

        default:
            return "error";
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Mezcla de tipos enteros y flotantes

* [x] Caso base y límites superados sin *Crash* en las pruebas ejecutadas.
* [x] No se realizan accesos a arreglos ni asignaciones dinámicas de memoria en estas funciones.
* [x] La complejidad teórica coincide con la implementada.

**Resultados obtenidos en Google Colab:**

```text
=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===
Caso                               esperado:              obtenido:              resultado
[ 1] promover(CHAR,INT)                 esperado: int          obtenido: int          OK
[ 2] promover(INT,FLOAT)                esperado: float        obtenido: float        OK
[ 3] promover(FLOAT,FLOAT)              esperado: float        obtenido: float        OK
[ 4] promover(ERROR,INT)                esperado: error        obtenido: error        OK
[ 5] hay_perdida(CHAR,INT)              esperado: 1            obtenido: 1            OK
[ 6] hay_perdida(INT,CHAR)              esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion08.md
```

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código generado por la IA, observé que cumplía correctamente con las reglas de promoción de tipos y detectaba las conversiones hacia tipos más angostos. Sin embargo, identifiqué que no contemplaba completamente los valores inválidos del `enum Tipo`, ya que la función `promover()` compara directamente los valores recibidos y podría devolver un tipo que no está definido en el contrato. También observé que `hay_perdida()` considera únicamente el rango numérico de los tipos, por lo que podría marcar una pérdida cuando interviene `TIPO_ERROR`, aunque este representa un estado inválido y no un tipo numérico. Para solucionar estas limitaciones, sería necesario validar primero que ambos tipos estén dentro del rango permitido y establecer un comportamiento específico para `TIPO_ERROR`. Finalmente, ejecuté el test harness en Google Colab y comprobé que tanto mi implementación manual como la generada por IA superaron las 6 pruebas establecidas. Esta actividad me ayudó a comprender mejor cómo los compiladores determinan el tipo resultante de una expresión y cómo identifican las conversiones que pueden provocar pérdida de información.

---

**Firma digital:** Josue Martin

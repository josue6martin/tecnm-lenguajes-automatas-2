
# Bitácora de Requerimientos y Auditoría IA — Sesión 25
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Bucles while y for  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353                  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de ciclo en instrucciones TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar las funciones manualmente, se me complicó organizar correctamente las etiquetas de inicio y fin de cada ciclo, porque tenía que asegurarme de que los saltos condicionales y los GOTO apuntaran al lugar correcto. También tuve que cuidar la posición de la actualización del iterador en el ciclo for, ya que debe colocarse antes del salto de regreso. Otro detalle fue controlar el arreglo de líneas mediante el contador `n` y el límite `max`, para evitar escribir fuera de los espacios disponibles. 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. La implementación debe incluir `api_sesion25.h` y compilar con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores, con experiencia en generación de código de tres direcciones (TAC). |
| **Especificación (E)** | Implementar `emitir_while()` y `emitir_for()` respetando exactamente las firmas del contrato. El while debe generar cuatro instrucciones: etiqueta de inicio, salto condicional, GOTO de regreso y etiqueta de fin. El for debe generar inicialización opcional, etiqueta de inicio, salto condicional, actualización opcional, GOTO de regreso y etiqueta de fin. Cada ciclo debe consumir dos etiquetas únicas, incrementar el contador de líneas solamente cuando se emita una instrucción y respetar el límite `max`. Si `inicio` o `post` son NULL, no deben emitirse esas instrucciones. |
| **Output (O)** | Código C modular, sin función `main()`, que implemente las dos funciones solicitadas, respete los límites del arreglo y presente un análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

```text
Necesito que actúes como un ingeniero de software especializado en compiladores y generación de código TAC.

CONTEXTO:
Estoy trabajando en la Sesión 25 de Lenguajes y Autómatas II (SCD-1016), utilizando C y GCC. La implementación debe incluir api_sesion25.h y compilar con las opciones -std=c11 -Wall -Wextra -Werror.

ROL:
Actúa como un programador experto en C y generación de código de tres direcciones.

ESPECIFICACIÓN:
Implementa exactamente las siguientes funciones:

void emitir_while(char lineas[][64], int max, int *n, const char *cond, int *etiquetas);

void emitir_for(char lineas[][64], int max, int *n, const char *inicio, const char *cond, const char *post, int *etiquetas);

Para emitir_while:
1. Genera una etiqueta de inicio utilizando el contador de etiquetas.
2. Genera una instrucción IFFALSE con la condición y un salto hacia la etiqueta final.
3. Genera un GOTO que regrese a la etiqueta de inicio.
4. Genera la etiqueta final.
5. Cada ciclo debe consumir dos etiquetas únicas.

Para emitir_for:
1. Emite la inicialización si inicio no es NULL.
2. Emite la etiqueta de inicio.
3. Emite la instrucción IFFALSE con la condición y el salto hacia la etiqueta final.
4. Emite la actualización del iterador si post no es NULL.
5. Emite el GOTO hacia la etiqueta de inicio.
6. Emite la etiqueta final.
7. Cada ciclo debe consumir dos etiquetas únicas.

RESTRICCIONES:
- Respeta exactamente las firmas de las funciones.
- No agregues una función main().
- No utilices memoria dinámica ni variables globales.
- Verifica que n sea menor que max antes de escribir en el arreglo.
- Utiliza snprintf para limitar la escritura a 64 caracteres por línea.
- Comprueba los punteros antes de utilizarlos.
- Si inicio o post son NULL, omite esa instrucción y no incrementes n.
- Incrementa n únicamente cuando se haya agregado una línea.
- Mantén las etiquetas únicas para evitar colisiones entre ciclos.
- El código debe compilar sin advertencias con -Wall -Wextra -Werror.

OUTPUT:
Devuelve el código completo de ia_sesion25_c.c, sin main(), y agrega una explicación breve de la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion25_c.c - Fase 2: El Cientifico
 *
 * Implementación robusta y limpia respetando el contrato estricto de api_sesion25.h.
 */

#include "api_sesion25.h"
#include <stdio.h>
#include <string.h>

void emitir_while(char lineas[][64], int max, int *n, const char *cond, int *etiquetas) {
    if (!lineas || !n || !cond || !etiquetas) return;

    int lbl_start = (*etiquetas)++;
    int lbl_end = (*etiquetas)++;

    // 1. L_start:
    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", lbl_start);
        (*n)++;
    }

    // 2. IFFALSE cond GOTO L_end
    if (*n < max) {
        snprintf(lineas[*n], 64, "IFFALSE %s GOTO L%d", cond, lbl_end);
        (*n)++;
    }

    // 3. GOTO L_start
    if (*n < max) {
        snprintf(lineas[*n], 64, "GOTO L%d", lbl_start);
        (*n)++;
    }

    // 4. L_end:
    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", lbl_end);
        (*n)++;
    }
}

void emitir_for(char lineas[][64], int max, int *n, const char *inicio, const char *cond, const char *post, int *etiquetas) {
    if (!lineas || !n || !cond || !etiquetas) return;

    int lbl_start = (*etiquetas)++;
    int lbl_end = (*etiquetas)++;

    // 1. Inicialización (si existe)
    if (inicio && *n < max) {
        snprintf(lineas[*n], 64, "%s", inicio);
        (*n)++;
    }

    // 2. L_start:
    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", lbl_start);
        (*n)++;
    }

    // 3. IFFALSE cond GOTO L_end
    if (*n < max) {
        snprintf(lineas[*n], 64, "IFFALSE %s GOTO L%d", cond, lbl_end);
        (*n)++;
    }

    // 4. Post-incremento (si existe)
    if (post && *n < max) {
        snprintf(lineas[*n], 64, "%s", post);
        (*n)++;
    }

    // 5. GOTO L_start
    if (*n < max) {
        snprintf(lineas[*n], 64, "GOTO L%d", lbl_start);
        (*n)++;
    }

    // 6. L_end:
    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", lbl_end);
        (*n)++;
    }
}
```

**Análisis de complejidad:**

- **Tiempo:** O(1) por función, considerando que se genera una cantidad fija de instrucciones y se utilizan operaciones de formato para cada línea.
- **Espacio auxiliar:** O(1), porque se trabaja directamente sobre el arreglo recibido y no se reserva memoria dinámica.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Bucles anidados y condiciones de parada.

### Salida del harness

```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion25.md
```

- [x] Caso base y límites superados sin *Crash* en las pruebas ejecutadas.
- [x] No se observaron accesos fuera de límites ni *Memory Leaks* en los seis casos del harness.
- [ ] La complejidad teórica coincide completamente con la implementada: el formato de cadenas depende de la longitud de los textos y no solamente de la cantidad fija de instrucciones.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código de la IA encontré que, aunque utiliza `snprintf` para limitar la escritura a 64 caracteres, no valida que `n` sea mayor o igual a cero ni que sea menor que `max` antes de acceder al arreglo. También puede consumir las dos etiquetas aunque el arreglo ya no tenga espacio suficiente para guardar las instrucciones. Además, si las cadenas son demasiado largas, `snprintf` las recorta y el código no comprueba si la instrucción quedó completa.
>
> Las seis pruebas del harness pasaron correctamente, pero no comprueban esos casos extremos. Por eso identifiqué que sería necesario validar los límites de `n` y revisar el resultado de `snprintf` para detectar posibles truncamientos. Esta mejora queda como observación de auditoría, ya que no se muestra una segunda ejecución del harness con esas correcciones.

---

**Firma digital:** josue6martin

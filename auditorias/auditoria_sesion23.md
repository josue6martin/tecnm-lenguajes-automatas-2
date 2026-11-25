
# Bitácora de Requerimientos y Auditoría IA — Sesión 23
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Etiquetas y Saltos Incondicionales  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353      
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Generador de etiquetas e instrucciones goto  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar el código manualmente, tuve que cuidar que las etiquetas se generaran de manera consecutiva, comenzando desde L0, y que el contador aumentara después de generar cada una. También tuve que respetar la sintaxis de las instrucciones, ya que un salto debe escribirse con la palabra GOTO seguida de la etiqueta, mientras que una marca de destino debe llevar dos puntos al final. Otra dificultad fue controlar el tamaño del buffer para evitar escribir fuera de sus límites. Por eso utilicé `snprintf()` y agregué validaciones para los punteros NULL y las capacidades menores o iguales a cero.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 con GCC, utilizando la interfaz definida en `api_sesion23.h`, sin bibliotecas no estándar y compilando con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar `nueva_etiqueta()` para generar etiquetas únicas y consecutivas con formato L0, L1, L2..., utilizando e incrementando el contador. Implementar `emitir_flujo()` para escribir instrucciones GOTO o marcas de destino con el formato correspondiente. Ambas funciones deben respetar la capacidad del buffer y manejar correctamente los punteros NULL. |
| **Output (O)** | Código modular en C, respetando las firmas originales, sin incluir `main()`, con manejo de límites de memoria y análisis de complejidad. |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C y compiladores.

Necesito implementar dos funciones para la Sesión 23 de Lenguajes y Autómatas II,
cuyo objetivo es generar etiquetas y saltos incondicionales para representar
el flujo de control de un programa.

Utiliza C estándar compatible con GCC y respeta exactamente las siguientes firmas:

void nueva_etiqueta(char *dst, int cap, int *contador);

void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto);

Requisitos:

1. La función nueva_etiqueta() debe generar etiquetas con el formato L0, L1,
   L2, etc.
2. Debe utilizar el valor actual del contador y aumentarlo después de generar
   correctamente la etiqueta.
3. La primera etiqueta debe ser L0 cuando el contador inicia en cero.
4. La función emitir_flujo() debe escribir "GOTO etiqueta" cuando es_salto
   sea distinto de cero.
5. Cuando es_salto sea cero, debe escribir la marca de destino con el formato
   "etiqueta:".
6. Si etiqueta es NULL, debe producir una cadena vacía.
7. Ambas funciones deben validar que dst no sea NULL y que cap sea mayor
   que cero.
8. nueva_etiqueta() también debe validar que contador no sea NULL.
9. Utiliza snprintf() para evitar desbordamientos del buffer.
10. No utilices memoria dinámica, variables globales ni incluyas main().
11. Respeta exactamente las firmas declaradas en api_sesion23.h.

El código debe compilar sin advertencias con:

gcc -std=c11 -Wall -Wextra -Werror

Devuelve el código completo para ia_sesion23_c.c, con comentarios
explicativos y una breve explicación de la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion23.h"
#include <stdio.h>
#include <string.h>

/**
 * @brief Genera una etiqueta unica con formato "Lx", usando y luego incrementando el contador.
 *
 * @param dst Buffer de destino donde se escribirá la etiqueta.
 * @param cap Capacidad máxima de caracteres admisibles en dst.
 * @param contador Puntero al entero que lleva la secuencia del etiquetado.
 */
void nueva_etiqueta(char *dst, int cap, int *contador) {
    if (dst && cap > 0 && contador) {
        int escritos = snprintf(dst, (size_t)cap, "L%d", *contador);
        if (escritos >= 0 && escritos < cap) {
            (*contador)++;
        } else {
            /* En caso de desbordamiento, nos aseguramos de terminar en nulo */
            dst[cap - 1] = '\0';
        }
    }
}

/**
 * @brief Emite el flujo correspondiente a un salto incondicional (GOTO) o marca de destino.
 *
 * @param dst Buffer de destino.
 * @param cap Capacidad del buffer dst.
 * @param etiqueta Nombre de la etiqueta involucrada.
 * @param es_salto Si es distinto de cero emite "GOTO etiqueta", si es cero emite "etiqueta:".
 */
void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto) {
    if (!dst || cap <= 0) {
        return;
    }

    /* Caso borde de etiqueta NULL: produce una cadena vacía */
    if (!etiqueta) {
        dst[0] = '\0';
        return;
    }

    int escritos;
    if (es_salto) {
        escritos = snprintf(dst, (size_t)cap, "GOTO %s", etiqueta);
    } else {
        escritos = snprintf(dst, (size_t)cap, "%s:", etiqueta);
    }

    /* Validación estricta del límite del buffer */
    if (escritos < 0 || escritos >= cap) {
        dst[cap - 1] = '\0';
    }
}
```

**Análisis de complejidad:**

- `nueva_etiqueta()`: tiempo O(d), donde d es la cantidad de dígitos del contador, porque debe convertir el número a texto.
- `emitir_flujo()`: tiempo O(n), donde n es la longitud de la etiqueta, porque debe copiarla al buffer.
- Espacio auxiliar: O(1), ya que ambas funciones trabajan directamente sobre el buffer recibido y no reservan memoria dinámica.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Saltos adelante y atrás estructurados.

### Salida del harness

**Ejecución de Artesano:**
```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion23.md
```

**Ejecución de la IA:**
```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion23.md
```

- [x] Caso base y límites probados sin *Crash* en los casos ejecutados.
- [x] No se detectaron accesos fuera de límites ni *Memory Leaks* en las pruebas ejecutadas. El código no utiliza memoria dinámica.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, identifiqué que el harness utiliza buffers suficientemente grandes, por lo que no comprueba qué sucede cuando la capacidad es demasiado pequeña. Aunque la IA utiliza `snprintf()` para evitar escribir fuera del buffer, no verifica en todos los casos si el texto se guardó completamente. En `nueva_etiqueta()`, el contador solamente aumenta cuando la etiqueta se escribió correctamente, lo cual es adecuado, pero cuando el buffer es insuficiente no se informa directamente del problema. En `emitir_flujo()`, el texto puede quedar truncado si no hay espacio suficiente. Para mejorar este comportamiento, sería conveniente agregar pruebas con buffers pequeños y comprobar el valor que devuelve `snprintf()`. También se pueden probar capacidades iguales a cero y punteros NULL. Las seis pruebas proporcionadas sí fueron superadas tanto por la implementación manual como por la de IA, pero no cubren todos esos escenarios.

---
**Firma digital:** josue6martin


# Bitácora de Requerimientos y Auditoría IA — Sesión 26
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** TAC para Llamadas a Funciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353            
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Instrucciones param, call y return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar la implementación manual tuve que cuidar el orden en que se agregan los parámetros, porque primero se debe emitir `PARAM x` y después `PARAM y`. También tuve que controlar la posición del arreglo mediante el contador `n` para no escribir fuera de los espacios disponibles. Otro detalle fue calcular correctamente la cantidad de parámetros que se envían en la instrucción `CALL`. Para el retorno utilicé `snprintf`, pero en la parte de las llamadas utilicé `sprintf`, por lo que identifiqué que debía tener cuidado con el tamaño de los buffers para evitar escribir más información de la que pueden almacenar.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 / GCC, utilizando la interfaz `api_sesion26.h`, sin bibliotecas no estándar y compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software especializado en programación en C, sistemas y compiladores. |
| **Especificación (E)** | Implementar `emitir_llamada()` para generar instrucciones PARAM en el orden de los argumentos recibidos y una instrucción CALL con la cantidad real de parámetros. Si existe una variable de resultado, debe asignarse el resultado de la llamada. Implementar `emitir_return()` para generar RETURN con valor o sin valor. Respetar las firmas de la API, controlar los límites del arreglo y permitir llamadas anidadas. |
| **Output (O)** | Código C modular, sin función `main()`, compatible con el harness, utilizando funciones seguras para manejar cadenas y un análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C y compiladores.

CONTEXTO:
Estoy trabajando en la Sesión 26 de Lenguajes y Autómatas II.
Necesito implementar instrucciones de código de tres direcciones (TAC)
para llamadas a funciones y retornos. El programa se compilará con GCC
utilizando -std=c11 -Wall -Wextra -Werror.

ROL:
Compórtate como un programador experto en C, cuidando la memoria,
los límites de los arreglos y el manejo correcto de cadenas.

ESPECIFICACIÓN:
Implementa exactamente las siguientes funciones, respetando sus firmas:

int emitir_llamada(char lineas[][64], int max, int *n,
                   const char *funcion, const char *a1,
                   const char *a2, const char *resultado);

void emitir_return(char *dst, int cap, const char *valor);

Requisitos:
1. Cada argumento diferente de NULL debe generar una instrucción PARAM.
2. Los parámetros deben emitirse en el orden a1 y después a2.
3. La instrucción CALL debe indicar la cantidad real de parámetros emitidos.
4. Si resultado tiene un valor, generar:
   _t0 = CALL f, 2
5. Si resultado es NULL, generar CALL sin asignación.
6. Si no hay argumentos, generar CALL con cantidad 0.
7. Permitir llamadas anidadas como f(g(x)), conservando el temporal
   de la llamada interna.
8. emitir_return debe generar RETURN valor cuando recibe un valor
   y solamente RETURN cuando recibe NULL.
9. Verificar que dst no sea NULL y que cap sea mayor que cero.
10. Respetar los límites del arreglo y evitar escrituras fuera de rango.
11. Utilizar snprintf para controlar el tamaño de las cadenas.
12. No modificar las firmas de las funciones.
13. No incluir main() porque el harness ya contiene esa función.
14. El código debe compilar sin advertencias con GCC C11.

OUTPUT:
Entrega únicamente el código C completo para ia_sesion26_c.c,
incluyendo la cabecera api_sesion26.h, y después explica brevemente
la complejidad temporal y espacial de cada función.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion26_c.c - Fase 2: El Cientifico
 *
 * Codigo robusto y optimizado generado con asistencia de IA.
 */
#include "api_sesion26.h"
#include <stdio.h>
#include <string.h>

int emitir_llamada(char lineas[][64], int max, int *n, const char *funcion, const char *a1, const char *a2, const char *resultado) {
    int count = 0;
    const char *args[2] = { a1, a2 };

    // Emitir parametros validos de forma secuencial
    for (int i = 0; i < 2; i++) {
        if (args[i] != NULL) {
            if (*n < max) {
                snprintf(lineas[*n], 64, "PARAM %s", args[i]);
                (*n)++;
            }
            count++;
        }
    }

    // Emitir la instruccion CALL correspondiente
    if (*n < max) {
        if (resultado != NULL && strlen(resultado) > 0) {
            snprintf(lineas[*n], 64, "%s = CALL %s, %d", resultado, funcion, count);
        } else {
            snprintf(lineas[*n], 64, "CALL %s, %d", funcion, count);
        }
        (*n)++;
    }

    return count;
}

void emitir_return(char *dst, int cap, const char *valor) {
    if (dst == NULL || cap <= 0) return;

    if (valor != NULL && strlen(valor) > 0) {
        snprintf(dst, cap, "RETURN %s", valor);
    } else {
        snprintf(dst, cap, "RETURN");
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas anidadas f(g(x)), parámetros, CALL sin argumentos y RETURN con valor.

### Salida del harness
```text
=== AUDITORIA SESION 26 - Llamadas, parametros y retorno ===
Caso                               esperado:              obtenido:              resultado
[ 1] PARAM x                            esperado: PARAM x      obtenido: PARAM x      OK
[ 2] PARAM y                            esperado: PARAM y      obtenido: PARAM y      OK
[ 3] CALL con dos argumentos
     esperado: _t0 = CALL f, 2
     obtenido: _t0 = CALL f, 2
     -> OK
[ 4] llamada anidada f(g(x))
     esperado: _t1 = CALL f, 1
     obtenido: _t1 = CALL f, 1
     -> OK
[ 5] CALL sin argumentos
     esperado: _t0 = CALL p, 0
     obtenido: _t0 = CALL p, 0
     -> OK
[ 6] RETURN con valor                   esperado: RETURN _t0   obtenido: RETURN _t0   OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion26.md
```

* [x] Caso base y límites superados sin *Crash* en los seis casos ejecutados.
* [ ] No existen accesos fuera de límites ni *Memory Leaks* en todos los escenarios posibles.
* [ ] La complejidad teórica coincide con la implementada en todos los escenarios.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código de la IA encontré que no valida si `lineas` o `n` son NULL antes de utilizarlos dentro de `emitir_llamada()`. Tampoco comprueba que `max` sea positivo ni que `*n` sea un índice válido antes de acceder al arreglo. Otro detalle es que, cuando ya no hay espacio para guardar una instrucción, la función puede contar un parámetro aunque no lo haya escrito. En el caso de `emitir_return()`, sí se validan `dst` y `cap`, pero no se comprueba si el texto quedó completo cuando el buffer es pequeño. Los seis casos del harness pasaron correctamente, pero no prueban estas situaciones. Por eso identifiqué que se deben agregar validaciones de punteros, capacidad e índices antes de escribir, y comprobar el resultado de `snprintf` para detectar truncamientos. Estas mejoras son recomendaciones y no están comprobadas por las seis pruebas ejecutadas.

---
**Firma digital:** josue6martin

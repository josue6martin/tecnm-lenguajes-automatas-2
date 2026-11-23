
# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Comprobación Semántica de Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Validación de tipo de sentencia return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar esta función manualmente, tuve que tener cuidado al comparar los tipos de datos que devuelve una función con el tipo que realmente espera recibir. También fue necesario revisar que todas las rutas de ejecución terminaran con un return, porque una función puede tener diferentes condiciones y no todas necesariamente devuelven un valor. Otra dificultad fue organizar las validaciones para que primero se revisara si los tipos eran compatibles y después si faltaba alguna ruta por retornar.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Rutas de ejecución con retorno garantizado |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en sistemas y compiladores.

Necesito implementar en lenguaje C la función diagnosticar_retorno() para la Sesión 17 de Lenguajes y Autómatas II.

La función debe respetar exactamente esta firma:

int diagnosticar_retorno(const char *tipo_funcion, const char *tipo_expresion, int rutas_totales, int rutas_con_return);

Requisitos:
1. Validar que tipo_funcion y tipo_expresion no sean NULL.
2. Verificar que rutas_totales sea mayor que cero.
3. Verificar que rutas_con_return no sea negativo ni mayor que rutas_totales.
4. Si la función es void y la expresión devuelta no es void, devolver 1.
5. Si la función no es void y la expresión devuelta es void, devolver 1.
6. Aceptar tipos iguales.
7. Permitir que double acepte una expresión int.
8. No permitir que int acepte una expresión double.
9. Comprobar la compatibilidad de tipos antes de revisar las rutas de retorno.
10. Si los tipos son compatibles, pero rutas_con_return es menor que rutas_totales, devolver 2.
11. Si todo es correcto, devolver 0.
12. Si los parámetros son inválidos o los conteos de rutas son incoherentes, devolver -1.

No cambies la firma ni la interfaz declarada en api_sesion17.h.
No incluyas main(), porque el harness proporciona su propio programa principal.

Entrega código modular compatible con C11 y con la compilación:
gcc -std=c11 -Wall -Wextra -Werror

Incluye un breve análisis de complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion17.h"
#include <string.h>
#include <stdbool.h>

static bool es_tipo_compatible(const char *destino, const char *origen) {
    if (strcmp(destino, origen) == 0) {
        return true;
    }

    /* Regla de ensanchamiento: double acepta int */
    if (strcmp(destino, "double") == 0 && strcmp(origen, "int") == 0) {
        return true;
    }

    return false;
}

int diagnosticar_retorno(const char *tipo_funcion, const char *tipo_expresion, int rutas_totales, int rutas_con_return) {
    /* 1. Validación de precondiciones y parámetros */
    if (!tipo_funcion || !tipo_expresion || rutas_totales <= 0 ||
        rutas_con_return < 0 || rutas_con_return > rutas_totales) {
        return -1;
    }

    /* 2. Comprobación de tipos antes que las rutas */
    bool es_funcion_void = (strcmp(tipo_funcion, "void") == 0);
    bool es_expresion_void = (strcmp(tipo_expresion, "void") == 0);

    if (es_funcion_void) {
        if (!es_expresion_void) {
            return 1;
        }
    } else {
        if (es_expresion_void) {
            return 1;
        }

        if (!es_tipo_compatible(tipo_funcion, tipo_expresion)) {
            return 1;
        }
    }

    /* 3. Comprobación de cobertura de rutas de control */
    if (rutas_con_return < rutas_totales) {
        return 2;
    }

    return 0;
}
```

**Análisis de complejidad:**

- **Temporal:** O(1), porque se realizan comparaciones de cadenas de tipos y validaciones de cantidades, sin ciclos ni recorridos de estructuras.
- **Espacial:** O(1), porque solamente se utilizan variables booleanas y no se reserva memoria dinámica.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Funciones sin retorno o retornos mixtos.

### Salida del harness
```text
=== AUDITORIA SESION 17 - Validacion de retornos y rutas ===
Caso                               esperado:              obtenido:              resultado
[ 1] int con int y dos rutas            esperado: 0            obtenido: 0            OK
[ 2] void devuelve valor                esperado: 1            obtenido: 1            OK
[ 3] ruta sin retorno                   esperado: 2            obtenido: 2            OK
[ 4] double acepta int                  esperado: 0            obtenido: 0            OK
[ 5] int no acepta double               esperado: 1            obtenido: 1            OK
[ 6] cero rutas de ejecucion            esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion17.md
```

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar la implementación, consideré importante comprobar que el conteo de rutas fuera coherente, ya que no tendría sentido tener cero rutas o indicar más retornos que rutas existentes. También se debe tener cuidado con el orden de las validaciones: primero se comprueba si el tipo de retorno es compatible y después se revisa si todas las rutas tienen un return. De esta manera, un tipo incorrecto no se confunde con una ruta sin retorno. La implementación contempla estas condiciones y las seis pruebas del harness se ejecutaron correctamente.

---
**Firma digital:** josue6martin

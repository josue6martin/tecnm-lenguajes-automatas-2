
# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Firmas de Función y Parámetros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353           
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Registro de prototipos y aridad  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar la función manualmente, tuve que cuidar que la búsqueda del nombre fuera exacta y que la cantidad de argumentos se comparara con la aridad registrada de cada función. Una dificultad fue contemplar los diferentes resultados: devolver 1 cuando la llamada es correcta, 0 cuando la cantidad de argumentos no coincide y -1 cuando la función no existe o se recibe un dato inválido. También fue necesario validar los punteros para evitar errores al trabajar con valores NULL.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Coincidencia estricta de parámetros |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en sistemas y compiladores.

Necesito implementar en lenguaje C la función verificar_llamada() para la Sesión 16 de Lenguajes y Autómatas II.

La función debe respetar exactamente la siguiente firma:

int verificar_llamada(const Funcion *catalogo, int total, const char *nombre, int nargs);

La estructura Funcion está declarada en api_sesion16.h y contiene:
- const char *nombre;
- int aridad;

Requisitos:
1. Recorrer el catálogo de funciones para buscar el nombre solicitado.
2. La comparación de nombres debe ser exacta utilizando strcmp.
3. Si el nombre existe y la aridad coincide con nargs, devolver 1.
4. Si el nombre existe, pero la cantidad de argumentos no coincide, devolver 0.
5. Si el nombre no existe, devolver -1.
6. Si catalogo o nombre son NULL, nargs es negativo o total no es válido, devolver -1.
7. Una función con cero parámetros solamente debe aceptar cero argumentos.
8. Comprobar que los nombres de las entradas del catálogo no sean NULL antes de compararlos.
9. No modificar la estructura ni la firma de la función.
10. No incluir main(), porque el programa será probado por un harness externo.

Entrega únicamente el código modular en C, compatible con C11 y compilación con:
gcc -std=c11 -Wall -Wextra -Werror

Incluye un breve análisis de la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion16.h"
#include <string.h>
#include <stddef.h>

int verificar_llamada(const Funcion *catalogo, int total, const char *nombre, int nargs) {
    // Verificacion de precondiciones y parametros
    if (!catalogo || !nombre || nargs < 0 || total <= 0) {
        return -1;
    }

    // Busqueda de la funcion dentro del catalogo
    for (int i = 0; i < total; i++) {
        if (catalogo[i].nombre && strcmp(catalogo[i].nombre, nombre) == 0) {
            return (catalogo[i].aridad == nargs) ? 1 : 0;
        }
    }

    // La funcion no existe en el catalogo
    return -1;
}
```

**Análisis de complejidad:**

- **Temporal:** O(n), porque en el peor caso se recorre todo el catálogo para encontrar el nombre o comprobar que no existe.
- **Espacial:** O(1), porque utiliza únicamente una variable de control y no necesita reservar memoria adicional.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas con número incorrecto de argumentos.

### Salida del harness
```text
=== AUDITORIA SESION 16 - Registro de prototipos y aridad ===
Caso                               esperado:              obtenido:              resultado
[ 1] suma con 3 argumentos              esperado: 1            obtenido: 1            OK
[ 2] suma con 2 argumentos              esperado: 0            obtenido: 0            OK
[ 3] suma con 4 argumentos              esperado: 0            obtenido: 0            OK
[ 4] funcion no registrada              esperado: -1           obtenido: -1           OK
[ 5] main sin argumentos                esperado: 1            obtenido: 1            OK
[ 6] nombre NULL                        esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion16.md
```

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código identifiqué que también era importante contemplar los nombres NULL dentro del catálogo, ya que intentar compararlos directamente con strcmp podría provocar un error. La implementación comprueba que el nombre de cada registro no sea NULL antes de realizar la comparación. También valida que el catálogo y el nombre recibido sean válidos, y que el número de argumentos no sea negativo. Con estas condiciones se evita comparar datos inválidos y se mantiene la diferencia entre una función inexistente y una llamada con aridad incorrecta. Las seis pruebas del harness se ejecutaron correctamente.

---
**Firma digital:** josue6martin

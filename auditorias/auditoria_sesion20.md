
# Bitácora de Requerimientos y Auditoría IA — Sesión 20
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 2: Analizador Semántico y Scopes  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353            
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Tabla de Símbolos y AST  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar las funciones manualmente, tuve que cuidar que la búsqueda recorriera todos los ámbitos mediante el puntero `padre`, comenzando por el más interno. Una de las dificultades fue controlar el shadowing, porque un nombre puede repetirse en diferentes ámbitos y se debe tomar primero el que está más cerca. También tuve que llevar un registro de los símbolos encontrados para evitar contarlos más de una vez en `contar_visibles()`. Para esto utilicé un arreglo auxiliar y comparaciones con `strcmp()`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 con GCC 13.3.0, utilizando la estructura `Ambito` definida en `api_sesion20.h`. Compilación con `-Wall -Wextra -Werror`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas, tablas de símbolos y compiladores. |
| **Especificación (E)** | Implementar `profundidad_de()` para buscar un nombre desde el ámbito actual hacia sus padres, respetando el shadowing. Implementar `contar_visibles()` para contar los nombres distintos de toda la cadena de ámbitos. Devolver -1 cuando un nombre no exista y -2 cuando se reciba un argumento NULL, según corresponda. |
| **Output (O)** | Código C modular que respete las firmas de `api_sesion20.h`, sin función `main()`, sin cambiar la estructura de datos y con análisis de complejidad. |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C y desarrollo de compiladores.

CONTEXTO:
Estoy trabajando en la Sesión 20 de Lenguajes y Autómatas II. Utilizo C11 con GCC y una estructura llamada Ambito que contiene un nombre, un arreglo de símbolos, un total de elementos y un puntero al ámbito padre.

ROL:
Actúa como un programador experto en tablas de símbolos, resolución de nombres y manejo de ámbitos (scopes).

ESPECIFICACIÓN:
Implementa las siguientes funciones respetando exactamente sus firmas:

int profundidad_de(const Ambito *actual, const char *nombre);
int contar_visibles(const Ambito *actual);

La función profundidad_de debe:
- Recorrer el ámbito actual y todos sus padres.
- Devolver 0 si el nombre se encuentra en el ámbito actual.
- Devolver 1, 2 o el nivel correspondiente si se encuentra en un ámbito superior.
- Respetar el shadowing, dando prioridad al ámbito más interno.
- Devolver -1 si el nombre no existe.
- Devolver -2 si actual o nombre es NULL.

La función contar_visibles debe:
- Recorrer todos los ámbitos desde el actual hasta el global.
- Contar cada nombre una sola vez, aunque aparezca repetido en diferentes ámbitos.
- Devolver -2 si el ámbito recibido es NULL.

RESTRICCIONES:
- Utiliza la estructura Ambito definida en api_sesion20.h.
- No cambies las firmas de las funciones.
- No agregues una función main().
- No utilices bibliotecas no estándar.
- Evita accesos inválidos a memoria.
- Considera el límite AMBITO_MAX definido en la API.
- El código debe compilar con GCC usando -std=c11 -Wall -Wextra -Werror.

OUTPUT:
Entrega únicamente el código C completo y modular, seguido de una explicación breve de su funcionamiento y complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion20.h"
#include <string.h>
#include <stdbool.h>

int profundidad_de(const Ambito *actual, const char *nombre) {
    if (!actual || !nombre) {
        return -2;
    }

    int profundidad = 0;
    for (const Ambito *curr = actual; curr != NULL; curr = curr->padre) {
        for (int i = 0; i < curr->total; ++i) {
            if (strcmp(curr->simbolos[i], nombre) == 0) {
                return profundidad;
            }
        }
        profundidad++;
    }
    return -1;
}

int contar_visibles(const Ambito *actual) {
    if (!actual) {
        return -2;
    }

    const char *vistos[AMBITO_MAX * 16];
    int unicos = 0;

    for (const Ambito *curr = actual; curr != NULL; curr = curr->padre) {
        for (int i = 0; i < curr->total; ++i) {
            bool duplicado = false;
            for (int j = 0; j < unicos; ++j) {
                if (strcmp(vistos[j], curr->simbolos[i]) == 0) {
                    duplicado = true;
                    break;
                }
            }
            if (!duplicado) {
                vistos[unicos++] = curr->simbolos[i];
            }
        }
    }
    return unicos;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Casos complejos de shadowing y ámbito.

### Salida del harness — Artesano
```text
=== AUDITORIA SESION 20 - Resolucion de nombres y visibilidad ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre en el ambito actual         esperado: 0            obtenido: 0            OK
[ 2] nombre dos niveles arriba          esperado: 2            obtenido: 2            OK
[ 3] shadowing gana el interno          esperado: 0            obtenido: 0            OK
[ 4] nombre no visible                  esperado: -1           obtenido: -1           OK
[ 5] visibles sin repetir               esperado: 3            obtenido: 3            OK
[ 6] ambito NULL                        esperado: -2           obtenido: -2           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion20.md
```

### Salida del harness — IA
```text
=== AUDITORIA SESION 20 - Resolucion de nombres y visibilidad ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre en el ambito actual         esperado: 0            obtenido: 0            OK
[ 2] nombre dos niveles arriba          esperado: 2            obtenido: 2            OK
[ 3] shadowing gana el interno          esperado: 0            obtenido: 0            OK
[ 4] nombre no visible                  esperado: -1           obtenido: -1           OK
[ 5] visibles sin repetir               esperado: 3            obtenido: 3            OK
[ 6] ambito NULL                        esperado: -2           obtenido: -2           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion20.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks* en los casos ejecutados; no se utiliza memoria dinámica.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código de la IA, observé que la función `contar_visibles()` utiliza un arreglo auxiliar de tamaño fijo (`AMBITO_MAX * 16`). Esto puede ser un problema si la cadena de ámbitos contiene más de 16 niveles con símbolos diferentes, porque se podrían guardar más elementos de los que permite el arreglo. También noté que no se comprueba si cada símbolo es NULL antes de utilizar `strcmp()`. Para prevenir estos problemas, se deben validar los símbolos antes de compararlos y controlar la cantidad de elementos almacenados para no sobrepasar la capacidad disponible. Las seis pruebas del harness pasaron correctamente, pero no comprueban estos casos adicionales.

---
**Firma digital:** `josue6martin`

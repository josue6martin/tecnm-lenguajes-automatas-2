
# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Tabla Hash con encadenamiento  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar la tabla hash me resultó un poco complicado organizar la búsqueda de los nombres y controlar las colisiones. Tuve que revisar cómo calcular la posición con la función djb2 y recorrer la lista enlazada para comprobar si un símbolo ya estaba registrado. También identifiqué que el campo `nombre` solamente permite almacenar 15 caracteres más el carácter nulo, por lo que es importante cuidar la copia de las cadenas para evitar escribir fuera de los límites de memoria.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Implementar una función hash djb2 y una función de inserción con manejo de colisiones mediante listas enlazadas. Evitar nombres duplicados y validar los argumentos. |
| **Output (O)** | Código modular en C, respetando las firmas de `api_sesion11.h`, sin función `main()` y con análisis de complejidad. |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy trabajando en la materia Lenguajes y Autómatas II y necesito
implementar una tabla de símbolos utilizando el lenguaje C y GCC.
Debo respetar el contrato definido en api_sesion11.h.

Rol:
Actúa como un ingeniero de software especializado en sistemas,
estructuras de datos y desarrollo de compiladores.

Especificación:
Implementa las funciones:
unsigned int hash_djb2(const char *clave);

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas,
                     const char *nombre, int tipo, int offset);

La función hash_djb2 debe utilizar la fórmula djb2, iniciando
con el valor 5381 y multiplicando por 33 para cada carácter.

La función insertar_simbolo debe:
- Validar que tabla y nombre no sean NULL.
- Validar que cubetas sea mayor que cero.
- Calcular el índice mediante hash_djb2(nombre) % cubetas.
- Recorrer la lista enlazada para verificar si el nombre ya existe.
- Regresar 0 si el símbolo ya está registrado.
- Crear un nuevo nodo y devolver 1 si se inserta correctamente.
- Devolver -1 cuando los argumentos sean inválidos o falle malloc.
- Manejar las colisiones mediante encadenamiento.
- Respetar la estructura Simbolo definida en api_sesion11.h.
- No incluir la función main().

Output:
Entrega únicamente el código modular en C, compatible con
gcc -std=c11 -Wall -Wextra -Werror, y explica brevemente
la complejidad temporal de las funciones.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion11.h"
#include <stdlib.h>
#include <string.h>

unsigned int hash_djb2(const char *clave) {
    if (!clave) return 0;

    unsigned int hash = 5381;
    int c;

    while ((c = (unsigned char)*clave++)) {
        hash = ((hash << 5) + hash) + c;
    }

    return hash;
}

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas,
                     const char *nombre, int tipo, int offset) {
    if (!tabla || !nombre || cubetas == 0) {
        return -1;
    }

    unsigned int indice = hash_djb2(nombre) % cubetas;

    /* Comprobacion de existencia previa */
    Simbolo *actual = tabla[indice];

    while (actual != NULL) {
        if (strcmp(actual->nombre, nombre) == 0) {
            return 0;
        }

        actual = actual->sig;
    }

    /* Asignacion de memoria para el nuevo simbolo */
    Simbolo *nuevo_nodo = (Simbolo *)malloc(sizeof(Simbolo));

    if (!nuevo_nodo) {
        return -1;
    }

    /* Inicializacion segura de campos */
    strncpy(nuevo_nodo->nombre, nombre,
            sizeof(nuevo_nodo->nombre) - 1);

    nuevo_nodo->nombre[sizeof(nuevo_nodo->nombre) - 1] = '\0';
    nuevo_nodo->tipo = tipo;
    nuevo_nodo->offset = offset;

    /* Insercion a la cabeza de la lista enlazada */
    nuevo_nodo->sig = tabla[indice];
    tabla[indice] = nuevo_nodo;

    return 1;
}
```

**Análisis de complejidad:**

- `hash_djb2()`: O(L), porque recorre cada carácter de la clave.
- `insertar_simbolo()`: O(1) en promedio cuando las claves se distribuyen adecuadamente; en el peor caso puede ser O(n), si todos los símbolos caen en una misma cubeta.
- Espacio: O(n) por los nodos almacenados en la tabla.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Manejo de claves duplicadas y colisiones.

### Salida del harness
```text
=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===
Caso                               esperado:              obtenido:              resultado
[ 1] hash_djb2("")                      esperado: 5381         obtenido: 5381         OK
[ 2] hash_djb2("abc")                   esperado: 193485963    obtenido: 193485963    OK
[ 3] insertar x dos veces               esperado: 0            obtenido: 0            OK
[ 4] dos claves, una cubeta             esperado: 2            obtenido: 2            OK
[ 5] offset de beta en cadena           esperado: 4            obtenido: 4            OK
[ 6] insertar en tabla NULL             esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion11.md
```

- [x] Caso base y límites probados superados sin Crash.
- [ ] No existen accesos fuera de límites ni Memory Leaks. No se comprobó de forma específica la ausencia de fugas de memoria; el harness no libera los nodos que reserva.
- [x] La complejidad teórica coincide con la implementada, considerando que la inserción es O(1) en promedio y O(n) en el peor caso.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código identifiqué que la IA no consideró completamente qué sucede cuando se intenta insertar un nombre de más de 15 caracteres. Esto se debe a que el campo `nombre` de la estructura `Simbolo` tiene una capacidad de 16 posiciones, incluyendo el carácter nulo. Aunque `strncpy` evita escribir fuera del arreglo, puede recortar el nombre y provocar problemas al buscarlo o comparar símbolos posteriormente. Para solucionar este caso, se debe validar la longitud del nombre antes de insertarlo y rechazar aquellos que superen los 15 caracteres. También observé que el harness comprueba las colisiones y los nombres repetidos, pero no prueba directamente este límite ni la liberación de memoria.

---

**Firma digital:** `josue6martin`


# Bitácora de Requerimientos y Auditoría IA — Sesión 14

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Declaración vs Uso de Variables  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353            
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Detección de variables no declaradas

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar esta actividad, identifiqué que uno de los principales problemas era comprobar que una variable estuviera declarada antes de poder utilizarla. También tuve que tener cuidado al trabajar con los arreglos y sus posiciones, ya que la tabla tiene una capacidad limitada de 16 nombres. Otro detalle fue utilizar correctamente las funciones `strcmp()` y `strlen()` para comparar los nombres y detectar si una cadena estaba vacía. Comprendí que no basta con buscar una parte del nombre, sino que la comparación debe ser exacta para evitar confundir variables diferentes.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. Implementación de una tabla de símbolos con capacidad máxima de 16 nombres. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Implementar `declarar()` para registrar nombres nuevos y evitar redeclaraciones. Implementar `usar()` para comprobar si una variable fue declarada previamente. Realizar comparaciones exactas y validar los parámetros recibidos. |
| **Output (O)** | Código modular en C, respetando las firmas de `api_sesion14.h`, sin función `main()` y con análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en la Sesión 14 de Lenguajes y Autómatas II.
Necesito implementar una tabla de símbolos en lenguaje C para
detectar el uso de variables no declaradas y evitar redeclaraciones.

Rol:
Actúa como un ingeniero de software especializado en C y desarrollo
de compiladores.

Especificación:
Implementa las siguientes funciones respetando exactamente sus firmas:

int declarar(Tabla *tabla, const char *nombre);
int usar(const Tabla *tabla, const char *nombre);

La estructura Tabla contiene un arreglo de nombres con capacidad
máxima de 16 elementos y un contador total.

La función declarar() debe:
- Devolver 1 cuando inserte un nombre correctamente.
- Devolver 0 cuando el nombre ya exista.
- Devolver -1 si recibe un puntero NULL o un nombre vacío.
- Evitar insertar nombres duplicados.
- No superar la capacidad máxima de la tabla.

La función usar() debe:
- Devolver 1 si el nombre ya fue declarado.
- Devolver 0 si el nombre no existe o los parámetros son inválidos.
- Utilizar comparaciones exactas para distinguir nombres como "x" y "xy".

Restricciones:
- Utilizar C11 compatible con GCC.
- Incluir api_sesion14.h.
- No modificar las estructuras ni las firmas.
- No incluir main().
- Compilar sin advertencias con -Wall -Wextra -Werror.
- No utilizar memoria dinámica.

Output:
Entrega únicamente el código modular de ambas funciones,
comentarios explicativos y el análisis de complejidad temporal
y espacial de la implementación.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion14.h"
#include <string.h>
#include <stddef.h>

int declarar(Tabla *tabla, const char *nombre) {
    if (tabla == NULL || nombre == NULL || *nombre == '\0') {
        return -1;
    }

    // Buscar si el nombre ya fue declarado.
    for (int i = 0; i < tabla->total; ++i) {
        if (strcmp(tabla->nombres[i], nombre) == 0) {
            return 0;
        }
    }

    // Comprobar que todavía exista espacio.
    if (tabla->total >= TABLA_MAX) {
        return 0;
    }

    // Registrar el nuevo nombre.
    tabla->nombres[tabla->total++] = nombre;

    return 1;
}

int usar(const Tabla *tabla, const char *nombre) {
    if (tabla == NULL || nombre == NULL || *nombre == '\0') {
        return 0;
    }

    // Buscar el nombre mediante comparación exacta.
    for (int i = 0; i < tabla->total; ++i) {
        if (strcmp(tabla->nombres[i], nombre) == 0) {
            return 1;
        }
    }

    return 0;
}
```

**Análisis de complejidad:**

- Complejidad temporal: O(n), porque se realiza una búsqueda lineal en la tabla para encontrar un nombre.
- Complejidad espacial: O(1) adicional, porque se utiliza la tabla existente y no se reserva memoria dinámica.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Uso antes de declarar y redeclaración.

### Salida del harness

```text
=== AUDITORIA SESION 14 - Declaracion previa y redeclaracion ===
Caso                               esperado:              obtenido:              resultado
[ 1] usar sin declarar                  esperado: 0            obtenido: 0            OK
[ 2] usar despues de declarar           esperado: 1            obtenido: 1            OK
[ 3] redeclarar el mismo nombre         esperado: 0            obtenido: 0            OK
[ 4] total tras redeclarar              esperado: 1            obtenido: 1            OK
[ 5] prefijo no es el mismo nombre      esperado: 0            obtenido: 0            OK
[ 6] declarar nombre vacio              esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion14.md
```

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks* en las pruebas realizadas.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código generado por la IA, observé que no se explicaba completamente qué sucedía cuando la tabla alcanzaba su capacidad máxima o cuando recibía parámetros inválidos. También era importante comprobar que los nombres se compararan de manera exacta y que no se confundieran identificadores como "x" y "xy". Para solucionar estos detalles, revisé las validaciones de los parámetros, agregué la comprobación del límite `TABLA_MAX` y verifiqué el funcionamiento con el harness. Finalmente, las seis pruebas se ejecutaron correctamente y se obtuvo un resultado de 6/6.

---

**Firma digital:** josue6martin

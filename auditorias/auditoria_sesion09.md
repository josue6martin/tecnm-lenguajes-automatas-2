
# Bitácora de Requerimientos y Auditoría IA — Sesión 09

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Reporte Formal de Errores Semánticos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness).

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto: Diagnóstico con línea y columna*

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar la función manualmente, identifiqué que uno de los principales cuidados era construir correctamente el mensaje con el nombre del archivo, la línea, la columna y la descripción del error. También tuve que considerar el tamaño del buffer para evitar escribir más caracteres de los permitidos. Para resolverlo utilicé `snprintf`, ya que permite limitar la cantidad de caracteres escritos y agregar el carácter nulo al final de la cadena.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Generar diagnósticos con el formato de GCC: archivo:línea:columna: error: mensaje. Validar los parámetros y evitar desbordamientos del buffer. |
| **Output (O)** | Código modular en C, respetando la firma de la función `reportar()` y considerando la seguridad de memoria y la complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en C y compiladores.

CONTEXTO:
Estoy trabajando en la Sesión 09 de Lenguajes y Autómatas II.
Necesito implementar una función en C que genere reportes de errores
con el formato utilizado por GCC.

ROL:
Actúa como un programador experto en C99 y manejo seguro de memoria.

ESPECIFICACIÓN:
Implementa la siguiente función respetando exactamente su firma:

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap);

El resultado debe tener el formato:
archivo:línea:columna: error: mensaje

Requisitos:
- Utilizar snprintf para evitar desbordamientos.
- Validar que salida no sea NULL y que cap sea mayor que cero.
- Manejar archivo y mensaje cuando sean NULL.
- Garantizar la terminación de la cadena con '\0'.
- No modificar la firma de la función.
- No incluir main().
- El código debe compilar con -Wall -Wextra -Werror.

OUTPUT:
Devuelve únicamente el código modular en C y una breve
explicación de su complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion09.h"
#include <stdio.h>
#include <stddef.h>

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap) {

    if (salida == NULL || cap <= 0) {
        return;
    }

    if (archivo == NULL) {
        archivo = "";
    }

    if (mensaje == NULL) {
        mensaje = "";
    }

    snprintf(salida, (size_t)cap,
             "%s:%d:%d: error: %s",
             archivo, linea, columna, mensaje);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Cuatro casos de prueba para comprobar el formato del diagnóstico, los valores de línea y columna, y la limitación del tamaño del buffer.

### Salida del harness

```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===
Caso                               esperado:              obtenido:              resultado
[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK
[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK
[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK
[ 4] cap=10 trunca                      esperado: main.c:14    obtenido: main.c:14    OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion09.md
```

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks* en las pruebas realizadas.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código, observé que no se incluyeron pruebas específicas para comprobar qué sucede cuando alguno de los parámetros recibe un valor NULL o cuando la capacidad del buffer es cero o negativa. Aunque la función de la IA contempla estas validaciones, el harness no las comprueba directamente. Por eso revisé las condiciones de entrada y mantuve las validaciones para evitar accesos inválidos y escrituras cuando no existe espacio disponible. También comprobé que `snprintf` limita la escritura cuando el buffer es pequeño.

---

**Firma digital:** `josue6martin`
